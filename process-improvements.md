# Melhorias de Processo

## Contexto

As melhorias propostas têm como objetivo reduzir recorrência de incidentes como degradação por N+1 query, fila acumulada e falha silenciosa em dependências. Eu priorizaria mudanças que aumentem a segurança de deploy sem travar o ritmo de entrega.

A abordagem inicial seria pragmática: poucos gates bem escolhidos, critérios claros de observabilidade em PRs e resposta a incidentes com donos definidos.

## Gates na pipeline

Os gates de pipeline cobririam mudanças com maior risco operacional:

- teste de performance para endpoints críticos;
- validação de consultas em mudanças que acessam banco;
- checagem de migrações e criação de índices;
- smoke test pós-deploy em produção;
- validação automática de health checks e métricas expostas.

O objetivo não é bloquear todo deploy com uma bateria pesada, mas identificar cedo mudanças que possam consumir error budget de forma acelerada.

## Observabilidade como critério de PR

Toda mudança relevante deveria responder algumas perguntas antes de ir para produção:

- quais métricas mostram que a mudança está saudável;
- quais logs ajudam a investigar falha;
- se existe `request_id` ou correlação suficiente para diagnóstico;
- se a mudança afeta latência, erro, fila ou dependência externa;
- se há risco de cardinalidade alta em métricas ou logs;
- se existe runbook ou procedimento de rollback.

Esse checklist evita que observabilidade seja tratada como algo posterior ao deploy.

## Estratégia de deploy

Para mudanças de maior risco, o caminho preferencial seria rollout progressivo com canary ou blue-green, começando por pequena parcela de tráfego e expandindo conforme métricas de SLO permanecem saudáveis.

Para mudanças com risco de performance, eu usaria feature flag sempre que possível. Isso permite desligar comportamento novo sem depender de rollback completo.

O rollback deve ser simples, testado e aceito culturalmente como mecanismo normal de proteção do cliente.

## Alertas baseados em SLO

Os alertas principais devem ser baseados em SLO e burn rate. Isso reduz fadiga porque prioriza degradações com impacto real.

Eu separaria os alertas em três grupos:

| Tipo | Exemplo | Ação |
| --- | --- | --- |
| Página imediata | Latência ou erro consumindo error budget rapidamente | Acionar on-call |
| Investigação | Degradação sustentada sem impacto crítico imediato | Abrir análise durante horário operacional |
| Informativo | CPU, memória ou logs acima do normal | Usar como contexto em dashboard |

Essa separação reduz ruído e deixa claro quando alguém precisa agir fora do horário comercial.

## Runbooks e resposta a incidentes

Os runbooks curtos cobririam os cenários mais prováveis:

- API lenta após deploy;
- fila RabbitMQ acumulando;
- saturação de conexões no Postgres;
- Redis com timeout;
- aumento de 5xx ou timeouts no Sentry.

Cada runbook deve ter sintomas, impacto, comandos de diagnóstico, mitigação imediata, critério de rollback e links para dashboards.

## Governança e acompanhamento

As ações de postmortem devem ter dono, prazo e acompanhamento. Sem isso, postmortem vira apenas documentação.

A revisão mensal acompanharia:

- consumo de error budget;
- incidentes recorrentes;
- alertas mais ruidosos;
- tempo médio de detecção e mitigação;
- ações corretivas vencidas;
- custo de observabilidade versus utilidade dos sinais.

O objetivo é criar um ciclo simples: medir, aprender, corrigir e revisar.