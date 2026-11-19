# Da Reunião ao Documento: Design Docs Gerados por IA

> Enunciado original: [devfullcycle/mba-ia-desafio-design-docs-com-ia](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia)

## Sobre o desafio

O ponto de partida foi a transcrição de uma reunião técnica de cerca de 55 minutos (`TRANSCRICAO.md`). Nela, tech lead, PM, e engenheiros decidem como construir um **Sistema de Webhooks de Notificação de Pedidos** para um Order Management System já em produção (Node 20 + TypeScript, Express, Prisma + MySQL). A tarefa era transformar essa conversa, junto com o código existente, num pacote de design docs (PRD, RFC, FDD, ADRs e Tracker) detalhado o bastante para o time começar a implementar, usando IA como ferramenta principal de produção.

O foco foi usar como redatora, aplicando conceitos vistos no módulo de Design Docs do MBA de IA, e não como engenheira: não se tratava de criar uma solução nova, mas de registrar a decisão tomada na reunião e consolidá-la em documentos formais. O maior desafio foi na escrita dos prompts, tentando encontrar uma maneira de instruir a IA a produzir cada documento de forma que um não repetisse o conteúdo do outro, e que cada decisão tivesse rastreabilidade até a transcrição ou até o código.

## Ferramentas de IA utilizadas

- **Claude Code (Claude Opus 5.5)**: ferramenta principal. Leu a transcrição e o código, gerou os templates e todos os documentos, manteve o tracker e fez os commits.
- **Subagente do Claude Code**: fez a triagem inicial da transcrição (o que entra e o que fica de fora) e gravou o resultado em `docs/transcription_analysis.md`.
- **Skill `smart-commit`** (`.claude/skills/smart-commit/SKILL.md`): padroniza as mensagens de commit no formato `<tipo>: <resumo> — <motivo>`, com um item por arquivo alterado.

## Workflow adotado

A produção foi de baixo para cima: primeiro as decisões pontuais e depois os documentos que as consolidam. Assim, cada documento se apoia apenas no que já foi validado nos anteriores.

1. **Setup**: criação do `CLAUDE.md` com o contexto do repositório e as regras de elaboração de documentos.
2. **Templates** (`docs/templates/`): templates de PRD, RFC e ADR (formato MADR) gerados com base no template de FDD que já existia, material de referência extraído da internet, e conceitos vistos no módulo do curso. Cada documento ficou restrito à sua "altura" (produto, arquitetura, decisão pontual, implementação), para que um não repetisse o conteúdo do outro.
3. **Triagem da transcrição** (`docs/transcription_analysis.md`): separação entre o que foi considerado e o que foi descartado, com o timestamp e o falante de cada item. Essa etapa permitiu que o agente principal se concentrasse apenas no que seria usado na produção dos documentos, sem precisar reler a transcrição inteira.
4. **ADRs** (`docs/adrs/`): 8 decisões que passaram pelo filtro dos "3 Es" (estrutural, evidente, estável). As candidatas excedentes foram empurradas para o FDD.
5. **RFC**: consolidação das ADRs numa proposta técnica com alternativas descartadas e questões em aberto, sem chegar ao nível de detalhe do FDD.
6. **FDD**: especificação de implementação a partir das ADRs e do RFC, complementada com o código (contratos HTTP, matriz de erros `WEBHOOK_*`, resiliência, observabilidade e integração com o código existente).
7. **PRD**: visão de produto consolidada a partir de ADRs, RFC, FDD e transcrição.
8. **Tracker** (`docs/TRACKER.md`): atualizado em todas as etapas. É o único elo entre os documentos e as fontes (`TRANSCRICAO` ou `CODIGO`).

**Como organizei a interação com a IA:** cada documento teve um prompt próprio, salvo em `prompts/`, sempre com a mesma estrutura: papel, objetivo, requisitos, workflow passo a passo, negativas e material de apoio. Todo workflow termina com um passo de autorrevisão contra o template e as fontes. Depois de cada etapa eu revisava o resultado e fazia um commit antes de seguir para a próxima.

A revisão foi realizada com o documento aberto ao lado do tracker. Toda afirmação feita nos documentos foi buscada no tracker, que por sua vez aponta para a transcrição ou para o código. Se não havia rastreabilidade, a afirmação era removida ou corrigida. Houve apenas um caso em que a IA inventou uma informação (sobre contagem de tentativas de retry) e foi necessário reverter o trecho do FDD e do RFC para o que estava na transcrição.

## Prompts customizados

Todos os prompts estão em [`prompts/`](prompts/).

## Iterações e ajustes

