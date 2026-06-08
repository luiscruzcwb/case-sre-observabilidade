# ADR-001: Alertas baseados em SLO e burn rate

**Status:** Accepted

## Contexto

A stack atual já permite coletar métricas das APIs via Prometheus, mas alertas puramente técnicos tendem a gerar ruído e fadiga operacional. Para uma operação SRE inicial, o mais importante é diferenciar sintomas com impacto real de sinais usados apenas para diagnóstico.

## Drivers de decisão

- Reduzir alert fatigue.
- Priorizar impacto em cliente e processamento.
- Criar critérios objetivos para acionar on-call.
- Conectar alertas ao consumo de error budget.

## Opções consideradas

1. Alertas por métricas técnicas isoladas
   - Prós: simples de implementar.
   - Contras: alto risco de ruído e baixa relação com impacto real.

2. Alertas baseados em SLO e burn rate
   - Prós: prioriza impacto real e consumo acelerado de error budget.
   - Contras: exige definição prévia de SLIs/SLOs e maturidade de medição.

3. Alertas manuais por dashboard
   - Prós: baixo esforço inicial.
   - Contras: detecção tardia e dependência de observação humana.

## Decisão

Adotar alertas baseados em SLO e burn rate como padrão para incidentes críticos. Métricas técnicas como CPU, memória e volume de logs continuam disponíveis em dashboards, mas não devem acionar on-call automaticamente sem relação clara com impacto.

## Consequências

Essa decisão reduz ruído e melhora a qualidade dos acionamentos. O risco é demorar mais para configurar alertas maduros, mas isso é mitigado começando com dois serviços piloto: API Platform e Core Processor.