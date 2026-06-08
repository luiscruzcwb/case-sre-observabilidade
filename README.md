# Case SRE: Observabilidade, SLOs e Resposta a Incidente

Estudo de caso de SRE a partir de um cenário técnico: uma API que degradou de ~200ms para ~5s em produção numa sexta-feira às 18h. O material percorre o ciclo completo, do diagnóstico do incidente à prevenção de recorrência, passando por estratégia de observabilidade, definição de SLIs/SLOs, recomendação de APM/tracing e análise de custo (FinOps).

🔗 **Versão navegável:** [luiscruz.com.br/case-sre](https://luiscruz.com.br/case-sre)

## O incidente em números

| Métrica | Antes | Depois |
| --- | --- | --- |
| Latência P95 | 220ms | 5.200ms (~24×) |
| Latência P99 | — | 9.000ms |
| Taxa de erro | 0,3% | 6,0% |

Causa raiz: consulta N+1 em um endpoint crítico, agravada pela ausência de índice em `account_id` e por um staging que não reproduzia o volume de produção.

## Conteúdo

Ordem de leitura sugerida:

1. [`observability-strategy.md`](observability-strategy.md) — o que manter, o que melhorar, redução de alert fatigue e evolução para tracing
2. [`slo-targets.md`](slo-targets.md) — SLIs, SLOs, error budget e alertas multi-window por burn rate
3. [`apm-recommendation.md`](apm-recommendation.md) — OpenTelemetry + backend de tracing, com trade-offs e ROI
4. [`incident-response-plan.md`](incident-response-plan.md) — resposta nos primeiros 5/15/60 minutos e critério de rollback
5. [`postmortem-example.md`](postmortem-example.md) — postmortem blameless com timeline e ações corretivas
6. [`process-improvements.md`](process-improvements.md) — gates de pipeline, observabilidade como critério de PR e governança
7. [`cost-analysis.csv`](cost-analysis.csv) e [`cost-analysis-alavancas.csv`](cost-analysis-alavancas.csv) — custo atual vs proposto e alavancas de redução
8. [`adrs/`](adrs/) — decisões arquiteturais registradas
9. [`evidencias/`](evidencias/) — prints da validação local da stack

## Stack

Prometheus · Grafana · Loki · OpenTelemetry · RabbitMQ · PostgreSQL · Redis. Serviços em C# e Python sobre Kubernetes (AKS).

## Nota

Produzi este material como resposta a um desafio técnico de SRE. Os dados de slow query e de fila vêm dos cenários do exercício; as métricas do incidente (P95/P99 e taxas de erro) são ilustrativas, coerentes com o cenário; os custos são estimativas, a calibrar com dados reais. Conteúdo independente, sem vínculo com qualquer empresa nem com dados de produção reais.

---

Luis Cruz · DevOps & SRE · [luiscruz.com.br](https://luiscruz.com.br)
