### Objetivo

Você é um especialista em design docs para a engenharia de software, e nessa tarefa deve criar templates de alguns tipos de documentos, seguindo as especificações de cada um deles.

Os templates devem ser criados dentro de /docs/templates, para cada um dos documentos abaixo:

- PRD (Product Requirement Document) da feature
- RFC (Request for Comments) com a proposta técnica da solução, submetida à equipe para revisão
- ADR (Architecture Decision Record) 

Cada documento tem a responsabilidade de responder informar e responder perguntas em diferentes níveis de negócio, conforme tabela abaixo:

| Documento | Papel | Altura | Pergunta que responde |
| --- | --- | --- | --- |
| **PRD** | Problema, público, escopo e métricas de sucesso | Produto / negócio | *Por que e o quê?* |
| **RFC** | Proposta técnica da solução para revisão: abordagem geral, alternativas e questões em aberto | Arquitetura | *Como pretendemos resolver, e o que ainda está em aberto?* |
| **ADRs** | Cada decisão arquitetural isolada, com contexto e consequências | Decisão pontual | *Por que decidimos exatamente assim?* |
| **FDD** (já existente) | Especificação de implementação: fluxos, contratos, erros, integração com o código | Implementação | *Como construir, em detalhe?* |

O template de cada tipo de documento deve conter os tópicos que atendam a TODOS os requisitos descritos abaixo, limitando-se a apenas eles, e não repetindo informações que são de responsabilidade de outros documentos.

### 1\. PRD da feature

- Resumo e contexto da feature
- Problema e motivação
- Público-alvo e cenários de uso
- Objetivos e métricas de sucesso
- Escopo (incluso e fora de escopo)
- Requisitos funcionais
- Requisitos não funcionais
- Decisões e trade-offs principais
- Dependências
- Riscos e mitigação
- Critérios de aceitação
- Estratégia de testes e validação

### 2\. RFC da feature

- Metadados (autor, status, data, revisores); use os participantes da reunião como revisores
- Resumo executivo (TL;DR) da proposta
- Contexto e problema
- Proposta técnica (visão geral da solução, sem descer ao detalhe de implementação do FDD)
- Alternativas consideradas (pelo menos 2 alternativas reais discutidas e descartadas na reunião, cada uma com o trade-off que levou ao descarte)
- Questões em aberto (pelo menos 2 pontos levantados na reunião e não decididos ou adiados)
- Impacto e riscos
- Decisões relacionadas (links para os ADRs correspondentes)

### 3\. FDD da feature (apenas para referência - template já existente)

- Contexto e motivação técnica
- Objetivos técnicos
- Escopo e exclusões
- Fluxos detalhados (criação do evento na outbox, processamento pelo worker, retry, DLQ)
- Contratos públicos (endpoints HTTP com payloads de exemplo, headers, status codes, semântica)
- Matriz de erros previstos com códigos no padrão `WEBHOOK_*`
- Estratégias de resiliência (timeouts, retries, backoff, fallback)
- Observabilidade (métricas, logs, tracing)
- Dependências e compatibilidade
- Critérios de aceite técnicos
- Riscos e mitigação

### 4\. ADRs

Devem seguir o formato MADR, contendo:

- Metadados (Status: Supersedes or Superseded by, data, related to ADR, Amends)
- Contexto
- Decisão
- Alternativas consideradas (pelo menos uma)
- Consequencias positivas e negativas (com trafe-off explícito)
- Referências

MADR docs -> https://adr.github.io/madr/decisions/adr-template.html


### Informações adicionais

- Dentro da pasta `/docs/templates` já existe um template de uma FDD. Use-a como referência de formatação para os demais templates.
- Estilo de escrita: conciso e informativo. Jamais use emojis nos templates.
