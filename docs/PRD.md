### PRD: Webhooks de notificação de mudança de status de pedidos

Versão: 0.1
Data: 2026-11-13
Responsável: Marcos (Product Manager)

Documentos de referência: [RFC](RFC.md), [FDD](FDD.md) e [ADR-001 a ADR-008](adrs/). Rastreabilidade de cada item em [TRACKER.md](TRACKER.md) (IDs `PRD-*`).

---

### 1. Resumo e contexto
A feature notifica clientes B2B, por webhook de saída, sempre que o status de um pedido deles muda na plataforma. Hoje a plataforma não emite eventos: Atlas Comercial, MaxDistribuição e Nova Cargo consultam `GET /orders` periodicamente para detectar mudanças, o que deixa a integração lenta e cara para eles. Os três fizeram um pedido formal para receber essas notificações em "tempo real", que para eles significa menos de 10 segundos.

---

### 2. Problema e motivação
- **Problema:** os clientes B2B só descobrem mudanças de status consultando `GET /orders` de tempos em tempos, o que torna a integração lenta e cara.
- **Impacto de não resolver:** a Atlas Comercial sinalizou que pode migrar para um concorrente se a feature não for entregue no prazo.
- **Por que agora:** pedido formal dos três clientes recebido na semana anterior à reunião, com prazo da Atlas citado como "fim do trimestre" e depois como "fim de novembro". A divergência entre os dois prazos está em aberto (RFC, seção 5).

---

### 3. Público-alvo e cenários de uso

**Público-alvo**
- Clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo): precisam saber em menos de 10 segundos quando cada pedido deles muda de status, sem consultar `GET /orders`.
- Usuários do sistema que representam o cliente, autenticados com JWT da plataforma: precisam cadastrar e gerenciar os webhooks do customer pela API.
- Usuários com role ADMIN: precisam reprocessar eventos que falharam de forma permanente.

**Cenários de uso**
- Um usuário que representa o cliente cadastra um webhook com url `https` e a lista `SHIPPED` e `DELIVERED`; recebe a secret gerada pela plataforma e passa a receber apenas eventos desses dois status.
- Um pedido do cliente muda de status; o endpoint do cliente recebe em menos de 10 segundos um `POST` assinado com HMAC-SHA256, verifica a assinatura e deduplica pelo `X-Event-Id`.
- O endpoint do cliente fica fora do ar por duas horas em manutenção planejada; as retentativas com backoff cobrem a janela e o evento é entregue quando o endpoint volta.
- O endpoint do cliente fica indisponível por mais de quase 15 horas; o evento vai para a dead letter e um ADMIN o reprocessa manualmente pelo endpoint de replay.
- Após um vazamento de secret, o cliente pede uma nova pela API e tem 24 horas, com a antiga ainda válida, para migrar os sistemas.
- O cliente consulta o histórico de entregas de um webhook para ver sucesso ou falha, payload, response e tempo de resposta dos últimos envios.

---

### 4. Objetivos e métricas de sucesso

| Objetivo | Métrica | Linha de base | Meta | Prazo |
| --- | --- | --- | --- | --- |
| Notificar o cliente em "tempo real" | Tempo entre a mudança de status e a entrega no endpoint do cliente | Sem notificação; cliente depende de consulta periódica a `GET /orders` | Menos de 10 segundos | Três sprints, com revisão de segurança incluída; Atlas quer até o fim de novembro |
| Não perder eventos de mudança de status | Mudanças de status commitadas sem evento na outbox, quando há webhook interessado | Não se aplica; a plataforma não emite eventos | 0 | Três sprints |
| Dar fim definido a todo evento | Tentativas de entrega por evento antes de entrega ou dead letter | Não se aplica; a plataforma não emite eventos | 5 tentativas; inclusão do envio inicial em aberto (RFC, seção 5) | Três sprints |

---

### 5. Escopo

