# ADR-003: Retenção de logs e controle de cardinalidade

**Status:** Accepted

## Contexto

Loki já está configurado com retenção de 7 dias e recebe logs das aplicações via Promtail. Logs são úteis para investigação, mas podem se tornar caros e difíceis de consultar quando há excesso de volume ou labels de alta cardinalidade.

## Drivers de decisão

- Controlar custo de storage e ingestão.
- Preservar logs úteis para incidentes.
- Evitar degradação de consulta no Loki.
- Separar logs investigativos de métricas de alerta.

## Opções consideradas

1. Aumentar retenção de todos os logs
   - Prós: mais histórico disponível.
   - Contras: custo maior e mais ruído.

2. Manter retenção curta para logs brutos e ampliar apenas eventos críticos
   - Prós: equilíbrio entre custo e utilidade.
   - Contras: exige classificação mínima dos eventos.

3. Reduzir logs ao mínimo
   - Prós: menor custo.
   - Contras: perda de contexto em incidentes.

## Decisão

Manter retenção curta para logs brutos operacionais e controlar labels de alta cardinalidade. Logs com necessidade de auditoria ou investigação estendida devem ser tratados separadamente.

## Consequências

A decisão reduz custo e melhora a qualidade das consultas. O risco é perder logs antigos de incidentes descobertos tardiamente, mitigado por postmortems rápidos e retenção diferenciada para eventos críticos.