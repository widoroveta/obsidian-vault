## Labels en GCP

Las labels son pares clave-valor que se adjuntan a los recursos de Google Cloud para clasificarlos. No hacen nada por sí solas: no otorgan permisos, no cambian el comportamiento del recurso, no imponen restricciones. Son **metadatos organizativos**, y su valor aparece cuando querés filtrar, agrupar o desglosar información sobre una infraestructura que ya creció.

### Formato y reglas

Las restricciones técnicas son el tipo de detalle que se pregunta directamente:

- Máximo **64 labels por recurso**.
- La **clave** es obligatoria y debe empezar por una letra minúscula. Puede tener entre 1 y 63 caracteres.
- El **valor** puede estar vacío, y admite hasta 63 caracteres.
- Sólo se permiten **minúsculas, números, guiones y guiones bajos**. Nada de mayúsculas, espacios ni caracteres especiales.
- Se admiten caracteres internacionales UTF-8, aunque en la práctica casi nadie los usa.
- Las claves deben ser **únicas dentro de un recurso**: no podés tener dos labels con la misma clave.

Se pueden aplicar y modificar en cualquier momento, sin recrear el recurso.

### Para qué se usan realmente

**Desglose de facturación.** Este es el caso de uso principal y el que aparece en casi todas las preguntas. Las labels se propagan a la exportación de facturación a BigQuery, lo que permite responder "¿cuánto gastó el equipo de marketing el mes pasado?" o "¿cuánto cuesta el entorno de staging?". Sin labels, la factura es una lista de servicios sin contexto de negocio. Los informes de Cloud Billing permiten agrupar y filtrar por label directamente.

**Filtrado e inventario.** Buscar todos los recursos de un entorno, de un dueño o de una aplicación concreta:

```
gcloud compute instances list --filter="labels.env=prod"
```

**Automatización y scripting.** Apagar todas las VMs con `env=dev` los fines de semana, aplicar respaldos según `backup=daily`, o alimentar informes de cumplimiento.

**Monitorización.** Cloud Monitoring puede agrupar métricas y definir alertas por label, lo que permite paneles por equipo o por servicio.

### Convenciones habituales

Las claves que casi todo el mundo acaba usando, y que conviene estandarizar antes de que la infraestructura crezca:

|Clave|Ejemplo de valor|Para qué|
|---|---|---|
|`env`|`prod`, `staging`, `dev`|Separar entornos|
|`team` u `owner`|`data-platform`, `jperez`|Responsabilidad|
|`cost-center`|`cc-4471`|Imputación contable|
|`app` o `service`|`checkout-api`|Agrupar por aplicación|
|`component`|`frontend`, `db`|Rol dentro de la app|
|`managed-by`|`terraform`|Distinguir lo automatizado de lo manual|

Lo importante no es la lista concreta sino la **consistencia**: `env=prod` y `environment=production` conviviendo en la misma organización arruinan cualquier informe. Definir el esquema por escrito y hacerlo cumplir con Terraform o con una política es lo que marca la diferencia.

### Labels frente a tags — la confusión clásica

Los dos son pares clave-valor y por eso se mezclan constantemente, pero cumplen funciones distintas:

||Labels|Tags|
|---|---|---|
|Propósito|Clasificar, facturar, filtrar|Control de acceso condicional|
|Efecto funcional|Ninguno|Sí: condicionan políticas|
|Herencia|No se heredan|**Sí**, por la jerarquía de recursos|
|Definición|Libres, cualquier usuario con permiso de edición|Recursos gestionados a nivel de organización|
|Usos típicos|Informes de coste, inventario|Condiciones IAM, reglas de firewall, Organization Policy|

Los **tags** son objetos de primera clase que se crean como claves y valores definidos centralmente, se heredan hacia abajo en la jerarquía y se pueden usar como condición en una política de IAM ("este rol aplica sólo a recursos con `environment=production`") o en una regla de firewall. Es la herramienta de gobierno; las labels son la de contabilidad.

Aparte existen las **network tags** de Compute Engine, que son simples cadenas sin valor asociado (no pares) y se usan para aplicar reglas de firewall a grupos de VMs. Tres cosas con nombres parecidos y funciones diferentes.

### Limitaciones que sorprenden

- **No todos los servicios admiten labels**, y no todos los que las admiten las propagan a la facturación. La cobertura mejoró mucho pero conviene verificar el servicio concreto.
- Las labels de un proyecto **no se heredan** a los recursos dentro de él. Etiquetar el proyecto no etiqueta sus VMs.
- Los recursos creados antes de definir la convención quedan sin etiquetar, y no hay un mecanismo nativo que los etiquete retroactivamente. Hay que hacerlo con scripts.
- No podés **exigir** labels con una Organization Policy de forma directa; lo habitual es imponerlo en el pipeline de infraestructura o detectar los recursos sin etiquetar con un análisis periódico.

### Para el examen

|Pista en el enunciado|Respuesta|
|---|---|
|Desglosar costes por equipo o departamento|Labels|
|Filtrar o inventariar recursos|Labels|
|Restringir permisos según el entorno del recurso|Tags con condiciones IAM|
|Aplicar reglas de firewall a un grupo de VMs|Network tags|
|Agrupar proyectos por departamento|Carpetas (no labels)|
|Aislar cuotas y facturación de forma estricta|Proyectos separados|

La distinción de la última fila merece atención: si la pregunta pide **organizar y aplicar políticas**, la respuesta es carpetas o proyectos; si pide **informar y clasificar**, son labels.

Si querés, te armo un cuestionario que mezcle labels, cuotas y Resource Manager, o seguimos con IAM, que es la pieza que cierra toda esta sección de gestión de recursos.
