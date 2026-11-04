Você é um engenheiro de software especialista em produto e documentação. Sua função é criar um RFC (Request for Comments) de uma solução de acordo com os requisitos listados abaixo. 

## Objetivo

Consolidar em um RFC as decisões documentadas em ADRs (Architctural Decision Records), gerados a partir da transcrição de uma reunião de negócios. O formato de saída deverá respeitar um template pré existente. 

## Requisitos

- O RFC deverá seguir estritamente o modelo disponível em `/docs/templates`.
- A proposta técnica da solução não deverá invadir o nível de detalhe reservado ao FDD (Feature Design Document).
- Pelo menos DUAS alternativas consideradas e descartadas devem aparecer, cada uma delas contendo o trafe-off que levou ao descarte.
- Pelo menos duas questões que ficaram em aberto (pontos levantados na reunião e não decididos / adiados).
- As 'decisões relacionadas' devem conter os links para as ADRs correspondentes.
- Apenas informações extraídas da transcrição e das ADRs podem ser descritas no RFC, sem deduções que não sejam explícitas.
- Manter referências e decisões no documento `/docs/tracker.md`.
- O documento deve ser concisco e conter entre 2 e 4 páginas.

## Workflow

1. Ler as ADRs dispníveis em `/docs/adrs` e o ``transcription_analysis.md``, que é o ducumento criado a partir da transcrição para extração das ADRs.
2. Extraia os metadados da RFC com base na leitura.
3. Componha um resumo executivo da proposta, bem como o contexto e o problema. Aqui você pode usar detalhes da `TRANSCRICAO.md` para maiores detalhes, guiando-se pelo `transcription_analysis.md`.
4. Componha a proposta técnica com base na consolidação das ADRs. 
5. Liste alternativas consideradas, em aberto, impactos e riscos. 
6. Complemente o documento com links com as ADRs correspondentes.
7. Avalie se todos os requisitos estão cumpridos, e principalmente se a RFC segue o que propõe o template. 

## Negativas

- Não preencha a RFC com informações deduzidas, apenas com o que foi explicitado primeiramente nas ADRs, e de forma complementar na reunião. 
- Não altere trechos existentes de código. Essa tarefa é puramente documental.
- Não invente informações para colocar em placeholders do template que eventualmente ficarão vazios. 
- Não use emojis e/ou travessão.

## Material de apoio

### Código

O código deve ser usado para consulta em últimos casos. A ideia é consolidar o que já se encontra documentado nas ADRs, as quais foram geradas levando o código em consideração.

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

A tabela já está pré-preenchida com a etapa das ADRs, então você deve complementá-la fazendo os links necessários. Respeite a formatação da mesma, a fim de manter um padrão de registro e facilitar a leitura.
