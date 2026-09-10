![[Pasted image 20260909230543.png]]
## Resource Manager de GCP

El Resource Manager es el servicio que define y organiza la **jerarquía de recursos** de Google Cloud. No almacena datos ni ejecuta nada: su función es dar estructura, y esa estructura es la que determina cómo se aplican los permisos y las políticas en toda la organización.

### La jerarquía

Cuatro niveles, de arriba hacia abajo:

**Organización.** El nodo raíz, que representa a la empresa. Se crea automáticamente al vincular un dominio de Cloud Identity o Google Workspace. Sin organización (por ejemplo, con una cuenta personal de Gmail) los proyectos quedan sueltos, sin nodo padre, y se pierden las políticas centralizadas. Es el primer paso de cualquier despliegue empresarial serio.

**Carpetas.** Agrupaciones opcionales que permiten modelar departamentos, equipos, entornos o unidades de negocio. Se pueden anidar hasta **10 niveles de profundidad**, y una carpeta puede contener otras carpetas y proyectos, pero cada recurso tiene un único padre — es un árbol, no un grafo.

**Proyectos.** El nivel donde realmente vive todo. Un proyecto es el contenedor de los recursos, la unidad de facturación, la frontera de aislamiento y el ámbito donde se habilitan las APIs. Todo recurso de GCP pertenece a exactamente un proyecto.

**Recursos.** Las VMs, los buckets, las instancias de Cloud SQL, etcétera.

### Los tres identificadores de un proyecto

Esto sale en los exámenes con frecuencia porque es fácil de confundir:

- **Project ID**: cadena única a nivel global, elegida por ti, **inmutable**, y no reutilizable ni después de borrar el proyecto. Es el que usan los comandos de `gcloud` y las APIs.
- **Project name**: etiqueta legible, modificable, no necesariamente única.
- **Project number**: número asignado automáticamente por Google, inmutable, usado internamente por algunos servicios.

### Herencia de políticas

Aquí está lo importante, y es el motivo real por el que existe el Resource Manager.

Las **políticas de IAM** se aplican en cualquier nivel de la jerarquía y **se heredan hacia abajo**. Un rol concedido en la organización aplica a todas las carpetas, proyectos y recursos por debajo. La política efectiva sobre un recurso es la unión de su propia política y de todas las heredadas de sus ancestros.

La consecuencia práctica es que la herencia es **acumulativa y no se puede restar**: una política en un nivel inferior no puede quitar un permiso concedido más arriba. Por eso los permisos amplios se conceden con mucho cuidado en los nodos altos, y se sigue el principio de mínimo privilegio concediendo lo justo en el nivel más bajo que sirva.

### Organization Policy Service

Es el complemento del anterior y suele confundirse con IAM, así que conviene separar los dos:

- **IAM** responde a "¿quién puede hacer qué?" — concede permisos a identidades.
- **Organization Policy** responde a "¿qué se puede hacer, sin importar quién?" — impone restricciones sobre la configuración de los recursos.

Las restricciones se aplican en organización, carpeta o proyecto y también se heredan. Ejemplos habituales: prohibir IPs externas en las VMs, limitar las regiones donde se pueden crear recursos, exigir OS Login, bloquear la creación de claves para cuentas de servicio, o impedir el acceso público a buckets. Una política de organización puede restringir a un administrador que técnicamente tiene el permiso IAM para hacer la acción — es una barrera de guardia, no un permiso.

### Etiquetas y cuotas

Las **labels** son pares clave-valor para clasificar recursos, sobre todo para desglosar la facturación (por equipo, entorno o centro de coste). Los **tags** son distintos: se usan como condición en políticas de IAM y reglas de firewall, y sí participan en la lógica de control de acceso.

Las **cuotas** se gestionan a nivel de proyecto, y ese es otro motivo para dividir en varios proyectos: aíslan los límites de consumo entre entornos.

### Ciclo de vida del proyecto

Al eliminar un proyecto entra en un estado de borrado pendiente con un margen de recuperación de **30 días**, tras el cual se destruye definitivamente. También se pueden mover proyectos entre carpetas y organizaciones, con permisos en el origen y el destino.

### Para el examen

Las señales típicas y su respuesta:

|Pista en el enunciado|Respuesta|
|---|---|
|Agrupar proyectos por departamento|Carpetas|
|Aplicar una política a toda la empresa de golpe|Nivel de organización + herencia|
|Impedir una configuración concreta a todos, incluidos los admins|Organization Policy|
|Conceder acceso a una persona|IAM|
|Desglosar la factura por equipo|Labels|
|Aislar cuotas y entornos|Proyectos separados|

Si querés, puedo armarte preguntas de práctica sobre esta sección, o profundizar en IAM, que es la parte que más se entrelaza con esto y donde más se pierden puntos.