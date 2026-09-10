## Bigtable

Bigtable es el almacén de datos distribuido de Google para cargas de gran volumen, descrito en el paper de OSDI 2006. Es el sistema del que salió toda la familia: Spanner nació para cubrir lo que le faltaba, y HBase y Cassandra se inspiraron directamente en él. Internamente Google lo usó (y usa) para Search, Google Analytics, Earth, Gmail y muchas otras cosas.

La descripción del paper es "un mapa multidimensional ordenado, disperso y distribuido", que es literal: la clave es la tripleta `(fila, columna, timestamp)` y el valor es un array de bytes sin interpretar.

### Modelo de datos

Una tabla tiene:

- **Filas** identificadas por una clave de fila arbitraria (hasta 4 KB), **ordenadas lexicográficamente**. Este orden es la característica más importante del sistema desde el punto de vista práctico.
- **Familias de columnas**, declaradas en el esquema, que agrupan columnas relacionadas y son la unidad de control de acceso y de configuración (compresión, número de versiones a retener, TTL).
- **Calificadores de columna** dentro de cada familia, que no se declaran: puedes crear millones de columnas al vuelo. De ahí lo de "disperso" — una fila sólo ocupa espacio por las celdas que realmente tiene.
- **Versiones** por celda, indexadas por timestamp, con políticas de recolección de basura configurables.

Las operaciones sobre una fila individual son atómicas, sin importar cuántas columnas toquen. **No hay transacciones entre filas** ni joins ni SQL. Ésa es la limitación fundamental y la razón de existir de Spanner.

### Arquitectura

Los datos de una tabla se parten por rangos de clave en **tablets**, que son la unidad de distribución y balanceo. Un tablet vive en un **tablet server**, y un **master** asigna tablets a servidores, detecta caídas y balancea carga.

El punto clave, y esto se repite en AlloyDB y en Spanner, es que **el cómputo está separado del almacenamiento**: los tablets no guardan datos en el disco del servidor, sino en Colossus (originalmente GFS). Por eso reasignar un tablet de un servidor a otro no mueve datos, sólo cambia quién lo sirve, y por eso escalar es cuestión de añadir nodos.

Debajo de eso:

- **Chubby** (el servicio de bloqueos distribuidos, equivalente a ZooKeeper) elige el master, registra los tablet servers vivos y guarda el esquema.
- **SSTables** en Colossus: ficheros inmutables y ordenados con un índice al final.
- Un **memtable** en memoria más un log de commits para las escrituras. Cuando el memtable se llena, se vuelca a una SSTable nueva; periódicamente se hacen **compactaciones** que fusionan SSTables y descartan datos borrados. Es una arquitectura LSM-tree.
- Una jerarquía de metadatos de tres niveles (un tablet raíz en Chubby → tablets METADATA → tablets de usuario) para localizar cualquier fila.

Las lecturas consultan memtable y SSTables a la vez; los **filtros de Bloom** evitan tocar SSTables que seguro no contienen la fila.

### Diseño de la clave de fila

Aquí es donde se gana o se pierde con Bigtable, y el razonamiento es el mismo que te comentaba en Spanner pero aún más determinante.

Como las filas están ordenadas, las claves secuenciales (timestamps, IDs autoincrementales) concentran toda la escritura en el último tablet: eso es un **hotspot** y desperdicia el resto del clúster. Las técnicas habituales son invertir el orden de los campos, poner un hash o _salt_ delante, o usar claves compuestas del tipo `usuario#métrica#timestamp_invertido`.

En cambio, ese mismo orden es lo que hace eficientes los **escaneos de rango**: leer todas las lecturas de un sensor en una ventana temporal es una lectura secuencial de un tramo contiguo. Series temporales, datos de IoT, perfiles de usuario, grafos de adyacencia y almacenes de features para ML son los casos donde encaja bien.

### Cloud Bigtable hoy

El producto gestionado ofrece latencias de un dígito en milisegundos, escalado añadiendo nodos (cada nodo aporta un límite de operaciones por segundo y de rendimiento), réplicas en varios clústeres y regiones con consistencia eventual entre ellas, y perfiles de aplicación para dirigir cargas concretas a clústeres concretos. Es compatible con la API de HBase, tiene una interfaz SQL limitada añadida recientemente, y se integra con Dataflow y BigQuery.

### Cuándo usarlo

Encaja cuando tienes terabytes o petabytes, escrituras muy intensas, un patrón de acceso conocido por clave o por rango, y no necesitas transacciones ni consultas ad hoc. No encaja si necesitas joins, transacciones multi-fila, consultas analíticas flexibles (eso es BigQuery) o simplemente tienes menos de un terabyte, donde el coste base del clúster no se justifica frente a Cloud SQL o Firestore.

Puedo profundizar en lo que te interese: el diseño de claves con ejemplos concretos, cómo funciona la compactación LSM, o la comparación con Cassandra y DynamoDB.