## Quiz: Servicios de Almacenamiento y Bases de Datos

_Puntuación para aprobar: 66%_

---

**1. ¿Qué servicio de almacenamiento de datos elegirías si solo necesitaras migrar a la nube una base de datos relacional estándar que corre en una única máquina en un centro de datos?**

- Cloud SQL
- Cloud Storage
- Persistent Disk
- BigQuery

**Respuesta: Cloud SQL**

Es el caso de "lift and shift" clásico. Cloud SQL ofrece MySQL, PostgreSQL y SQL Server gestionados, así que migrás el mismo motor sin reescribir la aplicación, y Google se encarga de backups, parches y replicación. Las dos pistas del enunciado son "relacional estándar" y "una única máquina": no hace falta escalado horizontal de escrituras, así que no necesitás Spanner.

Por qué no las otras: **Cloud Storage** guarda objetos (archivos), no tablas consultables por SQL. **Persistent Disk** es almacenamiento de bloques que se conecta a una VM — podrías instalar la base a mano encima, pero entonces seguís administrando todo vos. **BigQuery** es analítico: consultas de escaneo masivo, no transacciones fila por fila.

---

**2. ¿Qué servicio de almacenamiento de datos de Google Cloud ofrece transacciones ACID y puede escalar a nivel global?**

- Cloud Storage
- Cloud SQL
- Spanner
- Cloud CDN

**Respuesta: Spanner**

La combinación "ACID + global" es la razón de existir de Spanner, y es justo lo que tradicionalmente se consideraba imposible de tener a la vez. Lo logra con TrueTime, la API de reloj basada en GPS y relojes atómicos que devuelve un intervalo de incertidumbre acotado y permite garantizar consistencia externa entre continentes.

Por qué no las otras: **Cloud SQL** sí da ACID, pero está limitado a un nodo primario en una región — falla la segunda condición, y es el distractor pensado para hacerte dudar. **Cloud Storage** no tiene transacciones. **Cloud CDN** ni siquiera es almacenamiento: es una red de distribución de contenido en el borde.

---

**3. ¿Qué servicio de almacenamiento de datos proporciona servicios de data warehouse para almacenar datos, pero además ofrece una interfaz SQL interactiva para consultarlos?**

- Managed Service for Apache Spark
- Datalab
- Cloud SQL
- BigQuery

**Respuesta: BigQuery**

BigQuery es el data warehouse serverless de Google: almacenamiento columnar, sin infraestructura que aprovisionar, y consultas SQL sobre terabytes o petabytes en segundos. La palabra clave del enunciado es "data warehouse", que en el vocabulario de GCP apunta siempre a BigQuery.

Por qué no las otras: **Managed Service for Apache Spark** es un motor de procesamiento, no un almacén de datos. **Datalab** era una herramienta de notebooks para análisis interactivo (ya retirada, reemplazada por Vertex AI Workbench) — no almacena nada. **Cloud SQL** es transaccional (OLTP), no analítico (OLAP): funciona bien con muchas operaciones pequeñas, mal con escaneos de tablas enteras.

---

### La regla para aprobar la sección

Casi todas estas preguntas se resuelven identificando **dos restricciones** en el enunciado:

|Pistas|Servicio|
|---|---|
|Relacional + una máquina / migración|Cloud SQL|
|Relacional + global + ACID|Spanner|
|SQL + data warehouse / analítica|BigQuery|
|Petabytes + clave-valor + baja latencia + series temporales|Bigtable|
|App móvil/web + tiempo real + offline|Firestore|
|Archivos / objetos / respaldos|Cloud Storage|
|Caché en memoria + microsegundos|Memorystore|
|NFS compartido entre VMs|Filestore|

Si querés, te armo un cuestionario de práctica con las que faltan de esa tabla, que son las que más suelen aparecer.