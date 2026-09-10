## Firestore

Firestore es la base de datos de documentos NoSQL de Google Cloud, orientada sobre todo a aplicaciones web y móviles. Es la sucesora de Cloud Datastore (que a su vez venía de Megastore, el sistema que Spanner acabó reemplazando), y llegó en 2017 tras la adquisición de Firebase.

Lo que la distingue de todo lo que hemos visto antes no es tanto la arquitectura como el **modelo de acceso**: es la única de estas bases pensada para que los clientes se conecten directamente a ella, sin capa de servidor intermedia.

### Modelo de datos

La jerarquía es colecciones → documentos → subcolecciones:

- Un **documento** es un objeto tipo JSON con campos tipados (cadenas, números, booleanos, timestamps, geopuntos, referencias a otros documentos, mapas y arrays anidados). Límite de 1 MiB por documento.
- Una **colección** es un contenedor de documentos. No tiene esquema: los documentos de una misma colección pueden tener campos distintos.
- Un documento puede contener **subcolecciones**, y así sucesivamente hasta 100 niveles. Los datos anidados no se cargan al leer el documento padre, lo que permite modelar jerarquías profundas sin penalización.

Por debajo, el almacenamiento es Spanner. De ahí vienen las garantías de consistencia fuerte y la replicación multirregional.

### Consultas e índices

Aquí está la particularidad más importante de Firestore, y conviene entenderla antes de diseñar nada.

**Todas las consultas usan un índice.** Firestore indexa automáticamente cada campo de cada documento (índices simples, ascendente y descendente), y para consultas que filtran u ordenan por varios campos hay que declarar un **índice compuesto**. Si falta, la consulta no se degrada: falla, y el mensaje de error incluye un enlace para crear el índice que falta.

La consecuencia es que el rendimiento de lectura **depende del tamaño del resultado, no del tamaño de la colección**. Una consulta sobre mil documentos y sobre mil millones cuesta lo mismo si ambas devuelven veinte resultados.

El precio de esa garantía son las limitaciones del lenguaje de consulta: no hay joins, no hay agregaciones más allá de `count()`, `sum()` y `average()`, no se pueden hacer desigualdades sobre más de un campo con libertad, y no existe la búsqueda de texto completo nativa (para eso se integra con Algolia, Elastic o similares). Firestore te obliga a saber de antemano cómo vas a consultar los datos, y a **desnormalizar** en consecuencia — duplicar campos entre documentos es una práctica normal, no un antipatrón.

### Escuchas en tiempo real

Cualquier consulta puede convertirse en una suscripción. El cliente recibe un snapshot inicial y luego, mientras la escucha esté activa, sólo los **cambios**: qué documentos se añadieron, modificaron o eliminaron. Es la razón por la que mucha gente elige Firestore, y sustituye toda la maquinaria de websockets y sincronización que normalmente habría que escribir a mano.

Los SDK incluyen además **persistencia offline**: la app sigue leyendo y escribiendo contra una caché local sin conexión, y los cambios se sincronizan al recuperarla. Las escrituras locales se reflejan de inmediato en la UI antes de confirmarse en el servidor.

### Seguridad

Como los clientes hablan directamente con la base de datos, la autorización no puede vivir en un backend. Se expresa mediante **Security Rules**, un lenguaje declarativo que se evalúa en el servidor para cada operación y que puede consultar la identidad del usuario autenticado, el contenido del documento e incluso otros documentos.

Esto es potente pero es también el punto donde más se falla: unas reglas mal escritas exponen la base entera, y hay un historial abundante de aplicaciones Firebase filtrando datos por dejarlas en modo de prueba. Conviene tratarlas con la misma seriedad que el código de producción y probarlas con el emulador.

### Modos y límites

Firestore tiene dos modos, que se eligen al crear la base de datos y no se pueden cambiar: **modo Nativo** (tiempo real, offline, acceso desde cliente) y **modo Datastore** (compatible con la API antigua de Datastore, para servidores, sin tiempo real). Para proyectos nuevos, Nativo salvo que haya que mantener código heredado.

Los límites operativos a tener presentes: unas 10.000 escrituras por segundo por base de datos, y **1 escritura por segundo por documento** en régimen sostenido. Ese segundo límite es el que sorprende: un contador global en un solo documento no aguanta tráfico real, y la solución habitual son los _distributed counters_ (repartir el contador entre varios documentos y sumarlos al leer).

La facturación es por **operación** — lecturas, escrituras y borrados de documentos — más almacenamiento y red. Eso cambia el cálculo: una consulta que devuelve 500 documentos son 500 lecturas facturadas, y un patrón de UI descuidado puede resultar caro. Las escuchas en tiempo real facturan sólo los documentos que cambian, lo que suele salir a favor.

### Dónde encaja

||Encaja|No encaja|
|---|---|---|
|Firestore|Apps móviles/web, tiempo real, offline, acceso desde cliente|Analítica, agregaciones complejas, escrituras masivas por documento|
|Bigtable|Series temporales, petabytes, escaneos de rango|Consultas ad hoc, apps de cliente|
|Spanner|Escala global con transacciones y SQL|Proyectos pequeños, presupuesto ajustado|
|AlloyDB|PostgreSQL existente, HTAP regional|Escrituras más allá de un nodo|

Con esto tienes las cuatro bases del portafolio de Google cubiertas. Puedo entrar en el diseño de datos desnormalizado con ejemplos, en las Security Rules, o en la comparación con MongoDB y DynamoDB.