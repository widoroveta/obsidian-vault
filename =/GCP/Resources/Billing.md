## Cloud Billing

Cloud Billing es el sistema de facturación y control de gasto de Google Cloud. Su pieza central es la **cuenta de facturación**, que es un objeto **independiente de la jerarquía de recursos** que vimos con el Resource Manager. Esa separación es la clave para entender todo lo demás: los proyectos viven en organizaciones y carpetas, pero se **vinculan** a una cuenta de facturación que puede estar fuera de ese árbol.

### La cuenta de facturación

Define quién paga y cómo. Sus propiedades importantes:

- Un **proyecto se vincula a exactamente una** cuenta de facturación. Sin vínculo, los servicios de pago deja de funcionar.
- Una **cuenta de facturación puede pagar muchos proyectos**. Es la relación uno a muchos habitual.
- El vínculo se puede cambiar, lo que permite reasignar el coste de un proyecto a otro departamento.
- Tiene su propia moneda y su propio ciclo de facturación.

Hay dos tipos:

**Autoservicio (self-serve).** Se paga con tarjeta de crédito o débito, el cargo es automático, y es lo que obtenés al registrarte por tu cuenta. La documentación de uso es un recibo, no una factura fiscal formal en la mayoría de países.

**Facturada (invoiced).** Google emite una factura con condiciones de pago, típicamente a 30 días, y se paga por transferencia. Requiere aprobación de crédito y un volumen mínimo. Es lo estándar en empresas, y es la respuesta cuando el enunciado menciona "pago mediante factura" o "condiciones de crédito".

### Subcuentas

Las **subcuentas de facturación** existen para revendedores y para grandes organizaciones que necesitan separar la facturación por unidad de negocio. Cada subcuenta genera su propia factura, pero el cargo se consolida en la cuenta padre. Es la forma correcta de dar autonomía contable a una división sin fragmentar la relación comercial con Google.

### Roles de IAM en facturación

Esta separación de roles es de las cosas que más se preguntan, porque encarna el principio de separación de funciones: quien controla el gasto no es necesariamente quien administra los recursos.

|Rol|Qué permite|
|---|---|
|**Billing Account Creator**|Crear cuentas de facturación nuevas. Se concede a nivel de organización.|
|**Billing Account Administrator**|Gestionar una cuenta: pagos, presupuestos, exportaciones, vincular y desvincular proyectos. No da acceso a los recursos.|
|**Billing Account User**|Vincular proyectos a la cuenta. Es el rol mínimo que necesita un desarrollador para crear proyectos que facturen.|
|**Billing Account Viewer**|Ver costes y transacciones, sin modificar nada. El rol típico para finanzas y auditoría.|
|**Project Billing Manager**|Vincular o desvincular **ese proyecto** de una cuenta, sin controlar la cuenta en sí.|

El patrón habitual: para que alguien cree un proyecto y lo asocie a la facturación necesita **Project Creator** en la organización más **Billing Account User** en la cuenta. Las dos cosas por separado.

### Presupuestos y alertas

Un presupuesto define un importe — fijo o basado en el gasto del mes anterior — y unos umbrales que disparan notificaciones, normalmente al 50 %, 90 % y 100 %. El ámbito puede ser toda la cuenta, proyectos concretos, servicios o incluso filtros por label, que es donde se conecta con lo que vimos antes.

El punto crítico, y lo repito porque es la trampa más frecuente en los exámenes: **un presupuesto no detiene el gasto**. Es puramente informativo. Envía correos a los administradores de facturación, y opcionalmente publica en un tema de Pub/Sub.

Si querés un corte real, hay que construirlo: presupuesto → Pub/Sub → Cloud Function que llame a la API para **deshabilitar la facturación del proyecto**. Eso apaga los recursos y puede provocar pérdida de datos en discos y bases, así que es un mecanismo para entornos de prueba o límites de daño, no para producción. La alternativa preventiva y menos destructiva son las **cuotas**, que sí bloquean la creación de recursos antes de que el gasto ocurra.

### Exportación a BigQuery

Para cualquier análisis serio de coste, la vía es exportar los datos de facturación a BigQuery. Los informes de la consola sirven para exploración rápida, pero BigQuery permite consultas arbitrarias, unir con datos propios y construir paneles en Looker Studio.

Detalle importante: la exportación **no es retroactiva**. Los datos empiezan a acumularse el día que la activás, así que es una de las primeras cosas que se configuran en un proyecto nuevo. Hay tres exportaciones disponibles: coste estándar, coste detallado (con desglose a nivel de recurso) y datos de precios.

### Mecanismos de descuento

Vale la pena distinguirlos porque cada uno responde a una pista distinta:

- **Descuentos por uso sostenido (SUD):** automáticos, sin compromiso, en Compute Engine, por mantener instancias encendidas buena parte del mes.
- **Descuentos por compromiso de uso (CUD):** te comprometés a un consumo de CPU y memoria por 1 o 3 años a cambio de un descuento sustancial. Hay CUD basados en recursos y basados en gasto.
- **Spot VMs / preemptibles:** hasta un 60-91 % más baratas, pero Google puede reclamarlas en cualquier momento. Sólo para cargas tolerantes a interrupciones, como procesamiento por lotes.
- **Niveles gratuitos:** cada servicio tiene su cuota gratuita permanente, aparte de los créditos de prueba.
- **Recomendaciones del Active Assist / Recommender:** sugerencias automáticas de redimensionamiento, discos huérfanos e instancias inactivas.

### Para el examen

|Pista en el enunciado|Respuesta|
|---|---|
|Avisar cuando el gasto suba de un umbral|Presupuesto con alertas|
|**Detener** el gasto automáticamente|Presupuesto + Pub/Sub + Cloud Function que deshabilita la facturación|
|Analizar costes con SQL, paneles a medida|Exportación a BigQuery|
|Desglosar coste por equipo|Labels + informes de facturación|
|Pago mediante factura a 30 días|Cuenta facturada|
|Separar facturación por unidad de negocio|Subcuentas de facturación|
|Alguien de finanzas debe ver costes sin tocar recursos|Billing Account Viewer|
|Un dev debe crear proyectos que facturen|Project Creator + Billing Account User|
|Reducir coste con compromiso a largo plazo|CUD|
|Reducir coste en cargas por lotes interrumpibles|Spot VMs|

Un resumen de las tres herramientas de control que se confunden entre sí: **cuotas** previenen técnicamente, **presupuestos** informan, y **Organization Policy** restringe configuraciones. Ninguna sustituye a las otras.

Con esto tenés cubierta la sección de gestión de recursos y facturación. Si querés, el siguiente paso natural es IAM en detalle, o te armo un cuestionario de práctica que mezcle todo lo visto hasta ac