**Incluído**
- Registro do evento em `webhook_outbox` na mesma transação de `changeStatus`, com rollback da mudança de status se a inserção falhar.
- Worker em processo separado (`npm run worker`) com polling de 2 segundos.
- CRUD de configuração de webhook (url, lista de status, estado ativo), com secret gerada pela plataforma.
- Filtro por lista de status aplicado na inserção da outbox.
- Entrega via `POST` com payload JSON enxuto, assinatura HMAC-SHA256 e headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type`.
- Retry com backoff exponencial, dead letter em `webhook_dead_letter` e replay manual por ADMIN.
- Rotação de secret com grace period de 24 horas.
- Histórico de entregas por webhook.
- Validação de url `https` e limite de payload de 64KB.
- Documentação de integração, deduplicação e verificação de assinatura no portal de desenvolvedor.

**Fora de escopo**
- Webhooks de entrada (clientes enviando para a plataforma) (descartado): os clientes querem apenas receber | Referência [transcrição: `[09:02] Marcos`; `[09:03] Sofia`]
- Notificação por email ao cliente quando o webhook falha, inclusive como fallback (adiado): próxima fase, depois de medir o impacto | Referência [transcrição: `[09:37] Larissa`; `[09:48] Larissa`]
- Dashboard visual para o cliente acompanhar os webhooks (adiado): "agora não, só endpoints"; painel é projeto separado do time de frontend | Referência [transcrição: `[09:40] Larissa`]
- Rate limiting de envio para o cliente (adiado): observar e implementar se virar problema | Referência [transcrição: `[09:39] Diego`; `[09:39] Larissa`]
- Arquivamento das linhas entregues da outbox (adiado): fora do escopo desta feature | Referência [transcrição: `[09:08] Diego`]
- Garantia exactly-once (descartado): exigiria coordenação dos dois lados; at-least-once com `X-Event-Id` foi adotado | Referência [transcrição: `[09:25] Diego`]
- Ordenação global e escala para múltiplos workers (adiado): ordem garantida só por `order_id` e com um único worker; escalar é "problema do futuro" | Referência [transcrição: `[09:13] Diego`; `[09:13] Larissa`; `[09:14] Marcos`]

---

### 6. Requisitos funcionais

A reunião não estabeleceu priorização entre os requisitos; todos compõem a entrega estimada em três sprints e confirmada no resumo final.

| ID | Requisito | Prioridade |
| --- | --- | --- |
| RF-01 | O usuário que representa o cliente cadastra um webhook informando url e lista de status; a plataforma gera a secret e a devolve na resposta de criação | não definida |
| RF-02 | O usuário lista os webhooks de um customer | não definida |
| RF-03 | O usuário edita a configuração de um webhook (url, lista de status, estado ativo) | não definida |
| RF-04 | O usuário remove um webhook | não definida |
| RF-05 | A plataforma recusa com erro de validação o cadastro ou a edição de url que não seja `https` | não definida |
| RF-06 | O cliente recebe eventos apenas dos status presentes na lista do webhook; se nenhum webhook do customer quer o status, nenhum evento é gerado | não definida |
| RF-07 | A cada mudança de status de interesse, o endpoint do cliente recebe um `POST` com `event_id`, `event_type` `order.status_changed`, timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos da order como `total_cents`, sem items | não definida |
| RF-08 | Cada entrega leva os headers `X-Event-Id`, `X-Signature` (HMAC-SHA256 do corpo), `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json` | não definida |
| RF-09 | Entregas que falham são retentadas com backoff exponencial (1m, 5m, 30m, 2h, 12h) e, ao esgotar as tentativas, o evento vai para a dead letter com payload, motivo da falha e timestamp | não definida |
| RF-10 | Um ADMIN reprocessa manualmente um evento da dead letter, que volta para a outbox como pendente | não definida |
| RF-11 | O cliente pede uma nova secret pela API; a antiga continua válida por 24 horas em paralelo e depois deixa de valer | não definida |
| RF-12 | O cliente consulta o histórico de entregas de um webhook (referência de "os últimos 100"), com sucesso ou falha, payload, response e tempo de resposta | não definida |

---

### 7. Requisitos não funcionais

| ID | Categoria | Requisito |
| --- | --- | --- |
| RNF-01 | Desempenho | Entrega no endpoint do cliente em menos de 10 segundos após a mudança de status |
| RNF-02 | Desempenho | Worker lê a outbox por polling a cada 2 segundos, em batch pequeno, priorizando os pendentes mais antigos |
| RNF-03 | Disponibilidade | Timeout de 10 segundos por chamada HTTP ao cliente; sem resposta nesse prazo, a tentativa é falha e vai para retry |
| RNF-04 | Disponibilidade | 5 tentativas por evento com backoff exponencial; se o envio inicial conta entre elas está em aberto (RFC, seção 5) |
| RNF-05 | Disponibilidade | Worker roda em processo separado da API; reinício da API não interrompe as entregas |
| RNF-06 | Consistência | Nenhuma mudança de status é commitada sem o evento correspondente na outbox quando há webhook interessado; falha na inserção desfaz a mudança de status |
| RNF-07 | Consistência | Entrega at-least-once, com `X-Event-Id` UUID único por evento para deduplicação no cliente |
| RNF-08 | Consistência | Payload renderizado na inserção da outbox, refletindo o estado do pedido no momento da mudança de status |
| RNF-09 | Consistência | Ordem de entrega garantida apenas por `order_id` e enquanto houver um único worker |
| RNF-10 | Segurança | Secret única por endpoint de webhook; nunca há secret global da plataforma |
| RNF-11 | Segurança | TLS obrigatório: url de webhook sempre `https` |
| RNF-12 | Segurança | CRUD de configuração acessível a qualquer role autenticada; replay da dead letter exige role ADMIN |
| RNF-13 | Auditoria | Todo replay da dead letter é registrado em log com o usuário que o executou |
| RNF-14 | Capacidade | Payload limitado a 64KB; acima disso o evento não é enviado e é gerado erro |
| RNF-15 | Manutenibilidade | Módulo segue os padrões existentes (`AppError`, Pino, middleware de erro, schemas Zod) com códigos de erro prefixados por `WEBHOOK_` |

---

### 8. Decisões e trade-offs principais

| Decisão | Ganho para o produto | Custo aceito | ADR |
| --- | --- | --- | --- |
| Outbox no MySQL na mesma transação da mudança de status | Nunca há status alterado sem evento, sem nova infraestrutura | Falha na inserção da outbox impede a mudança de status | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| Worker em polling a cada 2 segundos | Atende ao requisito de menos de 10 segundos sem improvisar notificação a partir do banco | Latência de até 2 segundos inerente à solução | [ADR-002](adrs/ADR-002-worker-em-polling-de-2-segundos.md) |
| Worker em processo separado da API | Entregas não param quando a API reinicia | Processo adicional para operar | [ADR-003](adrs/ADR-003-worker-em-processo-separado.md) |
| 5 tentativas com backoff 1m/5m/30m/2h/12h | Cobre indisponibilidades como manutenção planejada de duas horas e evita evento pendurado para sempre | Cliente fora por mais de quase 15 horas tem o evento em falha permanente | [ADR-004](adrs/ADR-004-retry-com-backoff-exponencial.md) |
| Dead letter em tabela separada com replay manual | Evidência preservada para debug e reprocessamento | Tabela adicional e reprocessamento manual por ADMIN | [ADR-005](adrs/ADR-005-dead-letter-em-tabela-separada.md) |
| HMAC-SHA256 com secret por endpoint e rotação de 24 horas | Cliente valida origem e integridade; vazamento fica restrito a um endpoint | Duas secrets válidas em paralelo por 24 horas | [ADR-006](adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md) |
| At-least-once com `X-Event-Id` | Evita a complexidade de exactly-once | Cliente precisa deduplicar eventos | [ADR-007](adrs/ADR-007-entrega-at-least-once-com-event-id.md) |
| Payload renderizado na inserção da outbox | Evento reflete o estado de quando o status mudou | Nenhum custo explicitado | [ADR-008](adrs/ADR-008-payload-snapshot-na-insercao.md) |

---

### 9. Dependências

| Dependência | Tipo | Responsável | Impacto se indisponível |
| --- | --- | --- | --- |
| Revisão de segurança do HMAC e da geração de secret, de ao menos dois dias úteis, antes do deploy | interna | Sofia (Engenharia de Segurança) | Deploy não acontece sem a revisão |
| Revisão do doc de design por Bruno e Diego antes do início da implementação | interna | Larissa (Tech Lead) | Implementação não começa |
| Documentação de integração, deduplicação por `X-Event-Id` e verificação de HMAC no portal de desenvolvedor | interna | Marcos (Product Manager) | Clientes sem orientação para tratar entregas duplicadas, que podem ocorrer |
| Confirmação do prazo com a Atlas Comercial | externa | Marcos (Product Manager) | Prazo segue divergente entre "fim do trimestre" e "fim de novembro" |
| Endpoint `https` do cliente disponível e respondendo em até 10 segundos | externa | Clientes B2B | Entregas falham, entram em retry e, esgotadas as tentativas, vão para a dead letter |
| Deduplicação de eventos por `X-Event-Id` no sistema do cliente | externa | Clientes B2B | Cliente processa em duplicidade eventos reentregues |

---

### 10. Riscos e mitigação

### Cliente indisponível além da janela de retentativas

- **Probabilidade:** baixa: a maior indisponibilidade de cliente citada foi de duas horas, coberta pela janela de quase 15 horas
- **Impacto:** baixo: aceito pelo produto, pois um cliente fora por 15 horas "já tá com problema sério dele"; o evento cai em falha permanente
- **Mitigação:**
    - Evento preservado na dead letter com payload, motivo da falha e timestamp
    - Replay manual por ADMIN

### Perda de ordenação ao escalar para múltiplos workers

- **Probabilidade:** baixa: a escala para múltiplos workers não está prevista e foi tratada como "problema do futuro"
- **Impacto:** baixo: os clientes nunca pediram ordenação global, apenas saber se cada pedido mudou
- **Mitigação:**
    - Manter um único worker, com ordem por `created_at` da outbox
    - Particionar por `order_id` ou usar lock pessimista se a escala vier a ser necessária

### Payload acima de 64KB

- **Probabilidade:** baixa: nenhum evento atual chega perto do limite
- **Impacto:** não avaliado: o evento não é enviado e é gerado erro
- **Mitigação:**
    - Payload enxuto, sem items; o cliente consulta `GET /orders/:id` se precisar de detalhes

### Não entregar até o prazo da Atlas Comercial

- **Probabilidade:** não avaliada
- **Impacto:** alto: a Atlas pode migrar para o concorrente
- **Mitigação:**
    - Estimativa de três sprints com a revisão de segurança incluída
    - Confirmação do prazo com a Atlas por Marcos

### Vazamento de secret pelo cliente

- **Probabilidade:** não avaliada: já ocorreu uma vez, com secret exposta em log de aplicação do cliente
- **Impacto:** não avaliado: terceiros podem forjar ou adulterar entregas para o endpoint afetado
- **Mitigação:**
    - Secret única por endpoint, restringindo o vazamento a um cadastro
    - Rotação de secret pela API com grace period de 24 horas
    - Revisão de segurança do HMAC e da geração de secret

### Cliente receber o mesmo evento mais de uma vez

- **Probabilidade:** não avaliada: possível pela garantia at-least-once
- **Impacto:** não avaliado: processamento duplicado no cliente
- **Mitigação:**
    - `X-Event-Id` único por evento para deduplicação
    - Destaque da deduplicação no portal de desenvolvedor

---

### 11. Critérios de aceitação

- [ ] Cadastro com url `https` e lista de status retorna o webhook criado com a secret gerada pela plataforma; cada webhook tem secret distinta (RF-01, RNF-10)
- [ ] Listagem retorna os webhooks do customer; edição altera url, lista de status e estado ativo; remoção exclui o webhook (RF-02, RF-03, RF-04)
- [ ] Cadastro ou edição com url `http` é recusado com erro de validação (RF-05, RNF-11)
- [ ] Mudança para um status fora da lista de todos os webhooks do customer não gera evento (RF-06)
- [ ] O endpoint do cliente recebe o payload com `event_id`, `event_type` `order.status_changed`, timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e `total_cents`, sem items (RF-07)
- [ ] A entrega contém `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`, e `X-Signature` confere com o HMAC-SHA256 do corpo calculado com a secret do endpoint (RF-08)
- [ ] Com o worker rodando, o evento chega ao endpoint do cliente em menos de 10 segundos após a mudança de status (RNF-01)
- [ ] Endpoint que não responde em 10 segundos gera falha da tentativa e nova tentativa conforme o backoff (RF-09, RNF-03)
- [ ] Um evento recebe as 5 tentativas com backoff; ao esgotar, está na dead letter com payload, motivo da falha e timestamp (RF-09, RNF-04)
- [ ] Replay da dead letter é recusado para OPERATOR; para ADMIN, recoloca o evento na outbox como pendente e registra em log o usuário que o executou (RF-10, RNF-12, RNF-13)
- [ ] Após a rotação, a secret antiga continua válida por 24 horas e depois deixa de valer (RF-11)
- [ ] O histórico de entregas mostra sucesso ou falha, payload, response e tempo de resposta (RF-12)
- [ ] Falha forçada na inserção da outbox faz a mudança de status falhar sem alterar status, histórico nem estoque (RNF-06)
- [ ] Payload acima de 64KB não é enviado e gera erro (RNF-14)
- [ ] Todos os erros do módulo usam códigos com prefixo `WEBHOOK_` (RNF-15)

---

### 12. Estratégia de testes e validação

| Tipo de validação | O que valida | Responsável | Critério de sucesso |
| --- | --- | --- | --- |
| Revisão técnica de design | Doc de design da feature antes de codar | Larissa, com Bruno e Diego | Design revisado antes do início da implementação |
| Testes ponta a ponta | Integração com `changeStatus`, CRUD, entrega, retry, dead letter e replay (RF-01 a RF-12, RNF-01 a RNF-15), com Vitest e Supertest contra o MySQL real, no padrão de `tests/orders.test.ts` | não definido | Critérios de aceitação da seção 11 atendidos |
| Revisão de segurança | Código de HMAC e de geração de secret (RF-08, RF-11, RNF-10) | Sofia | Revisão de ao menos dois dias úteis concluída antes do deploy |
