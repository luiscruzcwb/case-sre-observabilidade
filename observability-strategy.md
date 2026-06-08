# Estratégia de Observabilidade

## Contexto atual

A stack atual já cobre uma base importante de observabilidade: Prometheus coleta métricas das APIs, Grafana está provisionado com Prometheus e Loki, e os serviços em C# e Python já expõem health checks e métricas. Durante a validação local, as duas APIs responderam corretamente, Postgres e Redis foram acessados com sucesso pelas aplicações, e Loki retornou logs estruturados dos dois serviços.

A stack ainda está em estágio inicial de maturidade operacional. Permite enxergar sinais básicos, mas ainda não garante investigação rápida de incidentes, correlação entre logs, métricas e traces, nem alertas orientados por SLO.

## O que eu manteria

Eu manteria Prometheus, Grafana e Loki como base inicial da operação. A validação local mostrou que Prometheus já coleta métricas das duas APIs, Grafana já está provisionado com Prometheus e Loki, e os logs das aplicações aparecem no Explore do Grafana.

Prometheus atende bem para métricas operacionais e SLIs simples, como disponibilidade, latência e taxa de erro. Grafana segue como ponto central de visualização. Loki faz sentido para logs estruturados porque já está integrado via Promtail e evita introduzir mais uma ferramenta neste primeiro momento.

Também manteria OpenTelemetry como padrão de instrumentação, mas não consideraria tracing como pronto. O collector está de pé, porém exporta apenas para `debug` e ainda não há backend de tracing configurado.

## O que eu melhoraria

O primeiro ajuste seria melhorar a qualidade dos sinais antes de aumentar a quantidade de dashboards e alertas. Hoje existem métricas HTTP e logs estruturados, mas ainda falta padronização de labels, correlação entre request_id, logs e traces, e dashboards orientados por jornada de serviço.

Eu priorizaria dashboards por serviço com quatro sinais principais: latência, tráfego, erros e saturação. Para a API C#, o foco seria experiência do usuário: P95/P99, 5xx e disponibilidade. Para o processamento em Python, o foco seria fila, backlog, idade da mensagem, taxa de ACK e falhas em dependências como Postgres e Redis.

Também revisaria métricas com label `path`, porque em ambientes reais isso pode gerar cardinalidade alta se parâmetros dinâmicos forem expostos como parte do path.

## Redução de alert fatigue

Eu evitaria alertas para toda anomalia técnica isolada. O modelo inicial seria separar alertas acionáveis de sinais apenas informativos. Um alerta precisa ter impacto claro, dono claro e ação esperada.

A prioridade seria alertar por sintomas percebidos pelo usuário ou pelo processamento: aumento sustentado de erro, latência acima do SLO, fila crescendo sem consumo, ou dependência saturada afetando serviço. Métricas como CPU alta, memória ou volume de logs entram como contexto de diagnóstico, não necessariamente como página para on-call.

Também adotaria alertas por burn rate de error budget. Isso reduz ruído porque o alerta passa a considerar impacto acumulado no SLO, e não apenas um pico curto que se recupera sozinho.

## Evolução para distributed tracing

A base para tracing já existe, porque as aplicações possuem configuração OpenTelemetry e o collector está presente no ambiente. Porém, a validação mostrou que o collector exporta apenas para `debug` e não há backend como Tempo, Jaeger, Datadog ou New Relic.

Eu trataria tracing como uma evolução em fases. Primeiro, corrigiria a comunicação dos serviços com o collector e padronizaria atributos como `service.name`, ambiente, versão e request_id. Depois, adicionaria um backend de traces e começaria por endpoints críticos da API e fluxos assíncronos envolvendo RabbitMQ.

O objetivo não seria coletar 100% dos traces desde o início. Eu usaria sampling, mantendo maior taxa para erros, endpoints críticos e transações lentas. Isso preserva capacidade de investigação sem criar custo desnecessário.

## Controle de custo

O controle de custo (FinOps) começa por retenção, cardinalidade e volume de ingestão. Para logs no Loki, eu manteria retenção curta para logs brutos operacionais e aumentaria retenção apenas para eventos úteis em auditoria, investigação ou compliance.

Em métricas, eu evitaria labels de alta cardinalidade, como IDs de cliente, request_id, payload ou paths dinâmicos. Em traces, adotaria sampling desde o início e preservaria amostras completas para erros e latências fora do padrão.

Também separaria sinais por finalidade: métricas para alertas e SLOs, logs para investigação contextual, traces para entender fluxo entre serviços. Quando cada sinal tem função clara, a tendência é coletar menos dado inútil e operar com menos ruído.

## Roadmap inicial

Nos primeiros 30 dias, o foco seria estabilizar a base: validar coleta de métricas e logs, criar dashboards mínimos por serviço, revisar labels de métricas e documentar runbooks para API lenta, fila acumulada e falhas em dependências.

Entre 30 e 60 dias, a meta seria implantar SLIs e SLOs para dois serviços piloto: API Platform em C# e Core Processor em Python. Também moveria alertas críticos para burn rate e reduziria alertas puramente técnicos que não exigem ação imediata.

Entre 60 e 90 dias, o avanço seria para tracing distribuído com OpenTelemetry, começando pelos fluxos mais críticos. A escolha do backend de tracing deve considerar custo, integração com Azure/AKS, suporte a C# e Python, e capacidade do time de operar a solução.

## Evidências de validação local

Durante a validação local, foram coletadas evidências em `entrega/evidencias/` mostrando Prometheus com targets saudáveis, Loki recebendo logs das APIs, RabbitMQ operacional e OTel Collector ativo apenas com exporter `debug`.