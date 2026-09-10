## AlloyDB

AlloyDB es el servicio de PostgreSQL gestionado "premium" de Google Cloud, anunciado en 2022. La idea es quedarse con PostgreSQL de verdad (el mismo motor, no un reimplementado compatible) pero sustituir la capa de almacenamiento por una diseñada para la nube. Es la respuesta de Google a Amazon Aurora, y la comparación es inevitable porque el enfoque arquitectónico es muy parecido.

### Separación de cómputo y almacenamiento

Ésta es la decisión de diseño de la que se derivan casi todas las demás. En PostgreSQL clásico el proceso escribe páginas de 8 KB a un disco local y además un WAL (write-ahead log) para durabilidad. AlloyDB desacopla las dos capas:

- El **nodo de cómputo** es PostgreSQL modificado. Cuando hace commit, escribe únicamente el WAL en un servicio de log de baja latencia. No escribe páginas de datos.
- La **capa de almacenamiento** es un servicio distribuido y regional que recibe ese WAL y se encarga del resto.

Como el commit sólo depende del log, las escrituras son más rápidas y el volumen de E/S en la ruta crítica es mucho menor.

### El "log processing service" (LPS)

Es la pieza distintiva. En lugar de que el nodo primario aplique el WAL a las páginas, hay un conjunto de procesos en la capa de almacenamiento que consumen el log y **materializan las páginas en paralelo**, cerca de donde viven los datos. Se escala de forma independiente al cómputo.

Consecuencias prácticas:

- **Recuperación tras caída casi instantánea**, porque no hay que reproducir el WAL al arrancar: las páginas ya están materializadas.
- **Réplicas de lectura sin retraso de aplicación**, ya que todas leen del mismo almacenamiento compartido en lugar de replicar y aplicar el log cada una por su cuenta. Añadir una réplica no cuesta una copia entera de los datos.
- **Vacuum y checkpoints** con mucho menos impacto en el rendimiento del primario.

Los datos se replican en tres zonas de la región, con backups continuos y recuperación a un punto en el tiempo.

### Caché en varios niveles

Hay tres niveles: el `shared_buffers` habitual de PostgreSQL, una **caché ultrarrápida** en SSD local del nodo de cómputo (que es donde AlloyDB gana buena parte de su ventaja en cargas transaccionales), y la capa de almacenamiento por debajo. El poblado de la caché SSD se guía por aprendizaje automático sobre los patrones de acceso.

### Motor columnar

AlloyDB añade un almacén **columnar en memoria** que mantiene automáticamente copias de las columnas más consultadas, y el planificador decide por consulta si usar el camino de filas o el columnar (puede combinar ambos en un mismo plan). Un asesor sugiere qué columnas mantener en memoria basándose en la carga real. Google reporta órdenes de magnitud de mejora en consultas analíticas frente a PostgreSQL estándar; la cifra de marketing habitual es "hasta 100x", que conviene tomar como lo que es.

El objetivo es HTAP: analítica razonable sobre los datos transaccionales sin montar un pipeline hacia un almacén separado.

### Compatibilidad

Es PostgreSQL real: mismas extensiones (pgvector, PostGIS, etc.), mismos drivers, mismo SQL. Eso significa que la migración desde PostgreSQL autogestionado o Cloud SQL es directa, y que no hay bloqueo de sintaxis. Lo que no es compatible es la capa de almacenamiento, así que no puedes llevarte el motor de AlloyDB a otro sitio; ahí sí hay dependencia del proveedor.

Existe **AlloyDB Omni**, una versión que se ejecuta en tu propia infraestructura (on-premise, otras nubes, portátil) con el motor columnar y muchas de las optimizaciones, aunque sin la capa de almacenamiento distribuida completa. Es una jugada interesante frente a Aurora, que no tiene equivalente.

### AlloyDB frente a Spanner

Es la pregunta natural después de tu mensaje anterior, y la respuesta es que resuelven problemas distintos:

||Spanner|AlloyDB|
|---|---|---|
|Escalado de escrituras|Horizontal, sin límite práctico|Un solo nodo primario|
|Alcance|Multirregional / global|Regional|
|Compatibilidad|GoogleSQL o dialecto PostgreSQL|PostgreSQL completo|
|Modelo|Propietario, requiere diseño de claves|Relacional convencional|
|Encaje|Escala global, esquemas nuevos|Migrar PostgreSQL existente, HTAP|

En resumen: si necesitas escalar escrituras más allá de una máquina o consistencia fuerte entre continentes, Spanner. Si tienes PostgreSQL y quieres mucho más rendimiento y disponibilidad sin reescribir nada, AlloyDB. Cloud SQL sigue siendo la opción más barata y sencilla cuando no necesitas ninguna de las dos cosas.

Puedo entrar en más detalle en lo que quieras: el funcionamiento del motor columnar, cómo se compara con Aurora punto por punto, o cuándo AlloyDB Omni tiene sentido.