![[Pasted image 20260909233914.png]]
### Cloud Monitoring

Es el servicio que recolecta **métricas** de tu infraestructura y aplicaciones para que puedas ver su estado en tiempo real y recibir alertas cuando algo se desvía de lo normal.

**Cómo obtiene los datos**

Muchas métricas llegan automáticamente sin que hagas nada: uso de CPU, tráfico de red, operaciones de disco de tus VMs, peticiones a load balancers, etc. Pero para métricas _dentro_ de la máquina —memoria usada, espacio en disco, procesos— necesitas instalar el **Ops Agent** en la VM. Es un punto que suele preguntarse: sin agente, no ves memoria ni disco desde adentro.

También puedes enviar tus propias métricas de negocio (_custom metrics_), por ejemplo "carritos abandonados por minuto".

**Conceptos clave**

**Workspace / Scoping project** — El contenedor donde se agrupan los datos. Un solo scoping project puede monitorear varios proyectos a la vez, para tener una vista unificada de todo tu entorno.

**Dashboards** — Paneles con gráficas. Google incluye dashboards predefinidos por servicio, y puedes crear los tuyos.

**Alerting policies** — La regla que define cuándo algo está mal. Tiene tres partes:

- **Condición**: qué métrica y qué umbral (ej. CPU > 80% durante 5 minutos)
- **Notification channels**: cómo te avisa — email, SMS, Slack, PagerDuty, Pub/Sub, webhook
- **Documentación**: texto que se incluye en la alerta para que quien la reciba sepa qué hacer

**Uptime checks** — Sondas que Google envía desde distintas regiones del mundo hacia tu URL o IP para confirmar que responde. Si falla, dispara una alerta. Sirven para detectar caídas desde la perspectiva del usuario real, no solo desde adentro.

**SLIs, SLOs y SLAs** (aparecen mucho en preguntas):

|Término|Qué es|
|---|---|
|**SLI** (_Indicator_)|La medición concreta: "99.8% de peticiones exitosas"|
|**SLO** (_Objective_)|La meta interna que te pones: "≥99.9% de disponibilidad"|
|**SLA** (_Agreement_)|El contrato con el cliente, con penalización si se incumple|

Regla práctica: el SLO siempre debe ser más estricto que el SLA, para tener margen antes de romper el contrato.

---

**Para recordar en el examen:** Monitoring = métricas y alertas (¿está sano?). Logging = registros de eventos (¿qué pasó exactamente?). Son servicios distintos aunque trabajen juntos.

Pásame el quiz de esta sección si lo tienes y te lo resuelvo con explicación.
![[Pasted image 20260909234445.png]]