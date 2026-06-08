# SLIs e SLOs

## Contexto

Eu usaria SLOs inicialmente em dois serviços piloto: a API Platform em C#, por ser o serviço user-facing, e o Core Processor em Python, por representar o processamento assíncrono baseado em fila.

A escolha dos dois pilotos cobre os dois tipos principais de experiência: resposta síncrona para usuário e conclusão de trabalho em background. Isso evita medir tudo apenas por disponibilidade HTTP, que não representa bem sistemas assíncronos.

## Serviço piloto 1: API Platform (C#)

A API Platform deve ser medida principalmente pela experiência percebida pelo cliente: disponibilidade, latência e taxa de erro.

| SLI | Como medir | SLO inicial |
| --- | --- | --- |
| Disponibilidade HTTP | Percentual de requests 2xx/3xx/4xx não considerados falha sobre total de requests | 99.5% ao mês |
| Latência P95 | P95 de duração das requisições HTTP | P95 menor que 500ms em janelas móveis de 5 minutos |
| Taxa de erro 5xx | Percentual de respostas 5xx sobre total de requests | Menor que 1% ao mês |

Exemplo de medição em Prometheus:

```promql
sum(rate(http_requests_received_total{job="csharp-api",code!~"5.."}[5m]))
/
sum(rate(http_requests_received_total{job="csharp-api"}[5m]))
```

## Serviço piloto 2: Core Processor (Python)

O Core Processor não deve ser medido apenas por health check HTTP. Como ele representa processamento em background, os sinais principais são backlog, idade da mensagem e sucesso no processamento.

| SLI | Como medir | SLO inicial |
| --- | --- | --- |
| Backlog da fila | Quantidade de mensagens prontas em `jobs` | Menos de 1.000 mensagens por mais de 10 minutos |
| Idade da mensagem | Tempo da mensagem mais antiga aguardando processamento | P95 abaixo de 5 minutos |
| Taxa de processamento com sucesso | Mensagens processadas com ACK sobre mensagens consumidas | 99% ao mês |

No cenário simulado de RabbitMQ, a fila `jobs` cresce de 10 para 12.000 mensagens enquanto a taxa de ACK cai até zero. Esse é um bom exemplo de por que backlog e ACK rate precisam ser SLIs do serviço, mesmo que a API continue respondendo health check.

## Error budget

Para a API Platform, um SLO de 99.5% ao mês representa aproximadamente 3h36min de indisponibilidade ou erro aceitável em um mês de 30 dias. Esse orçamento não deve ser visto como permissão para falhar, mas como limite operacional para tomada de decisão.

Se o serviço consumir error budget rápido demais, eu reduziria risco de mudança: congelamento temporário de deploys não críticos, rollback mais agressivo e priorização de correções de confiabilidade.

Para o Core Processor, o error budget deve considerar atraso de processamento e falhas de conclusão, não apenas disponibilidade do processo.

## Alertas por burn rate

Eu usaria alertas multi-window e multi-burn-rate para reduzir ruído. Um exemplo seria:

| Severidade | Janela | Condição | Ação |
| --- | --- | --- | --- |
| Alta | 5m e 1h | Consumo acelerado do error budget | Acionar on-call |
| Média | 30m e 6h | Degradação sustentada | Investigar durante horário operacional |
| Baixa | 24h | Tendência de consumo acima do esperado | Revisar capacidade ou regressões |

Essa abordagem evita alertar por picos curtos e concentra a atenção em degradações que ameaçam o SLO.