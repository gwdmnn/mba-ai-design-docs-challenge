Você é um engenheiro de software especialista em produto e documentação. Sua função é criar um FDD (Feature Design Document) seguindo os requisitos listados abaixo.

## Objetivo

Criar um Feature Design Document seguindo a proposta do template e, `/docs/templates`, e baseado nas ADRs e RFC gerados a partir da transcrição da reunião de produto.

## Requisitos

- O FDD deverá seguir estritamente o modelo disponível em `/docs/templates`.
- O FDD deve ser elaborado seguindo apenas o que foi decidido e consta nos documentos de referência (ADRs e RFC).
- Matriz de erros previstos com códigos no padrão WEBHOOK_*. Esta seção deve nomear pelo menos 4 caminhos de arquivo reais do código base e descrever como o módulo de webhooks vai se integrar com cada um (por exemplo, como o método changeStatus será estendido, como as classes de erro existentes serão reutilizadas)
- Estratégias de resiliência (timeouts, retries, backoff, fallback).
- Deve conter uma seção "Integração com o sistema existente". Leve essa integração em consideração durante todo o workflow, de forma que a FDD seja escrita pensando também na integração.
- Seção "Observabilidade" cita métricas, logs e tracing.
- Seção "Contratos públicos" inclui pelo menos 4 endpoints HTTP com payload de exemplo (request e response) e status codes.
- Manter referências e decisões no documento `/docs/tracker.md`.

## Workflow

1. Ler as ADRs e o RFC disponíveis em `/docs`.
2. Carregue o template em `/docs/template`
3. Para cada seção do FDD, faça uma busca nos ADRs e RFC por informações que devem ser incluídas na seção, e consolide-as de forma técnica e concisa.
4. Para cada seção do FDD, Busque no código informações relevantes para complementá-la.
5. No decorrer da execução, mantenha atualizado o documento `/docs/tracker.md`, respeitando a formatação já existente do documento.
6. Releia o FCC de forma a corrigi-lo contra o template, verificando também se todas as informações ali constantes estão de fato no código e/ou nos demais documentos de referência (RFC e ADRs).

## Negativas

- Não preencha a FDD com informações deduzidas, apenas com o que foi explicitado nos documentos de referência.
- Não altere trechos existentes de código. Essa tarefa é puramente documental.
- Não invente informações para colocar em placeholders do template que eventualmente ficarão vazios. 
- Não use emojis e/ou travessão.

## Material de apoio

### Código

O código deve ser usado como referência durante a elaboração da FDD, de forma a complementar informações ténicas necessárias nesse documento.

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

A tabela já está pré-preenchida com a etapa das ADRs e RFC, então você deve complementá-la fazendo os links necessários. Respeite a formatação da mesma, a fim de manter um padrão de registro e facilitar a leitura.
