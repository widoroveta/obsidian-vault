### Quiz: Gestión de Recursos (Resource Management)

**1. Ningún recurso en Google Cloud puede usarse sin estar asociado a...**

✅ **Un proyecto (A project)**

El proyecto es la unidad organizativa base de Google Cloud. Todo recurso —máquinas virtuales, buckets, bases de datos— debe pertenecer a un proyecto, porque el proyecto es lo que vincula el recurso con la facturación, los permisos de IAM, las APIs habilitadas y las cuotas. Sin proyecto no hay a quién cobrarle ni cómo controlar el acceso.

---

**2. ¿Cómo protegen las cuotas a los clientes de Google Cloud?**

✅ **Evitando el consumo descontrolado de recursos (By preventing uncontrolled consumption of resources)**

Las cuotas ponen un límite máximo al uso de cada servicio. Esto protege de dos cosas: errores de programación (por ejemplo, un bucle infinito que crea instancias sin parar) y de actores malintencionados que hayan obtenido acceso a tu cuenta. En ambos casos el daño económico queda limitado por la cuota.

---

**3. Presupuesto de $500 con alerta al 100%. ¿Qué pasa al gastar el monto completo?**

✅ **Se envía un correo de notificación al Administrador de Facturación (A notification email is sent to the Billing Administrator)**

Este es el punto clave del examen: **un presupuesto en Google Cloud NO detiene el gasto ni suspende recursos**. Es solo una herramienta de monitoreo y notificación. Los recursos siguen funcionando y el cobro continúa. Si quieres que algo se detenga automáticamente, debes configurarlo tú mismo conectando la alerta a Pub/Sub y una Cloud Function.

---

**Resumen de respuestas:** 1-C · 2-A · 3-A → 100% (aprobado, el mínimo es 66%)