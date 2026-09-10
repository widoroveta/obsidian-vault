## Memorystore

Memorystore es el servicio gestionado de cachés en memoria de Google Cloud. A diferencia de las cuatro anteriores, no es una base de datos propia: es Redis, Valkey y Memcached alojados y operados por Google, con el motor de código abierto sin modificar.

Eso cambia la naturaleza del resumen. En Spanner o Bigtable lo interesante era la arquitectura; aquí lo interesante son las decisiones operativas, porque el motor ya lo conoces (o lo puedes leer en la documentación de Redis).

### Los motores disponibles

- **Memorystore for Valkey** es la opción recomendada hoy para proyectos nuevos. Valkey es el fork de Redis que la Linux Foundation lanzó en 2024, cuando Redis Inc. cambió su licencia a una no libre; los grandes proveedores de nube se movieron en bloque hacia él. Es compatible a nivel de protocolo con Redis y Google le da mejor precio y las mejoras más recientes.
- **Memorystore for Redis Cluster** es la oferta en modo clúster: fragmentación horizontal, escalado en línea, hasta decenas de terabytes.
- **Memorystore for Redis** es la oferta original de instancia única, más limitada en escalado.
- **Memorystore for Memcached** existe para quien ya usa Memcached. Es más simple: sólo cadenas clave-valor, sin persistencia ni réplicas, pero con menos sobrecarga.

### Modelo de datos

Lo que aporta Redis/Valkey frente a un caché plano son las estructuras de datos nativas: cadenas, hashes, listas, conjuntos, conjuntos ordenados (la base de cualquier tabla de clasificación), HyperLogLog para conteos aproximados de cardinalidad, streams para colas de eventos, y tipos geoespaciales. Las operaciones son atómicas y se ejecutan en un hilo único, lo que elimina las condiciones de carrera sin necesidad de bloqueos explícitos.

Las latencias están en el rango de **microsegundos a un dígito de milisegundos**, uno o dos órdenes de magnitud por debajo de las bases de disco que hemos visto.

### Arquitectura y alta disponibilidad

En modo clúster los datos se reparten en **shards** por rango de slots hash, y cada shard puede tener de cero a cinco réplicas de lectura. Con réplicas en zonas distintas hay conmutación automática si cae el nodo principal, con un objetivo típico de decenas de segundos.

Un detalle que suele pasarse por alto: en modo clúster, **las operaciones multi-clave deben caer en el mismo shard**. Si tu código hace `MGET` sobre claves dispersas o transacciones con `MULTI` que abarcan varios shards, falla. La solución son los _hash tags_: poner una parte de la clave entre llaves (`user:{1234}:profile`) hace que sólo esa parte cuente para calcular el slot, forzando la colocación conjunta.

La conexión es privada dentro de la VPC, sin IP pública, con TLS opcional y autenticación mediante AUTH o IAM.

### Persistencia: el matiz importante

Redis puede persistir mediante snapshots (RDB) y/o append-only file (AOF), y Memorystore lo soporta. Pero conviene tener claro que **eso no lo convierte en una base de datos duradera**. La persistencia sirve para recuperar el estado del caché tras un reinicio y evitar la tormenta de fallos que provoca arrancar en frío; no da garantías comparables a un WAL sincrónico. Si un dato no puede perderse, su fuente de verdad tiene que estar en otro sitio.

La política de expulsión (`maxmemory-policy`) es la otra cara: cuando la memoria se llena, Memorystore descarta claves según la política configurada — normalmente `allkeys-lru` para un caché puro, o `noeviction` si el contenido no es descartable, aunque entonces las escrituras empiezan a fallar en lugar de perder datos silenciosamente. Elegir mal aquí es una fuente habitual de sorpresas.

### Patrones de uso

- **Caché de lectura** delante de Cloud SQL, AlloyDB o Spanner: el patrón _cache-aside_ clásico, donde la aplicación consulta el caché, y si falla va a la base y rellena. Reduce coste y latencia, y en Firestore también reduce la factura por operación.
- **Sesiones** de usuario, con TTL automático.
- **Limitación de tasa** y contadores, usando `INCR` atómico. Esto resuelve precisamente el problema del límite de una escritura por segundo por documento que mencionaba en Firestore: el contador vive en memoria y se descarga periódicamente.
- **Colas y pub/sub** ligeros, con listas o streams.
- **Bloqueos distribuidos** y coordinación entre instancias.
- **Búsqueda vectorial**, añadida recientemente a Valkey y Redis Cluster, para RAG y recomendaciones con latencia muy baja.

### Dónde encaja

Memorystore no compite con las otras cuatro: se pone **delante** de ellas. La decisión no es "¿Spanner o Memorystore?" sino "¿necesito una capa de caché?", y la respuesta depende de si tienes lecturas repetidas de los mismos datos, presión de latencia, o costes de consulta que quieras recortar.

Las alternativas dentro de GCP son el caché en proceso de la propia aplicación (más rápido pero no compartido y difícil de invalidar) o ejecutar Redis tú mismo en GKE (más control, más trabajo operativo).

Con esto cierras el recorrido: Spanner y AlloyDB para relacional, Bigtable y Firestore para NoSQL, Memorystore para la capa en memoria. Si quieres, puedo hacerte un cuadro comparativo de las cinco, o entrar en estrategias de invalidación de caché, que es donde se complica en la práctica.