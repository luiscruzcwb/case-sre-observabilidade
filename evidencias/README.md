# Evidências de Validação

Esta pasta contém evidências coletadas durante a validação local da stack.

- Prometheus coletando métricas das APIs.
- Grafana consultando logs no Loki.
- RabbitMQ operacional com fila `jobs`.
- APIs acessando Postgres e Redis com sucesso, validado via endpoints `/db/ping` e `/cache/ping`.
- OTel Collector ativo, ainda sem backend real de tracing.
- Evidência de que o OpenTelemetry Collector está ativo e recebe OTLP nas portas 4317/4318, mas ainda exporta traces apenas para `debug`. Isso confirma que existe base de instrumentação, porém ainda não há backend real de tracing configurado.