Estes foram os principais momentos em que precisei corrigir a IA:

1. **Formatação dos metadados das ADRs.** Os campos `Status`, `Data`, `Supersedes` etc. vieram em linhas consecutivas sem quebra, e no Markdown renderizado viravam um único parágrafo. Corrigi as 8 ADRs com quebra de linha explícita (commit `e36ba37`).
2. **Citações espalhadas no RFC.** A primeira versão do RFC tinha citações `[hh:mm] Nome` no meio do texto, misturando conteúdo com rastreabilidade. Movi todas para o tracker, que passou a ser o único elo entre documentos e transcrição, e reescrevi as frases afetadas (commit `83c7912`).
3. **Requisito inventado na política de retry.** Ao gerar o FDD, a IA "resolveu" uma questão em aberto: concluiu que as 5 tentativas incluíam o envio inicial. A partir disso, criou uma invariante de tentativas máximas no FDD e uma nova questão em aberto no RFC. Nada disso estava na transcrição. Reverti tudo para o texto decidido na reunião ("5 tentativas com backoff 1m/5m/30m/2h/12h") e mantive a contagem como ponto em aberto (commit `61c01f1`).
4. **Regras endurecidas nos prompts.** Depois desses desvios, as regras "não use emojis e/ou travessão" e "não invente informações para placeholders" passaram a constar em todos os prompts. A regra de rastrear toda afirmação até `TRANSCRICAO.md` ou até um arquivo real também foi parar no `CLAUDE.md`.

**Número de iterações principais:** No total, foram necessárias cerca de 10 iterações para chegar a um pacote de documentos coerente e consistente, com rastreabilidade completa, além do que foi corrigido manualmente durante a revisão.

**Lições aprendidas:** mesmo com prompts bem estruturados contendo alguns guardrails e um modelo poderoso e consolidado, a geração de documentos complexos, de uma quantidade grande de informações, ainda requer supervisão humana. O processo de revisão, claro, pode ser facilitado com o uso de agentes com objetivos mais específicos de revisão, dentro de um pipeline na produção de documentos, mas tudo isso precisa ser projetado de forma que haja rastreabilidade das decisões.

## Como navegar a entrega

```
docs/
├── PRD.md                     # Por que e o quê (produto/negócio)
├── RFC.md                     # Proposta técnica, alternativas e questões em aberto
├── adrs/                      # 8 decisões arquiteturais (ADR-001 a ADR-008)
├── FDD.md                     # Como construir em detalhe (fluxos, contratos, erros, integração)
├── TRACKER.md                 # Rastreabilidade de cada item até a transcrição ou o código
├── transcription_analysis.md  # Triagem da transcrição (considerado x descartado)
└── templates/                 # Templates usados para gerar cada documento
prompts/                       # Prompts usados em cada etapa
CLAUDE.md                      # Contexto e regras passados à IA
TRANSCRICAO.md                 # Fonte original (não editada)
```

**Ordem sugerida de leitura** (do mais abstrato ao mais concreto, o inverso da ordem de produção):

1. [`docs/PRD.md`](docs/PRD.md): problema, público, escopo, requisitos e métricas.
2. [`docs/RFC.md`](docs/RFC.md): visão geral da solução e o que ainda está em aberto.
3. [`docs/adrs/`](docs/adrs/): o racional de cada decisão.
   - [ADR-001](docs/adrs/ADR-001-outbox-no-mysql.md): outbox no MySQL
   - [ADR-002](docs/adrs/ADR-002-worker-em-polling-de-2-segundos.md): worker em polling de 2 segundos
   - [ADR-003](docs/adrs/ADR-003-worker-em-processo-separado.md): worker em processo separado
   - [ADR-004](docs/adrs/ADR-004-retry-com-backoff-exponencial.md): retry com backoff exponencial
   - [ADR-005](docs/adrs/ADR-005-dead-letter-em-tabela-separada.md): dead letter em tabela separada
   - [ADR-006](docs/adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md): assinatura HMAC-SHA256 por endpoint
   - [ADR-007](docs/adrs/ADR-007-entrega-at-least-once-com-event-id.md): entrega at-least-once com event id
   - [ADR-008](docs/adrs/ADR-008-payload-snapshot-na-insercao.md): payload snapshot na inserção
4. [`docs/FDD.md`](docs/FDD.md): especificação de implementação.
5. [`docs/TRACKER.md`](docs/TRACKER.md): para conferir a origem de qualquer item.

Para ver o processo: [`prompts/`](prompts/), [`docs/transcription_analysis.md`](docs/transcription_analysis.md) e o histórico de commits (`git log`).
