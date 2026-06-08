# Postmortem: Degradação de Latência na API

## Status

Concluído

## Resumo

Em 05/06/2026, após o deploy das 17h30, a API apresentou degradação de latência em produção. O P95 saiu de aproximadamente 220ms para 5.200ms, enquanto o P99 chegou a 9.000ms. A taxa de erro também aumentou de 0,3% para 6,0%.

O impacto foi percebido por clientes através de timeouts e lentidão em endpoints críticos. A causa raiz identificada foi uma consulta N+1 introduzida em um endpoint crítico, agravada pela ausência de índice adequado em `users.account_id` e por diferenças entre o volume de dados de staging e produção.

Este postmortem é blameless. O objetivo é entender por que nossos sistemas permitiram que a falha chegasse à produção e quais mudanças reduzem a chance de recorrência.

## Impacto

- Clientes afetados por lentidão e timeouts na API.
- P95 aumentou de ~220ms para ~5.200ms.
- P99 chegou a ~9.000ms.
- Erros subiram de 0,3% para 6,0%.
- Houve aumento de registros no Sentry relacionados a timeout.
- O error budget mensal da API foi consumido de forma acelerada durante o incidente.

## Timeline

| Horário | Evento |
| --- | --- |
| 17:20 | API operando em patamar normal, P95 em 180ms e erro em 0,2% |
| 17:30 | Deploy realizado em produção |
| 17:35 | P95 sobe para 900ms e P99 para 1.800ms |
| 17:36 | Postgres registra queries acima de 2.600ms em `users.account_id` |
| 17:40 | P95 chega a 1.800ms e erros sobem para 2,5% |
| 17:45 | P95 chega a 5.200ms, P99 a 9.000ms e erros a 6,0% |
| 17:50 | Degradação permanece, com P95 em 4.800ms e erro em 5,5% |
| 18:00 | Incidente tratado como impacto real em cliente e rollback/mitigação priorizados |

## Causa raiz

A causa raiz foi uma consulta N+1 introduzida em um endpoint crítico. Para cada item processado, a aplicação passou a executar consultas adicionais contra a tabela `users`, usando `account_id` como filtro.

Em produção, o volume e a cardinalidade dos dados tornaram esse padrão caro. O log de slow queries mostra consultas repetidas como:

```sql
SELECT * FROM users u WHERE u.account_id = $1;
```

com duração entre 2.500ms e 5.100ms. Também havia ausência de índice adequado em `account_id`, o que ampliou o custo da consulta sob carga real.

## Fatores contribuintes

- Staging não refletia volume, cardinalidade e distribuição de dados de produção.
- Não havia teste automatizado de performance para endpoint crítico.
- A revisão de PR não tinha critério explícito para identificar risco de N+1.
- Não havia tracing distribuído funcional para correlacionar endpoint, chamada de banco e tempo gasto por dependência.
- Alertas ainda não estavam orientados por burn rate de SLO.
- A ausência de índice em `users.account_id` não foi detectada antes do deploy.

## Por que passou em staging

O problema passou em staging porque o ambiente não reproduzia as características que tornaram a falha visível em produção. A massa de dados era menor, a cardinalidade de `account_id` era diferente e o tráfego não representava concorrência real.

A consulta parecia aceitável em baixa escala, mas degradou rapidamente quando exposta ao volume de produção. Esse é um caso clássico em que teste funcional passa, mas o comportamento operacional falha.

## O que funcionou bem

- As APIs expunham métricas para Prometheus.
- Logs estruturados estavam disponíveis no Loki.
- O aumento de latência e erro pôde ser observado por métricas.
- O Sentry ajudou a perceber o aumento de timeouts.
- O deploy recente deu uma boa pista para reduzir o espaço de investigação.

## O que não funcionou bem

- A detecção não foi orientada por SLO.
- Faltou tracing funcional para identificar rapidamente o trecho exato da degradação.
- O ambiente de staging não representava dados e carga reais.
- Não havia gate de performance na pipeline.
- A ausência de índice não foi capturada em revisão ou teste automatizado.
- A resposta inicial dependia demais de investigação manual.

## Ações corretivas

| Ação | Dono | Prazo | Tipo |
| --- | --- | --- | --- |
| Corrigir o endpoint para eliminar o padrão N+1 | Engenharia Backend | 3 dias | Correção definitiva |
| Criar índice para `users.account_id`, após validação de plano de execução | Engenharia Backend / DBA | 3 dias | Correção definitiva |
| Adicionar teste de performance para endpoint crítico na pipeline | Engenharia Backend | 2 semanas | Prevenção |
| Incluir análise de queries e risco de N+1 no checklist de PR | Tech Lead | 1 semana | Processo |
| Ativar tracing distribuído com OpenTelemetry e backend de traces | SRE / Plataforma | 30 dias | Observabilidade |
| Criar alerta de burn rate para latência e erro da API | SRE | 2 semanas | Detecção |
| Revisar massa de dados de staging para refletir cardinalidade real | Engenharia / QA | 30 dias | Ambiente |

## Lições aprendidas

A falha não foi causada por uma pessoa específica, mas por lacunas no sistema de entrega: staging pouco representativo, ausência de gate de performance, falta de tracing e critérios insuficientes para revisão de consultas.

A principal mudança é tratar performance e observabilidade como parte do contrato de produção, não como validação posterior ao deploy.

## Prevenção de recorrência

Para reduzir recorrência, vamos combinar três frentes:

1. Detecção mais cedo, com alertas baseados em SLO e burn rate.
2. Prevenção na entrega, com teste de performance e revisão explícita de queries.
3. Diagnóstico mais rápido, com tracing funcional e correlação entre métricas, logs e banco.

## Encerramento

O incidente será considerado encerrado quando as ações corretivas principais forem concluídas, os alertas de SLO estiverem ativos e o endpoint corrigido for validado com massa de dados representativa.