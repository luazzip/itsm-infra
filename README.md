# itsm-infra
Bus de eventos del SaaS ITSM: 1 tema SNS, 5 colas SQS con DLQ (maxReceiveCount 3),
filter policies por el atributo `evento`, ARN publicados en SSM y alarmas CloudWatch de las DLQ.

## Despliegue
    serverless deploy              # stage dev
    serverless deploy --stage prod

## Parámetros SSM
/itsm/{stage}/sns-eventos-arn, /itsm/{stage}/q-incidentes-arn, q-problemas-arn, q-cambios-arn, q-solicitudes-arn, q-notificaciones-arn

## Filtros
| Cola | Eventos |
|---|---|
| q-incidentes | problema.resuelto |
| q-problemas | incidente.escalado_a_problema, cambio.implementado |
| q-cambios | problema.causa_raiz_identificada, solicitud.cambio_estandar |
| q-solicitudes | cambio.implementado |
| q-notificaciones | todos (sin filtro) |
