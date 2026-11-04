# Análise da transcrição

## Triagem da transcrição

### Considerado

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| TRI-C-01 | `docs/transcription_analysis.md` | Contexto | Atlas Comercial, MaxDistribuição e Nova Cargo pediram formalmente notificação em tempo real quando o status dos pedidos muda | TRANSCRICAO | [09:00] Marcos |
| TRI-C-02 | `docs/transcription_analysis.md` | Contexto | Hoje os clientes consultam `GET /orders` periodicamente, o que deixa a integração lenta e cara para eles | TRANSCRICAO | [09:00] Marcos |
| TRI-C-03 | `docs/transcription_analysis.md` | Contexto | A Atlas sugeriu que pode migrar para o concorrente se a entrega não ocorrer até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| TRI-C-04 | `docs/transcription_analysis.md` | Requisito Não Funcional | Para os clientes, "tempo real" é qualquer entrega abaixo de 10 segundos | TRANSCRICAO | [09:01] Bruno; [09:02] Marcos |
| TRI-C-05 | `docs/transcription_analysis.md` | Decisão | Webhooks são apenas de saída (outbound), da plataforma para os clientes | TRANSCRICAO | [09:02] Sofia; [09:02] Marcos; [09:03] Sofia |
| TRI-C-06 | `docs/transcription_analysis.md` | Decisão | Usar padrão outbox no MySQL existente | TRANSCRICAO | [09:06] Diego; [09:07] Diego; [09:08] Larissa; [09:48] Larissa |
| TRI-C-07 | `docs/transcription_analysis.md` | Decisão | O evento é inserido na `webhook_outbox` dentro da mesma transação de `changeStatus` no service de orders | TRANSCRICAO | [09:06] Diego; [09:40] Bruno; [09:41] Diego; [09:48] Larissa |
| TRI-C-08 | `docs/transcription_analysis.md` | Requisito Funcional | Se a inserção na outbox falhar, a transação da mudança de status sofre rollback | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| TRI-C-09 | `docs/transcription_analysis.md` | Decisão | Função `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebe o tx da transação atual e é chamada pelo `order.service` | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| TRI-C-10 | `docs/transcription_analysis.md` | Requisito Funcional | Status da outbox: pendente, processando, falhou e entregue | TRANSCRICAO | [09:08] Diego |
| TRI-C-11 | `docs/transcription_analysis.md` | Requisito Não Funcional | Outbox com índices no campo de status e em `created_at` | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| TRI-C-12 | `docs/transcription_analysis.md` | Requisito Funcional | Worker lê apenas eventos pendentes, em batch pequeno, processa e marca como entregue | TRANSCRICAO | [09:08] Diego |
| TRI-C-13 | `docs/transcription_analysis.md` | Decisão | Id da outbox em UUID, seguindo o padrão do projeto (definido após a saída de Marcos e Sofia) | TRANSCRICAO | [09:51] Diego; [09:51] Larissa |
| TRI-C-14 | `docs/transcription_analysis.md` | Decisão | Outbox guarda o payload já renderizado na inserção, como snapshot do estado de quando o status mudou (definido após a saída de Marcos e Sofia) | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa; [09:52] Diego; [09:52] Bruno |
| TRI-C-15 | `docs/transcription_analysis.md` | Decisão | Worker faz polling em loop a cada 2 segundos, buscando os eventos pendentes mais antigos | TRANSCRICAO | [09:09] Diego; [09:10] Marcos; [09:10] Larissa; [09:48] Larissa |
| TRI-C-16 | `docs/transcription_analysis.md` | Limitação Conhecida | Latência de 2 segundos decorrente do polling foi aceita ("mínima" e "no pior caso", nas palavras da reunião) | TRANSCRICAO | [09:10] Larissa |
| TRI-C-17 | `docs/transcription_analysis.md` | Decisão | Worker roda em processo separado da API | TRANSCRICAO | [09:11] Diego; [09:11] Larissa; [09:48] Larissa |
| TRI-C-18 | `docs/transcription_analysis.md` | Decisão | Nova entry-point `src/worker.ts` com script `npm run worker` | TRANSCRICAO | [09:11] Larissa; [09:28] Bruno; [09:28] Diego |
| TRI-C-19 | `docs/transcription_analysis.md` | Decisão | Lógica de processamento do worker fica em arquivo do módulo, "tipo" `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts` | TRANSCRICAO | [09:28] Bruno; [09:28] Diego |
| TRI-C-20 | `docs/transcription_analysis.md` | Restrição | Worker usa o mesmo banco e a mesma stack da API | TRANSCRICAO | [09:11] Bruno; [09:11] Diego |
| TRI-C-21 | `docs/transcription_analysis.md` | Decisão | Worker usa instância própria de `PrismaClient`, com a mesma `DATABASE_URL` | TRANSCRICAO | [09:29] Diego; [09:30] Bruno |
| TRI-C-22 | `docs/transcription_analysis.md` | Decisão | Por enquanto, single-worker processando em ordem de `created_at` da outbox | TRANSCRICAO | [09:12] Diego |
| TRI-C-23 | `docs/transcription_analysis.md` | Limitação Conhecida | Não há garantia de ordering global: a ordem vale só por `order_id` e enquanto houver um único worker | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:14] Marcos |
| TRI-C-24 | `docs/transcription_analysis.md` | Decisão | Retry com backoff exponencial e, ao atingir o teto de tentativas, falha permanente com envio para DLQ | TRANSCRICAO | [09:15] Diego; [09:17] Larissa; [09:48] Larissa |
| TRI-C-25 | `docs/transcription_analysis.md` | Decisão | Total de 5 tentativas | TRANSCRICAO | [09:15] Diego; [09:16] Larissa; [09:17] Larissa; [09:48] Larissa |
| TRI-C-26 | `docs/transcription_analysis.md` | Decisão | Progressão do backoff: 1m, 5m, 30m, 2h, 12h | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:48] Larissa |
| TRI-C-27 | `docs/transcription_analysis.md` | Limitação Conhecida | Janela de quase 15 horas entre a primeira falha e a última tentativa foi considerada aceitável | TRANSCRICAO | [09:17] Diego; [09:17] Marcos |
| TRI-C-28 | `docs/transcription_analysis.md` | Decisão | DLQ em tabela separada `webhook_dead_letter`, com payload, motivo da falha e timestamp | TRANSCRICAO | [09:18] Diego; [09:18] Bruno; [09:48] Larissa |
| TRI-C-29 | `docs/transcription_analysis.md` | Requisito Funcional | Replay manual de DLQ via `POST /admin/webhooks/dead-letter/:id/replay`, recolocando o evento na outbox como pendente | TRANSCRICAO | [09:18] Diego; [09:19] Larissa; [09:35] Diego |
| TRI-C-30 | `docs/transcription_analysis.md` | Requisito Funcional | Endpoint de replay exige role ADMIN, reaproveitando o `requireRole` existente | TRANSCRICAO | [09:35] Larissa; [09:36] Sofia; [09:36] Larissa; [09:48] Larissa |
| TRI-C-31 | `docs/transcription_analysis.md` | Requisito Funcional | Endpoint de replay registra em log quem executou o replay, para auditoria | TRANSCRICAO | [09:36] Sofia |
| TRI-C-32 | `docs/transcription_analysis.md` | Decisão | Assinatura HMAC-SHA256 sobre o corpo do request | TRANSCRICAO | [09:20] Sofia; [09:22] Sofia; [09:48] Larissa |
| TRI-C-33 | `docs/transcription_analysis.md` | Decisão | Secret única por endpoint de webhook | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia; [09:48] Larissa |
| TRI-C-34 | `docs/transcription_analysis.md` | Requisito Funcional | Endpoint para o cliente solicitar nova secret pela API (rotação) | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| TRI-C-35 | `docs/transcription_analysis.md` | Decisão | Na rotação, a secret antiga permanece válida em paralelo por 24 horas | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia; [09:48] Larissa |
| TRI-C-36 | `docs/transcription_analysis.md` | Requisito Funcional | Configuração de webhook armazena url, secret, `customer_id` e estado ativo | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia |
| TRI-C-37 | `docs/transcription_analysis.md` | Requisito Não Funcional | TLS obrigatório: URL `http` é recusada com erro de validação no schema Zod | TRANSCRICAO | [09:23] Sofia |
| TRI-C-38 | `docs/transcription_analysis.md` | Requisito Não Funcional | Limite de payload de 64KB, com erro se ultrapassado | TRANSCRICAO | [09:23] Sofia; [09:24] Diego; [09:24] Larissa |
| TRI-C-39 | `docs/transcription_analysis.md` | Decisão | Garantia de entrega at-least-once | TRANSCRICAO | [09:24] Diego; [09:26] Larissa; [09:48] Larissa |
| TRI-C-40 | `docs/transcription_analysis.md` | Decisão | Deduplicação do lado do cliente via `X-Event-Id`, com UUID único gerado quando o evento entra na outbox | TRANSCRICAO | [09:25] Diego; [09:26] Larissa; [09:48] Larissa |
| TRI-C-41 | `docs/transcription_analysis.md` | Decisão | Webhooks em módulo `src/modules/webhooks`, seguindo o padrão de controller, service, repository, routes e schemas | TRANSCRICAO | [09:27] Bruno; [09:28] Diego; [09:30] Larissa; [09:48] Larissa |
| TRI-C-42 | `docs/transcription_analysis.md` | Decisão | Erros do módulo seguem o padrão `AppError` e classes específicas | TRANSCRICAO | [09:28] Bruno; [09:30] Larissa; [09:48] Larissa |
| TRI-C-43 | `docs/transcription_analysis.md` | Decisão | Códigos de erro do módulo usam o prefixo `WEBHOOK_` (ex.: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`) | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa; [09:48] Larissa |
| TRI-C-44 | `docs/transcription_analysis.md` | Decisão | Logs com o Pino já existente, sem adicionar nova ferramenta | TRANSCRICAO | [09:29] Bruno; [09:30] Larissa; [09:48] Larissa |
| TRI-C-45 | `docs/transcription_analysis.md` | Decisão | Reaproveitar o middleware de erro centralizado sem alterações | TRANSCRICAO | [09:29] Bruno; [09:30] Larissa; [09:48] Larissa |
| TRI-C-46 | `docs/transcription_analysis.md` | Decisão | Reaproveitar o padrão de schemas Zod do projeto | TRANSCRICAO | [09:30] Larissa |
| TRI-C-47 | `docs/transcription_analysis.md` | Requisito Funcional | Cadastro de webhook via `POST` com url e lista de status desejados | TRANSCRICAO | [09:31] Marcos |
| TRI-C-48 | `docs/transcription_analysis.md` | Requisito Funcional | Secret é gerada pela plataforma e devolvida na criação do webhook | TRANSCRICAO | [09:31] Marcos |
| TRI-C-49 | `docs/transcription_analysis.md` | Decisão | Cadastro é feito direto pela API da plataforma, autenticado com JWT do sistema por usuários que representam o cliente | TRANSCRICAO | [09:32] Bruno; [09:32] Marcos; [09:32] Larissa |
| TRI-C-50 | `docs/transcription_analysis.md` | Decisão | `customer_id` é passado no body ou no path; não vem do JWT (posição final, corrige [09:31] Marcos) | TRANSCRICAO | [09:32] Bruno; [09:32] Larissa; [09:33] Bruno |
| TRI-C-51 | `docs/transcription_analysis.md` | Requisito Funcional | `PATCH` para editar configuração de webhook | TRANSCRICAO | [09:33] Bruno |
| TRI-C-52 | `docs/transcription_analysis.md` | Requisito Funcional | `DELETE` para remover configuração de webhook | TRANSCRICAO | [09:33] Bruno |
| TRI-C-53 | `docs/transcription_analysis.md` | Requisito Funcional | `GET` para listar os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| TRI-C-54 | `docs/transcription_analysis.md` | Requisito Funcional | Cada endpoint escolhe os eventos a receber por meio de uma lista de status | TRANSCRICAO | [09:33] Bruno; [09:33] Marcos |
| TRI-C-55 | `docs/transcription_analysis.md` | Decisão | Filtro de eventos é aplicado na inserção da outbox; se nenhum webhook do customer quer o status, nada é inserido | TRANSCRICAO | [09:33] Marcos; [09:34] Diego; [09:34] Bruno |
| TRI-C-56 | `docs/transcription_analysis.md` | Requisito Funcional | `GET /webhooks/:id/deliveries` mostra o histórico de entregas (ex.: últimos 100), com sucesso/falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos; [09:35] Larissa |
| TRI-C-57 | `docs/transcription_analysis.md` | Decisão | CRUD de configuração de webhook aceita qualquer role autenticada, por enquanto | TRANSCRICAO | [09:36] Marcos; [09:37] Sofia; [09:48] Larissa |
| TRI-C-58 | `docs/transcription_analysis.md` | Ponto em Aberto | Rate limiting de envio para o cliente: observar e decidir depois | TRANSCRICAO | [09:38] Diego; [09:39] Diego; [09:39] Larissa; [09:48] Larissa |
| TRI-C-59 | `docs/transcription_analysis.md` | Requisito Não Funcional | Timeout de 10 segundos na chamada HTTP do worker; sem resposta nesse prazo, trata como falha e marca para retry | TRANSCRICAO | [09:42] Sofia; [09:42] Diego |
| TRI-C-60 | `docs/transcription_analysis.md` | Requisito Funcional | Payload JSON com `event_id`, `event_type` (ex.: `order.status_changed`), timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos da order, como `total_cents` | TRANSCRICAO | [09:43] Marcos; [09:43] Diego; [09:44] Bruno |
| TRI-C-61 | `docs/transcription_analysis.md` | Requisito Funcional | Header `X-Event-Id` com o UUID do evento | TRANSCRICAO | [09:44] Diego |
| TRI-C-62 | `docs/transcription_analysis.md` | Requisito Funcional | Header `X-Signature` com o HMAC | TRANSCRICAO | [09:20] Sofia; [09:44] Diego |
| TRI-C-63 | `docs/transcription_analysis.md` | Requisito Funcional | Header `X-Timestamp` com o timestamp do envio, para o cliente poder detectar replay attack | TRANSCRICAO | [09:44] Diego |
| TRI-C-64 | `docs/transcription_analysis.md` | Requisito Funcional | Header `Content-Type: application/json` | TRANSCRICAO | [09:44] Diego |
| TRI-C-65 | `docs/transcription_analysis.md` | Requisito Funcional | Header `X-Webhook-Id` com o id do endpoint de webhook | TRANSCRICAO | [09:44] Sofia; [09:45] Diego |
| TRI-C-66 | `docs/transcription_analysis.md` | Restrição | Atlas quer a entrega para o fim de novembro | TRANSCRICAO | [09:45] Marcos |
| TRI-C-67 | `docs/transcription_analysis.md` | Decisão | Prazo estimado de três sprints, com a revisão de segurança incluída no fim | TRANSCRICAO | [09:46] Larissa; [09:47] Larissa; [09:48] Larissa |
| TRI-C-68 | `docs/transcription_analysis.md` | Restrição | Reservar ao menos dois dias úteis para Sofia revisar o código de segurança (HMAC e geração de secret) antes do deploy | TRANSCRICAO | [09:46] Sofia; [09:47] Larissa; [09:49] Sofia |
| TRI-C-69 | `docs/transcription_analysis.md` | Contexto | Marcos documentará no portal de desenvolvedor, com destaque, a possibilidade de eventos duplicados e a dedup por `X-Event-Id` | TRANSCRICAO | [09:26] Marcos |
| TRI-C-70 | `docs/transcription_analysis.md` | Contexto | Marcos documentará no portal como os clientes integram via API | TRANSCRICAO | [09:40] Marcos |
| TRI-C-71 | `docs/transcription_analysis.md` | Contexto | Larissa abrirá o doc de design e marcará uma revisão com Bruno e Diego antes de começar a codar | TRANSCRICAO | [09:50] Larissa |

