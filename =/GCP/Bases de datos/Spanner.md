Asumo que te refieres a **Google Spanner**, la base de datos distribuida (y su famoso paper de OSDI 2012). Si querías otra cosa, dime.

### Qué es

Spanner es la base de datos distribuida de Google: escalable a nivel global, replicada de forma sincrónica y con transacciones ACID distribuidas. Nació hacia 2007 como sucesora de Bigtable y Megastore, que se quedaban cortas para aplicaciones que necesitaban transacciones y esquemas relacionales (el caso motivador fue F1, el backend de AdWords).

Su aportación central es combinar dos cosas que tradicionalmente se consideraban incompatibles: la escalabilidad horizontal de los sistemas NoSQL y la semántica transaccional fuerte de las bases relacionales.

### Modelo de datos

Es "semi-relacional": tablas con filas, columnas y esquemas, pero cada fila debe tener una clave primaria, y por debajo todo es un almacén clave-valor versionado (clave, timestamp → valor). Soporta SQL y permite _interleaving_: declarar que una tabla hija se almacene físicamente junto a las filas de su tabla padre, para que las consultas relacionadas caigan en el mismo nodo.

### Arquitectura

- Un **universe** se divide en **zonas** (unidad de despliegue físico y de aislamiento).
- Cada zona tiene muchos **spanservers**, y cada spanserver gestiona cientos de **tablets** (fragmentos de datos).
- Cada tablet se replica mediante un grupo **Paxos**. El líder del grupo mantiene una tabla de bloqueos para el control de concurrencia (2PL) y un gestor de transacciones para coordinar transacciones que cruzan varios grupos, con **2PC** por encima de Paxos.
- Los datos se agrupan en **directorios**, que son la unidad de movimiento y de configuración de ubicación (cuántas réplicas, en qué regiones, a qué distancia de los usuarios).

### TrueTime

Es la pieza que hace que todo lo demás funcione. En lugar de devolver un instante, la API de TrueTime devuelve un **intervalo** `[earliest, latest]` que garantiza contener el tiempo absoluto real. La incertidumbre (ε) se mantiene en pocos milisegundos gracias a receptores GPS y relojes atómicos desplegados en cada centro de datos.

Con eso Spanner consigue:

- **Consistencia externa** (linealizabilidad global): si una transacción T1 termina antes de que T2 empiece, el timestamp de T1 es menor. Se logra con la regla de _commit wait_: el líder espera a que su timestamp de commit haya pasado con certeza antes de hacerlo visible.
- **Lecturas sin bloqueos**: las lecturas de solo lectura y los snapshot reads se ejecutan a un timestamp concreto sin tomar candados, en cualquier réplica suficientemente actualizada.
- **Cambios de esquema no bloqueantes**, programados atómicamente en un timestamp futuro.

### Compromisos

La escritura paga el precio de Paxos entre regiones más el commit wait, así que la latencia de escritura es notablemente mayor que en una base local. En términos de CAP es técnicamente CP, aunque Google argumenta que en la práctica se comporta como CA porque su red privada hace que las particiones sean rarísimas (Cloud Spanner ofrece un SLA de 99,999 % en configuración multirregión).

### En la práctica hoy

Cloud Spanner es el producto gestionado: escalado horizontal por nodos o _processing units_, interfaces GoogleSQL y PostgreSQL, change streams, backups y recuperación a un punto en el tiempo. El punto de diseño más importante para el usuario es la **clave primaria**: las claves monótonas (timestamps, autoincrementales) concentran la carga en un solo split y provocan _hotspots_; se recomienda usar UUIDs o invertir los bits de la clave.

Su influencia se ve en CockroachDB y YugabyteDB, que replican el diseño pero, al no tener hardware de reloj dedicado, usan relojes híbridos lógicos (HLC) en lugar de TrueTime.

Si te interesa, puedo profundizar en cualquier parte: el protocolo de transacciones paso a paso, cómo se implementa TrueTime, o el diseño de esquemas en Cloud Spanner.