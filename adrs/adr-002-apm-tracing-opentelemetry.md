# ADR-002: OpenTelemetry como padrão para APM e tracing

**Status:** Accepted

## Contexto

As aplicações C# e Python já possuem configuração inicial de OpenTelemetry, e o ambiente inclui um OTel Collector. Porém, o collector exporta apenas para `debug` e não há backend real de tracing configurado.

## Drivers de decisão

- Suporte a C# e Python.
- Redução de lock-in.
- Possibilidade de trocar backend sem reinstrumentar aplicações.
- Evolução gradual para tracing distribuído.

## Opções consideradas

1. Instrumentação direta em vendor comercial
   - Prós: adoção rápida e experiência pronta.
   - Contras: maior lock-in e custo potencialmente menos previsível.

2. OpenTelemetry com backend gerenciado ou open source
   - Prós: padrão aberto, flexível e compatível com múltiplos backends.
   - Contras: exige padronização de atributos e configuração do collector.

3. Não adotar tracing neste momento
   - Prós: menor esforço imediato.
   - Contras: diagnóstico mais lento em incidentes envolvendo múltiplos serviços.

## Decisão

Adotar OpenTelemetry como padrão de instrumentação e executar um piloto com backend de tracing compatível, preferencialmente integrado ao ecossistema Grafana ou APM com bom suporte a OTel.

## Consequências

A arquitetura fica menos acoplada a fornecedor e preparada para evolução. O principal cuidado é definir sampling, atributos obrigatórios e governança de custo desde o início.