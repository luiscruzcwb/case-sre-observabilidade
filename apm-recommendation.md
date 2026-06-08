# Recomendação de APM e Distributed Tracing

## Contexto

A stack atual já possui OpenTelemetry nas aplicações e um OTel Collector no ambiente. Na validação local, o collector subiu corretamente e abriu os receivers OTLP em 4317 e 4318, mas a configuração atual exporta traces apenas para `debug`. Também não existe datasource de tracing no Grafana.

Por isso, eu não trataria tracing distribuído como capacidade pronta. A decisão principal é manter OpenTelemetry como padrão de instrumentação e escolher um backend que ajude na investigação sem criar lock-in excessivo ou custo difícil de controlar.

## Critérios de decisão

Os critérios que eu usaria para escolher a solução são:

| Critério | Peso | Motivo |
| --- | --- | --- |
| Integração com C# e Python | Alto | Os dois runtimes fazem parte do caminho crítico do desafio |
| Suporte a OpenTelemetry | Alto | Evita acoplamento direto a vendor |
| Custo previsível | Alto | Observabilidade pode ficar cara rapidamente com logs e traces |
| Facilidade operacional | Alto | O cenário assume início de maturidade SRE, possivelmente com time pequeno |
| Integração com Azure/AKS | Médio | Importante para evolução futura da plataforma |
| Correlação métricas/logs/traces | Alto | Reduz MTTR em incidentes reais |
| Lock-in | Médio | Deve ser controlado, mas não pode impedir evolução |

## Opções avaliadas

| Opção | Pontos fortes | Riscos / limitações |
| --- | --- | --- |
| Datadog | Excelente experiência operacional, boa correlação entre métricas/logs/traces, forte suporte a Kubernetes e cloud | Custo pode crescer rápido com ingestão, hosts, logs e APM; exige governança desde o início |
| New Relic | Boa cobertura de APM, dashboards e experiência para times de produto/engenharia | Modelo de custo por usuário/ingestão precisa ser acompanhado; menor controle se usado sem OTel |
| Dynatrace | Forte em ambientes enterprise, automação e análise de dependências | Pode ser pesado e caro para um estágio inicial; maior complexidade de adoção |
| Open Source: Tempo/Jaeger + OTel | Menor lock-in, bom encaixe com Grafana existente, controle sobre instrumentação | Aumenta responsabilidade operacional; exige cuidado com storage, retenção, upgrades e escala |

## Recomendação

Minha recomendação seria manter OpenTelemetry como padrão obrigatório de instrumentação e executar um piloto controlado com backend de tracing integrado ao ecossistema Grafana, como Grafana Tempo ou Grafana Cloud Traces. Essa escolha aproveita o que já existe no ambiente, reduz mudança operacional e mantém a arquitetura preparada para trocar de backend no futuro.

Dynatrace não seria o ponto de partida neste momento, por ser uma solução mais pesada para uma operação ainda em formação. Datadog seria uma opção forte caso a prioridade da empresa seja velocidade de adoção e menor esforço operacional, mas eu só adotaria com limites claros de ingestão, retenção e sampling desde o primeiro mês.

A decisão não é puramente técnica. Para um primeiro ciclo, eu priorizaria uma solução que entregue investigação ponta a ponta sem tornar a conta de observabilidade imprevisível.

## Trade-offs aceitos

Ao manter OpenTelemetry como camada neutra, aceitamos investir um pouco mais em padronização de atributos, propagação de contexto e configuração do collector. Em troca, reduzimos lock-in e mantemos liberdade para trocar o backend de tracing.

Ao preferir um piloto com Grafana Tempo/Grafana Cloud ou solução compatível com OTel, aceitamos que algumas funcionalidades prontas de plataformas comerciais podem demorar mais para aparecer. O ganho é evoluir em passos menores, aproveitando Grafana, Prometheus e Loki já presentes no ambiente.

Se a empresa decidir por Datadog ou New Relic, o trade-off aceito seria pagar mais por uma experiência operacional mais pronta, desde que exista governança de custo.

## Roadmap de adoção em 3 meses

| Período | Ação | Resultado esperado |
| --- | --- | --- |
| Mês 1 | Corrigir configuração do OTel Collector, padronizar `service.name`, ambiente, versão e propagação de contexto | Traces chegando com metadados consistentes |
| Mês 1 | Instrumentar endpoints críticos da API C# e fluxos principais da API Python | Primeira visão de latência por rota e dependência |
| Mês 2 | Adicionar backend de tracing e datasource no Grafana | Correlação entre métricas, logs e traces |
| Mês 2 | Aplicar sampling para tráfego normal e retenção maior para erros/lentidão | Controle de custo desde o início |
| Mês 3 | Criar dashboards e runbooks baseados em traces para incidentes recorrentes | Redução de tempo de diagnóstico |
| Mês 3 | Revisar ROI e decidir expansão ou troca de backend | Decisão baseada em dados reais |

## Como medir ROI

O retorno da adoção seria medido por indicadores operacionais, não por quantidade de dashboards criados.

Os principais indicadores seriam:

| Indicador | Como medir |
| --- | --- |
| MTTD | Tempo entre início da degradação e detecção pelo time |
| MTTR | Tempo entre detecção e mitigação |
| Tempo de diagnóstico | Tempo para identificar serviço, endpoint ou dependência causadora |
| Incidentes recorrentes | Quantidade de incidentes repetidos pelo mesmo padrão |
| Uso de error budget | Consumo mensal antes e depois da adoção |
| Custo por GB ingerido | Custo mensal dividido por volume útil de logs/traces/métricas |

A ferramenta só se justifica se reduzir tempo de investigação e melhorar decisões de deploy, rollback e priorização de confiabilidade.