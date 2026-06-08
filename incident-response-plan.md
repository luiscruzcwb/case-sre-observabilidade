# Plano de Resposta a Incidente

## Contexto

Cenário: sexta-feira, 18h. A latência da API subiu de aproximadamente 200ms para 5s, o Sentry começou a registrar muitos timeouts e clientes já estão reclamando. Staging segue normal e houve deploy às 17h30.

Eu trataria inicialmente como incidente de produção com impacto em cliente. A prioridade nos primeiros minutos não é encontrar a causa raiz definitiva, e sim reduzir impacto, organizar comunicação e preservar evidências suficientes para investigação posterior.

## Objetivos da resposta

Os objetivos imediatos são:

- confirmar impacto real e escopo;
- estabilizar a experiência do cliente;
- evitar novas mudanças no ambiente durante a investigação;
- decidir rollback com critério objetivo;
- manter stakeholders informados;
- preservar evidências para postmortem.

## Primeiros 5 minutos

Nos primeiros 5 minutos, as ações seriam:

1. Declarar incidente e assumir coordenação como on-call, iniciando uma war room.
2. Congelar deploys não relacionados até estabilização.
3. Validar se o problema afeta todos os clientes ou apenas uma rota, tenant ou região, checando também o status das ferramentas contratadas.
4. Checar dashboards de latência, taxa de erro, throughput e saturação de dependências.
5. Confirmar horário do deploy, versão atual e versão anterior.
6. Abrir canal de incidente com engenharia, suporte e liderança técnica.
7. Publicar primeira comunicação curta: impacto observado, investigação em andamento e próximo update.

A decisão nesse momento é trabalhar por sintoma: latência e timeout em produção. Como staging está normal, eu não descartaria regressão de código; staging raramente reproduz volume, cardinalidade e dados reais.

## Primeiros 15 minutos

Nos primeiros 15 minutos, a investigação se dividiria em três frentes:

- aplicação: endpoints mais afetados, erros no Sentry, logs da versão recém-publicada;
- banco/cache/fila: slow queries, conexões, locks, Redis timeout, RabbitMQ backlog;
- deploy: diff da mudança, feature flags, migrações e alteração de consulta.

Se a latência continuar acima do limite aceitável e o erro estiver correlacionado ao deploy das 17h30, eu executaria rollback sem esperar causa raiz completa. O objetivo é sempre restaurar o serviço primeiro e investigar com calma depois.

Também verificaria se há mitigação sem rollback, como desabilitar feature flag, reduzir tráfego de endpoint específico, aumentar pool temporariamente ou aplicar índice emergencial. Mas eu só escolheria esse caminho se fosse mais rápido e menos arriscado do que rollback.

## Primeiros 60 minutos

Na primeira hora, espero que o incidente esteja mitigado ou com plano claro de mitigação em andamento.

As ações seriam:

1. Confirmar se rollback ou mitigação reduziu P95/P99 e erros.
2. Acompanhar recuperação por pelo menos duas janelas de medição.
3. Registrar timeline com horários de alerta, deploy, início de reclamações, decisão e mitigação.
4. Manter comunicação periódica com suporte, liderança e áreas impactadas.
5. Criar issue de correção definitiva.
6. Agendar postmortem blameless se houve impacto relevante em clientes ou consumo significativo de error budget.

## Comunicação

A comunicação deve ser curta, factual e com cadência definida.

Mensagem inicial:

> Identificamos degradação de latência na API em produção desde aproximadamente 18h. O time de engenharia está investigando a relação com o deploy das 17h30 e avaliando rollback. Próxima atualização em 15 minutos.

Mensagem durante mitigação:

> A hipótese principal é regressão introduzida no deploy das 17h30. Estamos executando rollback para restaurar a latência da API. Seguiremos monitorando P95, P99 e taxa de erro.

Mensagem final:

> A latência retornou ao patamar esperado após rollback. O incidente será analisado em postmortem blameless, com foco em causa raiz, lacunas de detecção e ações preventivas.

Uma status page pública traria ganho relevante de comunicação durante o incidente.

## Critério de rollback

Eu faria rollback se pelo menos uma das condições abaixo fosse verdadeira:

- latência P95/P99 acima do SLO por mais de duas janelas consecutivas;
- aumento de 5xx/timeouts após o deploy;
- reclamações de clientes confirmando impacto real;
- ausência de mitigação simples e segura em até 15 minutos;
- evidência de correlação temporal forte com o deploy das 17h30.

Rollback não deve ser visto como falha do time. Em incidente com cliente impactado, rollback é mecanismo normal de controle de dano.

## Evidências esperadas

As evidências a buscar seriam:

- gráfico de P95/P99 antes e depois das 17h30;
- taxa de erro e timeouts no Sentry;
- logs por endpoint/rota;
- slow queries no Postgres;
- consumo de conexões no banco;
- alterações do deploy;
- volume de tráfego no período;
- eventos de infraestrutura no mesmo horário.

## Encerramento

O incidente só deve ser encerrado quando a latência, taxa de erro e reclamações retornarem ao normal por janelas consecutivas. Depois disso, o foco passa para postmortem, correção definitiva e melhoria dos mecanismos de prevenção.

A causa raiz não precisa estar totalmente comprovada para mitigar. Mas precisa estar bem documentada antes de encerrar as ações pós-incidente.

A condução do incidente e do postmortem deve seguir uma abordagem blameless, partindo do princípio de que as pessoas tomaram as melhores decisões possíveis com as informações disponíveis naquele momento.