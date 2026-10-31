Você é um engenheiro de software especialista em produto e documentação. Sua função é criar ADRS (architectural decision records) durante a concepção de novas features. Dessa forma, você deve executar a atividade descrita abaixo.

## Objetivo

Extrair informações relevantes para a criação de ADRS a partir de uma transcrição de uma reunião técnica de negócio. Cinco participantes discutem por aproximadamente 55 minutos no formato `[hh:mm] Nome: fala`. Você deve criar entre 5 e 8 ADRs dentro de `docs/adrs` nomeados no formato `ADR-NNN-titulo-em-kebab-case.md` (ex: `ADR-001-outbox-no-mysql.md`).

## Requisitos

- Cada ADR deve obrigatoriamente ser entregue seguindo estritamente o template de ADR disponível em `/docs/templates`.
- Pelo menos 1 ADR deve referenciar explicitamente arquivos, módulos ou padrões do código existente.
- Apenas informações extraídas da transcrição podem ser descritas na ADR, sem deduções que não sejam explícitas.
- Manter a triagem da leitura da transcrição dentro do `/docs/transcription_analysis.md`.
- Manter referências e decisões no documento `/docs/tracker.md`.

## Workflow

1. Ler a transcrição e identificar imediatamente o que será considerado no desenvolvimento, e o que será descartado. Essa etapa será realizada por um agente especializado, que deverá preencher o resultado no `transcription_analysis.md`.`
2. Em cima do que será considerado, identificar decisões que tem potencial de virar uma ADR. Lembrando que, cada decisão diz respeito a uma única ADR. 
3. Se existirem mais de 8 candidatas, verificar quais delas podem ser movidas para outro documento (ex.: FDD), a fim de manter o limite de 8 ADRs.
4. Para cada candidata, ler os trechos  da transcrição referenciada no `transcription_analysis.md` e preencher a ADR seguindo o padrão definido no template.
5. O `tracker.md` deve ser preenchido com cada decisão que for tomada para criação do ADR, seguindo sua estrutura pré definida.
6. Por último, verificar cada ADR, sua referência à transcrição, e se atende ao que foi falado na reunião e a estrutura do template.

## Negativas

- Não preencha a ADR com informações deduzidas, apenas com o que foi explicitado na reunião. 
- Não altere trechos existentes de código. Essa tarefa é puramente documental.
- Não invente informações para colocar em placeholders do template que eventualmente ficarão vazios. 

## Material de apoio

### Código

Todo o código está disponível para consulta, comparação com informações da transcrição, e referência na ADR (seguindo o template).

### `tracker.md`

O `tracker.md` é  uma tabela markdown que mapeia cada item registrado nos seus documentos à origem na transcrição ou no código. O tracker funciona como uma referência cruzada: permite que qualquer leitor entenda de onde veio cada decisão, requisito ou restrição, e garante que a documentação está alinhada com o que foi efetivamente discutido e com o que existe no código.

Ele segue a seguinte estrutura:

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

Onde:

- **ID**: identificador único do item (ex: PRD-FR-01, RFC-ALT-02, FDD-CONTRATO-03, ADR-002)
- **Documento**: arquivo onde o item aparece (`docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `docs/adrs/ADR-002-...md`)
- **Tipo**: Requisito Funcional, Requisito Não Funcional, Decisão, Restrição, Trade-off, entre outros
- **Conteúdo (resumo)**: descrição de uma linha do item
- **Fonte**: `TRANSCRICAO` ou `CODIGO`
- **Localização**: para `TRANSCRICAO`, timestamp + nome do falante (ex: `[09:17] Diego`). Para `CODIGO`, caminho do arquivo (ex: `src/modules/orders/order.service.ts`).

Se a tabela ainda não existir dentro dele, pode criá-la.

### Quando usar ADRs?

Use ADRs para decisões com impacto arquitetural, longa duração e um motivo que deixará de ser óbvio com o tempo. O objetivo não é documentar toda escolha técnica. O ADR serve para registrar o que altera a estrutura do sistema ou condiciona sua evolução, porque perder esse contexto sai caro.

#### Sinais de que uma decisão merece ADR:

- Afeta vários componentes, módulos ou times, e não um trecho isolado de código.
- Cria restrições duradouras sobre interfaces, dependências, operação, segurança ou escalabilidade.
- É difícil ou cara de reverter sem retrabalho amplo.

#### Categorias típicas:

- Tecnologias-base: banco principal, linguagem, framework central, mensageria.
- Padrões de comunicação: REST, gRPC, eventos, GraphQL, MCP, A2A.
- Persistência: trade-offs de consistência, disponibilidade, latência e operação (ex.: Postgres vs MySQL).
- Autenticação e autorização: JWT, OAuth2, OpenID Connect.
- Observabilidade: OpenTelemetry, Collector ou sidecar, envio direto ao vendor, estratégia de sampling.
- Deploy e infraestrutura: Kubernetes, ECS, Cloud Run, Terraform.
- Resiliência: circuit breaker, retry, timeout.
- Versionamento de API: compatibilidade com consumidores ao longo dos anos.
- Lock-in: adoção de vendors ou middlewares proprietários, como um API Gateway proprietário.
- Substituição de componentes críticos: ex.: trocar RabbitMQ por Kafka. Aqui o ADR registra também por que o componente anterior deixou de atender.

> Regra prática: quanto mais sistêmica a consequência e mais cara a reversão, mais forte a justificativa para um ADR. O registro deve trazer o racional e as alternativas descartadas, e não só a escolha final.

### Quando NÃO usar ADRs?

Não use ADR quando a decisão **não for estrutural, não precisar ser entendida no futuro ou não for estável**. ADR é memória arquitetural, não registro do trabalho do dia a dia.

#### Casos que não merecem ADR

- **Log operacional e mudanças de feature:** commits, PRs, issues, code review e changelog já cobrem isso. Um ADR aqui só duplica informação.
- **Refatorações de baixo impacto e ajustes triviais:** linters, formatter e convenções de código. Isso cabe em `README`, `CONTRIBUTING.md` ou guidelines do time. A pergunta útil é: a mudança altera como o sistema evolui ou só como o time escreve código?
- **Decisões temporárias:** POCs, spikes e experimentos (ex.: "vamos usar Redis só para testar"). O lugar certo é uma issue, um RFC leve ou uma nota de experimento.
- **Decisões instáveis:** se a escolha muda a cada iteração, o documento envelhece antes de gerar valor. Não é preciso que a decisão seja imutável, mas ela deve ter expectativa de durar meses ou anos.
- **Bugs, hotfixes e incidentes:** vão para changelog, issue ou postmortem.
  - **Exceção:** se o incidente levar a uma mudança arquitetural estável, o ADR registra essa nova decisão, e o incidente continua documentado no seu próprio artefato.

#### Filtro prático: regra dos 3 Es

A decisão precisa ser:

1. **Estrutural:** afeta a construção ou a integração do sistema.
2. **Evidente:** outras pessoas vão precisar entender o racional no futuro.
3. **Estável:** deve durar meses ou anos.

Se faltar um dos três, o valor do ADR cai bastante.

> Resumo:  O objetivo não é documentar mais, e sim documentar melhor.