### Descartado

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| TRI-D-01 | `docs/transcription_analysis.md` | Alternativa Descartada | Disparar o webhook de forma síncrona dentro do service de orders | TRANSCRICAO | [09:03] Larissa; [09:04] Bruno; [09:04] Larissa; [09:06] Diego |
| TRI-D-02 | `docs/transcription_analysis.md` | Alternativa Descartada | Dar rollback na mudança de status quando o cliente estiver fora do ar | TRANSCRICAO | [09:04] Bruno |
| TRI-D-03 | `docs/transcription_analysis.md` | Alternativa Descartada | Usar Redis Streams ou infra semelhante (Redis Cluster) no lugar da outbox em MySQL | TRANSCRICAO | [09:07] Larissa; [09:07] Diego; [09:08] Larissa |
| TRI-D-04 | `docs/transcription_analysis.md` | Alternativa Descartada | Usar trigger do banco para tornar o worker reativo, em vez de polling | TRANSCRICAO | [09:09] Bruno; [09:09] Diego; [09:10] Larissa |
| TRI-D-05 | `docs/transcription_analysis.md` | Alternativa Descartada | Rodar o worker no mesmo processo da API | TRANSCRICAO | [09:11] Diego |
| TRI-D-06 | `docs/transcription_analysis.md` | Alternativa Descartada | Compartilhar a mesma instância de `PrismaClient` entre API e worker | TRANSCRICAO | [09:29] Diego; [09:30] Bruno |
| TRI-D-07 | `docs/transcription_analysis.md` | Alternativa Descartada | Limitar o retry a 3 tentativas | TRANSCRICAO | [09:16] Bruno; [09:16] Diego; [09:16] Larissa |
| TRI-D-08 | `docs/transcription_analysis.md` | Alternativa Descartada | Retry indefinido com backoff | TRANSCRICAO | [09:15] Diego; [09:16] Larissa |
| TRI-D-09 | `docs/transcription_analysis.md` | Alternativa Descartada | Marcar a DLQ como `failed` na própria outbox, em vez de tabela separada | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:48] Larissa |
| TRI-D-10 | `docs/transcription_analysis.md` | Alternativa Descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| TRI-D-11 | `docs/transcription_analysis.md` | Alternativa Descartada | Truncar o payload que exceder o limite de tamanho | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| TRI-D-12 | `docs/transcription_analysis.md` | Alternativa Descartada | Garantia de entrega exactly-once | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| TRI-D-13 | `docs/transcription_analysis.md` | Alternativa Descartada | `customer_id` implícito do JWT (posição inicial, corrigida em [09:32] Larissa) | TRANSCRICAO | [09:31] Marcos; [09:32] Bruno; [09:32] Larissa |
| TRI-D-14 | `docs/transcription_analysis.md` | Alternativa Descartada | Cadastro de webhook pelo painel do cliente | TRANSCRICAO | [09:32] Bruno; [09:32] Marcos |
| TRI-D-15 | `docs/transcription_analysis.md` | Alternativa Descartada | Aplicar o filtro de eventos na hora do envio, em vez de na inserção da outbox | TRANSCRICAO | [09:34] Diego; [09:34] Bruno |
| TRI-D-16 | `docs/transcription_analysis.md` | Alternativa Descartada | Injetar o repository inteiro de webhook no `OrderService` | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| TRI-D-17 | `docs/transcription_analysis.md` | Alternativa Descartada | Incluir os items do pedido no payload; quem quiser detalhes consulta `GET /orders/:id` | TRANSCRICAO | [09:43] Diego; [09:44] Bruno |
| TRI-D-18 | `docs/transcription_analysis.md` | Alternativa Descartada | Id auto incremental na outbox | TRANSCRICAO | [09:51] Diego; [09:51] Larissa |
| TRI-D-19 | `docs/transcription_analysis.md` | Alternativa Descartada | Guardar só o `order_id` na outbox e renderizar o payload na hora do envio | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa; [09:52] Diego; [09:52] Bruno |
| TRI-D-20 | `docs/transcription_analysis.md` | Fora de Escopo | Webhooks de entrada (dos clientes para a plataforma) | TRANSCRICAO | [09:02] Sofia; [09:02] Marcos |
| TRI-D-21 | `docs/transcription_analysis.md` | Fora de Escopo | Arquivamento de linhas entregues da outbox após cerca de 30 dias | TRANSCRICAO | [09:08] Diego |
| TRI-D-22 | `docs/transcription_analysis.md` | Fora de Escopo | Múltiplos workers em paralelo, por ser "problema do futuro" | TRANSCRICAO | [09:12] Diego; [09:13] Bruno; [09:13] Diego |
| TRI-D-23 | `docs/transcription_analysis.md` | Fora de Escopo | Particionamento por `order_id` ou lock pessimista para escalar, por ser "problema do futuro" | TRANSCRICAO | [09:13] Bruno; [09:13] Diego |
| TRI-D-24 | `docs/transcription_analysis.md` | Fora de Escopo | Endurecer as roles do CRUD de configuração, possível "mais pra frente" | TRANSCRICAO | [09:36] Marcos; [09:37] Sofia |
| TRI-D-25 | `docs/transcription_analysis.md` | Fora de Escopo | Aviso por e-mail ao cliente quando o webhook apresenta falhas; talvez na próxima fase | TRANSCRICAO | [09:37] Marcos; [09:37] Larissa; [09:38] Marcos; [09:48] Larissa |
| TRI-D-26 | `docs/transcription_analysis.md` | Fora de Escopo | Dashboard visual para o cliente; painel é projeto separado do time de frontend | TRANSCRICAO | [09:39] Marcos; [09:40] Larissa; [09:48] Larissa |
## Candidatas a ADR

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| CAND-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Mantida como ADR | Outbox no MySQL, na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego; [09:08] Larissa |
| CAND-02 | `docs/adrs/ADR-002-worker-em-polling-de-2-segundos.md` | Mantida como ADR | Worker em polling de 2 segundos | TRANSCRICAO | [09:09] Diego; [09:10] Larissa |
| CAND-03 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Mantida como ADR | Worker em processo separado da API | TRANSCRICAO | [09:11] Diego; [09:11] Larissa |
| CAND-04 | `docs/adrs/ADR-004-retry-com-backoff-exponencial.md` | Mantida como ADR | 5 tentativas com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| CAND-05 | `docs/adrs/ADR-005-dead-letter-em-tabela-separada.md` | Mantida como ADR | DLQ em tabela separada | TRANSCRICAO | [09:18] Diego |
| CAND-06 | `docs/adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md` | Mantida como ADR | HMAC-SHA256, secret por endpoint e rotação com grace period de 24h | TRANSCRICAO | [09:22] Sofia |
| CAND-07 | `docs/adrs/ADR-007-entrega-at-least-once-com-event-id.md` | Mantida como ADR | At-least-once com dedup por `X-Event-Id` | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| CAND-08 | `docs/adrs/ADR-008-payload-snapshot-na-insercao.md` | Mantida como ADR | Snapshot do payload na inserção | TRANSCRICAO | [09:52] Larissa; [09:52] Bruno |
| CAND-09 | `docs/transcription_analysis.md` | Movida para FDD | Reuso de `AppError`, Pino, error middleware e padrão de módulos: segue convenção existente, sem nova decisão estrutural | TRANSCRICAO | [09:30] Larissa |
| CAND-10 | `docs/transcription_analysis.md` | Movida para RFC ou FDD | Single-worker e ordering por `order_id`: limitação conhecida, descrita como "por enquanto", logo não é estável | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| CAND-11 | `docs/transcription_analysis.md` | Movida para FDD | Filtro de eventos na inserção da outbox: regra de implementação localizada | TRANSCRICAO | [09:34] Bruno |
| CAND-12 | `docs/transcription_analysis.md` | Movida para FDD | Função `publishWebhookEvent(tx, order, fromStatus, toStatus)`: detalhe de integração da ADR-001 | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| CAND-13 | `docs/transcription_analysis.md` | Movida para FDD | Replay manual com role ADMIN e log de auditoria; CRUD com qualquer role autenticada "por enquanto" | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa; [09:37] Sofia |
| CAND-14 | `docs/transcription_analysis.md` | Movida para FDD | Origem do `customer_id` no body ou no path, não no JWT | TRANSCRICAO | [09:32] Larissa |
| CAND-15 | `docs/transcription_analysis.md` | Sem ADR, requisito | TLS obrigatório e limite de 64KB: a própria reunião os classificou como não arquiteturais | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |

## Verificação das ADRs

- Citações: todas as referências `[hh:mm] Nome` das 8 ADRs, de `docs/TRACKER.md` e desta análise existem em `TRANSCRICAO.md`, conferidas por script.
- Caminhos: todos os arquivos citados existem no código. A exceção é `src/worker.ts`, que é o arquivo novo decidido em [09:11] Larissa.
- Template: as 8 ADRs têm os metadados e as 5 seções de `docs/templates/ADR.md`, na mesma ordem, conferidos por script.
- Conteúdo: decisões, alternativas e consequências foram relidas contra os trechos citados. Dois trechos dedutivos foram removidos das ADR-004 e ADR-006.
