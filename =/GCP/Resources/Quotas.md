## Cuotas y límites en GCP

Las cuotas son los límites que Google Cloud impone sobre el consumo de recursos y el uso de APIs. Existen por dos razones que conviene distinguir, porque explican todo el comportamiento del sistema: **proteger la infraestructura compartida** de un consumo abusivo o de un vecino ruidoso, y **protegerte a ti** de un gasto descontrolado por un error de código o un bucle infinito.

### Los dos tipos

**Cuotas de tasa (rate quotas)**, también llamadas de uso de API. Limitan el número de peticiones en una ventana de tiempo — por ejemplo, X llamadas a la API de Compute Engine por minuto. **Se reinician automáticamente** al cerrarse la ventana (cada minuto, cada día). Al agotarse, la API devuelve un error `429 RESOURCE_EXHAUSTED` o `rateLimitExceeded`.

**Cuotas de asignación (allocation quotas)**, o de recursos. Limitan cuántos recursos podés tener existiendo a la vez: número de VMs, direcciones IP estáticas, CPUs, subredes, reglas de firewall. **No se reinician**: para liberar cuota hay que borrar recursos. El error es `quotaExceeded`.

La diferencia importa porque la reacción es distinta. Ante una cuota de tasa agotada, la solución es reintentar con retroceso exponencial (_exponential backoff_) — el sistema se recuperará solo. Ante una de asignación, reintentar no sirve de nada: hay que liberar recursos o pedir más cuota.

### Ámbito

Casi todas las cuotas se aplican **por proyecto**, y muchas además **por región o por zona**. Eso significa que una misma cuota se cuenta de forma independiente en `us-central1` y en `europe-west1`, y es una fuente habitual de confusión: te queda cuota de CPUs global pero no en la zona donde intentás crear la VM.

Este carácter por proyecto es una de las razones estructurales para dividir la infraestructura en varios proyectos, como mencionaba con el Resource Manager: separar producción de desarrollo aísla las cuotas, de modo que un experimento en dev no puede agotar la capacidad de prod.

Algunas cuotas nuevas se aplican a nivel de organización o carpeta, y existen los **quota overrides** para ajustar límites por debajo del máximo concedido — útil como freno de seguridad deliberado.

### Cómo gestionarlas

En la consola, **IAM y administración → Cuotas y límites del sistema**, o con `gcloud`:

```
gcloud compute project-info describe --project MI_PROYECTO
gcloud alpha services quota list --service=compute.googleapis.com \
  --consumer=projects/MI_PROYECTO
```

Para aumentar una cuota se envía una solicitud desde esa misma página. Consideraciones prácticas:

- Las solicitudes pequeñas suelen aprobarse de forma automática o en minutos; las grandes pasan por revisión humana y pueden tardar **de dos a tres días laborables**.
- Justificar el pedido con el caso de uso y el patrón de crecimiento acelera la aprobación.
- Las cuentas de **prueba gratuita** tienen cuotas muy reducidas y en general **no pueden aumentarlas** hasta convertirse en cuenta de pago. Esto aparece bastante en los exámenes.
- El historial de facturación influye: una cuenta con consumo consolidado obtiene aumentos más grandes con menos fricción.

El permiso necesario es `serviceusage.quotas.update`, incluido en el rol **Quota Administrator** (`roles/servicemanagement.quotaAdmin`) o en Project Owner/Editor.

### Cuotas frente a límites

No son lo mismo, y la distinción es exactamente el tipo de matiz que se pregunta:

- Una **cuota** es ajustable. Podés solicitar más.
- Un **límite del sistema** es fijo, definido por la arquitectura del servicio, y no se puede aumentar por ningún medio. Ejemplos: el tamaño máximo de un documento de Firestore (1 MiB), el máximo de reglas de firewall por red, el tope de niveles de anidamiento de carpetas.

### Cuotas frente a presupuestos

Otro par que se confunde. Las cuotas son un **control técnico** que bloquea la operación cuando se alcanza el límite. Los **presupuestos y alertas** de Cloud Billing son **informativos**: notifican por correo o Pub/Sub cuando el gasto cruza un umbral, pero **no detienen nada por sí solos**. Si querés un corte real de gasto, hay que combinar la alerta de presupuesto con una función de Cloud Functions que deshabilite la facturación del proyecto — y eso apaga los recursos, con las consecuencias que eso tiene.

Por eso, para evitar sorpresas en la factura, la cuota es la herramienta preventiva y el presupuesto la de vigilancia. Se usan juntas.

### Buenas prácticas

- Monitorizar el consumo con Cloud Monitoring y crear alertas sobre las métricas de cuota **antes** de llegar al 100 %, no cuando ya falló el despliegue.
- Pedir aumentos con antelación si hay un lanzamiento, una migración o un pico estacional previsto. Nadie quiere descubrir el límite un viernes a las seis.
- Implementar retroceso exponencial con _jitter_ en todo cliente que llame a APIs de Google.
- Bajar deliberadamente las cuotas en proyectos de desarrollo o de estudiantes como límite de daño.

### Para el examen

|Pista en el enunciado|Respuesta|
|---|---|
|Error 429 / rateLimitExceeded|Cuota de tasa; reintentar con backoff exponencial|
|No puedo crear más VMs|Cuota de asignación; borrar recursos o pedir aumento|
|Necesito más capacidad para un lanzamiento|Solicitar aumento de cuota con antelación|
|Quiero que me avisen del gasto|Presupuestos y alertas de facturación|
|Quiero impedir técnicamente el exceso|Cuotas (u Organization Policy)|
|Aislar el consumo entre entornos|Proyectos separados|
|El límite no se puede subir de ninguna forma|Límite del sistema, no cuota|

Si querés, puedo armarte un cuestionario de práctica que mezcle esta sección con Resource Manager e IAM, que es como suelen venir agrupadas en el examen.