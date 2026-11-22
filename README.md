# itsm-infra

Infraestructura de eventos del SaaS ITSM, desplegada en AWS mediante Serverless Framework 3.

## Estado del despliegue

- **Entorno:** `dev`, región `us-east-1`.
- **Stack CloudFormation:** `itsm-infra-dev`.
- **Estado:** desplegado y operativo.
- **Mensajería:** un tópico SNS, cinco colas SQS y sus respectivas DLQ.
- **Reintentos:** hasta tres recepciones antes de enviar a DLQ.
- **Enrutamiento:** suscripciones SNS con filtros por atributo `evento`.
- **Observabilidad:** alarmas CloudWatch asociadas a las DLQ.
- **Configuración compartida:** ARN publicados en AWS Systems Manager Parameter Store.

Se verificaron dos circuitos de integración de extremo a extremo:

1. `Incidentes -> Problemas -> Cambios -> Problemas -> Incidentes`.
2. `Solicitudes -> Cambios -> Solicitudes`.

Los eventos se procesan de forma asíncrona mediante SNS, SQS y consumidores Lambda.

El consumidor de Problemas estuvo deshabilitado durante una prueba y fue reactivado. Tras procesar los mensajes pendientes, el problema y el incidente vinculados quedaron resueltos. Se debe comprobar que los event source mappings permanezcan habilitados.

**Precaución:** no volver a desplegar ni eliminar este stack durante el cierre del proyecto sin revisar los cambios de infraestructura. Las colas y sus suscripciones contienen la configuración compartida por los microservicios.

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
