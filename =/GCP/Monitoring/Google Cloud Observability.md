### Google Cloud Observability

Es el conjunto de herramientas integradas para monitorear, depurar y entender el comportamiento de tus aplicaciones y tu infraestructura en Google Cloud. Antes se llamaba **Stackdriver**, luego _Google Cloud's operations suite_, y ahora **Google Cloud Observability**.

**Componentes principales:**

**Cloud Monitoring** — Recolecta métricas, eventos y metadatos. Te permite crear dashboards, definir _alerting policies_ (políticas de alerta) y usar _uptime checks_ para verificar que tus servicios respondan. Es lo que te avisa cuando el CPU de una VM se dispara o cuando un servicio deja de responder.

**Cloud Logging** — Almacena y permite consultar los logs de tus servicios. Puedes crear _log-based metrics_, exportar logs a BigQuery o Cloud Storage (_sinks_) y definir cuánto tiempo se retienen.

**Error Reporting** — Agrupa automáticamente los errores de tus aplicaciones y te notifica cuando aparece uno nuevo, en lugar de que tengas que buscarlo entre miles de líneas de log.

**Cloud Trace** — Rastreo distribuido (_distributed tracing_). Muestra cuánto tarda cada parte de una petición al recorrer tus microservicios, útil para encontrar cuellos de botella de latencia.

**Cloud Profiler** — Analiza el consumo de CPU y memoria del código en producción, con bajo impacto en el rendimiento, para identificar funciones ineficientes.

---

**Los tres pilares de la observabilidad** (concepto que suele aparecer en el examen):

|Pilar|Herramienta|
|---|---|
|Métricas (_metrics_)|Cloud Monitoring|
|Registros (_logs_)|Cloud Logging|
|Trazas (_traces_)|Cloud Trace|

**Diferencia clave para recordar:** _monitoring_ te dice **que** algo está mal (una métrica se salió del rango); _observability_ te da las herramientas para entender **por qué** está mal, incluso en fallas que nunca anticipaste.

Si me pasas las preguntas del quiz de esta sección, te las resuelvo igual que las anteriores