# Tracker

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-001 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Outbox no MySQL, com inserção na transação de `changeStatus` | TRANSCRICAO | [09:06] Diego; [09:07] Diego; [09:08] Larissa; [09:48] Larissa |
| ADR-001-COD-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Integração | `changeStatus` executa a mudança de status em `prisma.$transaction` | CODIGO | `src/modules/orders/order.service.ts` |
| ADR-001-COD-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Integração | Regras de débito e reposição de estoque na transição de status | CODIGO | `src/modules/orders/order.status.ts` |
| ADR-001-COD-03 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Integração | Modelo `OrderStatusHistory` gravado na mesma transação | CODIGO | `prisma/schema.prisma` |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Disparo síncrono no service de orders | TRANSCRICAO | [09:03] Larissa; [09:04] Bruno; [09:06] Diego |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Redis Streams ou similar | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| ADR-001-TO-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Trade-off | Falha na outbox desfaz a mudança de status, em troca de nunca haver status sem evento | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| ADR-002 | `docs/adrs/ADR-002-worker-em-polling-de-2-segundos.md` | Decisão | Polling em loop a cada 2 segundos | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:48] Larissa |
| ADR-002-COD-01 | `docs/adrs/ADR-002-worker-em-polling-de-2-segundos.md` | Restrição | Banco do projeto é MySQL | CODIGO | `prisma/schema.prisma` |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-em-polling-de-2-segundos.md` | Alternativa Descartada | Trigger do banco | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| ADR-002-TO-01 | `docs/adrs/ADR-002-worker-em-polling-de-2-segundos.md` | Trade-off | Latência do polling aceita, em troca de não improvisar notificação a partir do banco | TRANSCRICAO | [09:09] Diego; [09:10] Larissa |
| ADR-003 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Decisão | Processo separado com `src/worker.ts`, `npm run worker` e `PrismaClient` próprio | TRANSCRICAO | [09:11] Diego; [09:11] Larissa; [09:30] Bruno; [09:48] Larissa |
| ADR-003-COD-01 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Integração | Entry-point atual da API | CODIGO | `src/server.ts` |
| ADR-003-COD-02 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Integração | Criação do `PrismaClient` usado pela API | CODIGO | `src/config/database.ts` |
| ADR-003-COD-03 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Integração | Scripts do projeto | CODIGO | `package.json` |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Alternativa Descartada | Worker dentro da instância da API | TRANSCRICAO | [09:11] Diego |
| ADR-003-TO-01 | `docs/adrs/ADR-003-worker-em-processo-separado.md` | Trade-off | Processo adicional, em troca de independência do ciclo de vida da API | TRANSCRICAO | [09:11] Diego; [09:30] Bruno |
| ADR-004 | `docs/adrs/ADR-004-retry-com-backoff-exponencial.md` | Decisão | 5 tentativas com backoff 1m/5m/30m/2h/12h e depois DLQ | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:17] Larissa; [09:48] Larissa |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-retry-com-backoff-exponencial.md` | Alternativa Descartada | 3 tentativas | TRANSCRICAO | [09:16] Bruno; [09:16] Diego; [09:16] Larissa |
| ADR-004-ALT-02 | `docs/adrs/ADR-004-retry-com-backoff-exponencial.md` | Alternativa Descartada | Retry indefinido com backoff | TRANSCRICAO | [09:15] Diego |
| ADR-004-TO-01 | `docs/adrs/ADR-004-retry-com-backoff-exponencial.md` | Trade-off | Falha permanente após quase 15 horas, em troca de não ter eventos pendurados | TRANSCRICAO | [09:15] Diego; [09:17] Marcos |
| ADR-005 | `docs/adrs/ADR-005-dead-letter-em-tabela-separada.md` | Decisão | Tabela `webhook_dead_letter` com payload, motivo da falha e timestamp | TRANSCRICAO | [09:18] Diego; [09:48] Larissa |
| ADR-005-COD-01 | `docs/adrs/ADR-005-dead-letter-em-tabela-separada.md` | Integração | `requireRole`, a reaproveitar no endpoint de replay | CODIGO | `src/middlewares/auth.middleware.ts` |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-dead-letter-em-tabela-separada.md` | Alternativa Descartada | Marcar como "failed" na própria outbox | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| ADR-005-TO-01 | `docs/adrs/ADR-005-dead-letter-em-tabela-separada.md` | Trade-off | Tabela adicional e replay manual, em troca de outbox mais limpa e evidência para debug | TRANSCRICAO | [09:18] Diego |
| ADR-006 | `docs/adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md` | Decisão | HMAC-SHA256, secret por endpoint e rotação com grace period de 24h | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:48] Larissa |
| ADR-006-ALT-01 | `docs/adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md` | Alternativa Descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-006-TO-01 | `docs/adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md` | Trade-off | Duas secrets válidas por 24 horas, em troca de tempo para o cliente migrar | TRANSCRICAO | [09:21] Sofia |
| ADR-007 | `docs/adrs/ADR-007-entrega-at-least-once-com-event-id.md` | Decisão | At-least-once com UUID em `X-Event-Id` | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa; [09:48] Larissa |
| ADR-007-ALT-01 | `docs/adrs/ADR-007-entrega-at-least-once-com-event-id.md` | Alternativa Descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-007-TO-01 | `docs/adrs/ADR-007-entrega-at-least-once-com-event-id.md` | Trade-off | Deduplicação no cliente, em troca de evitar a complexidade de exactly-once | TRANSCRICAO | [09:25] Sofia; [09:25] Diego |
| ADR-008 | `docs/adrs/ADR-008-payload-snapshot-na-insercao.md` | Decisão | Payload renderizado na inserção da outbox | TRANSCRICAO | [09:52] Larissa; [09:52] Diego; [09:52] Bruno |
| ADR-008-COD-01 | `docs/adrs/ADR-008-payload-snapshot-na-insercao.md` | Integração | `changeStatus`, ponto em que a mudança de status ocorre | CODIGO | `src/modules/orders/order.service.ts` |
| ADR-008-ALT-01 | `docs/adrs/ADR-008-payload-snapshot-na-insercao.md` | Alternativa Descartada | Guardar só o `order_id` e renderizar no envio | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| LAC-01 | `docs/adrs/ADR-004-retry-com-backoff-exponencial.md` | Ponto em Aberto | Não foi dito se as 5 tentativas incluem o envio inicial, embora a progressão tenha 5 intervalos | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| LAC-02 | `docs/adrs/ADR-005-dead-letter-em-tabela-separada.md` | Ponto em Aberto | O papel do status "falhou" da outbox ao lado da DLQ não foi definido | TRANSCRICAO | [09:08] Diego; [09:18] Diego |
| LAC-03 | `docs/adrs/ADR-006-assinatura-hmac-sha256-por-endpoint.md` | Ponto em Aberto | A forma de armazenar a secret e qual secret assina durante o grace period não foram definidas | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| RFC-META-01 | `docs/RFC.md` | Contexto | Autoria do RFC atribuída a Larissa, que abrirá o doc de design e marcará revisão com Bruno e Diego | TRANSCRICAO | [09:50] Larissa |
| RFC-META-02 | `docs/RFC.md` | Contexto | Revisores são os cinco participantes da reunião, com seus papéis | TRANSCRICAO | Cabeçalho, lista de Participantes |
| RFC-CTX-01 | `docs/RFC.md` | Contexto | Clientes consultam `GET /orders` periodicamente, o que deixa a integração lenta e cara | TRANSCRICAO | [09:00] Marcos |
| RFC-CTX-02 | `docs/RFC.md` | Requisito Não Funcional | Entrega abaixo de 10 segundos é considerada "tempo real" pelos clientes | TRANSCRICAO | [09:02] Marcos |
| RFC-CTX-03 | `docs/RFC.md` | Restrição | Transação de `changeStatus` já é pesada; HTTP síncrono travaria outros pedidos | TRANSCRICAO | [09:04] Bruno |
| RFC-CTX-04 | `docs/RFC.md` | Integração | `changeStatus` executa a mudança de status, histórico e estoque em transação | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-CTX-05 | `docs/RFC.md` | Integração | Regras de movimentação de estoque na transição de status | CODIGO | `src/modules/orders/order.status.ts` |
| RFC-CTX-06 | `docs/RFC.md` | Contexto | Não existe módulo de webhooks entre os módulos atuais | CODIGO | `src/modules/` |
| RFC-CTX-07 | `docs/RFC.md` | Restrição | Escopo apenas de webhooks de saída | TRANSCRICAO | [09:02] Marcos; [09:03] Sofia |
| RFC-PROP-01 | `docs/RFC.md` | Decisão | Outbox no MySQL na transação de `changeStatus`, com rollback se a inserção falhar (consolida ADR-001) | TRANSCRICAO | [09:06] Diego; [09:40] Bruno; [09:41] Diego |
| RFC-PROP-02 | `docs/RFC.md` | Decisão | `webhook_outbox` com status, índices em status e `created_at`, id UUID e payload snapshot (consolida ADR-001 e ADR-008) | TRANSCRICAO | [09:08] Diego; [09:51] Larissa; [09:52] Larissa |
| RFC-PROP-03 | `docs/RFC.md` | Decisão | Configuração de webhook com url, secret, `customer_id`, estado ativo e lista de status | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia; [09:33] Marcos |
| RFC-PROP-04 | `docs/RFC.md` | Decisão | Worker em processo separado com polling de 2 segundos, suficiente para o requisito de latência (consolida ADR-002 e ADR-003) | TRANSCRICAO | [09:09] Diego; [09:10] Marcos; [09:11] Diego; [09:11] Larissa; [09:30] Bruno |
| RFC-PROP-05 | `docs/RFC.md` | Decisão | Retry 1m/5m/30m/2h/12h e DLQ em `webhook_dead_letter` com replay manual (consolida ADR-004 e ADR-005) | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:18] Diego |
| RFC-PROP-06 | `docs/RFC.md` | Decisão | HMAC-SHA256 em `X-Signature` com secret por endpoint e at-least-once com `X-Event-Id` (consolida ADR-006 e ADR-007) | TRANSCRICAO | [09:20] Sofia; [09:22] Sofia; [09:25] Diego; [09:26] Larissa; [09:44] Diego |
| RFC-PROP-07 | `docs/RFC.md` | Decisão | Módulo `src/modules/webhooks` reaproveitando `AppError`, Pino e middleware de erro | TRANSCRICAO | [09:27] Bruno; [09:30] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa Descartada | Disparo síncrono no service de orders (ver ADR-001-ALT-01) | TRANSCRICAO | [09:03] Larissa; [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa Descartada | Redis Streams ou similar (ver ADR-001-ALT-02) | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa Descartada | Trigger do banco para notificar o worker (ver ADR-002-ALT-01) | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa Descartada | Garantia exactly-once (ver ADR-007-ALT-01) | TRANSCRICAO | [09:25] Sofia; [09:25] Diego |
| RFC-OPEN-01 | `docs/RFC.md` | Ponto em Aberto | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:38] Diego; [09:39] Diego; [09:39] Larissa; [09:48] Larissa |
| RFC-OPEN-02 | `docs/RFC.md` | Ponto em Aberto | Escala para múltiplos workers e perda de ordenação, "problema do futuro" | TRANSCRICAO | [09:12] Diego; [09:13] Diego; [09:13] Larissa |
| RFC-OPEN-03 | `docs/RFC.md` | Decisão | Decidido após a reunião: as 5 tentativas incluem o envio inicial (ver LAC-01) | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| RFC-OPEN-04 | `docs/RFC.md` | Ponto em Aberto | Papel do status "falhou" da outbox ao lado da DLQ (ver LAC-02) | TRANSCRICAO | [09:08] Diego; [09:18] Diego |
| RFC-OPEN-05 | `docs/RFC.md` | Ponto em Aberto | Armazenamento da secret e qual secret assina no grace period (ver LAC-03) | TRANSCRICAO | [09:21] Sofia; [09:46] Sofia |
| RFC-OPEN-06 | `docs/RFC.md` | Ponto em Aberto | Com 5 tentativas incluindo o envio inicial, há 4 retentativas para 5 intervalos de backoff | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| RFC-OPEN-07 | `docs/RFC.md` | Ponto em Aberto | `customer_id` no body ou no path não foi escolhido | TRANSCRICAO | [09:32] Larissa; [09:33] Bruno |
| RFC-OPEN-08 | `docs/RFC.md` | Ponto em Aberto | Replay preserva ou não o `event_id` original | TRANSCRICAO | [09:18] Diego; [09:25] Diego |
| RFC-OPEN-09 | `docs/RFC.md` | Ponto em Aberto | Outbox com uma linha por evento ou por webhook interessado | TRANSCRICAO | [09:34] Bruno; [09:44] Sofia |
| RFC-OPEN-10 | `docs/RFC.md` | Ponto em Aberto | Momento da verificação do limite de 64KB (inserção ou envio) | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| RFC-OPEN-11 | `docs/RFC.md` | Ponto em Aberto | Lista completa dos campos básicos da order no payload | TRANSCRICAO | [09:43] Diego |
| RFC-OPEN-12 | `docs/RFC.md` | Ponto em Aberto | Rota, método e formato do endpoint de rotação de secret | TRANSCRICAO | [09:21] Sofia |
| RFC-OPEN-13 | `docs/RFC.md` | Ponto em Aberto | Persistência dos dados do histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| RFC-OPEN-14 | `docs/RFC.md` | Ponto em Aberto | Url não `https`: erro de validação Zod ou `WEBHOOK_INVALID_URL` | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno |
| RFC-OPEN-15 | `docs/RFC.md` | Ponto em Aberto | Middleware de validação atual emite `VALIDATION_ERROR` para erros do schema Zod | CODIGO | `src/middlewares/validate.middleware.ts` |
| RFC-OPEN-16 | `docs/RFC.md` | Ponto em Aberto | Situação em que ocorre `WEBHOOK_SECRET_REQUIRED`, dado que a secret é gerada pela plataforma | TRANSCRICAO | [09:28] Bruno; [09:31] Marcos |
| RFC-OPEN-17 | `docs/RFC.md` | Ponto em Aberto | Resposta do cliente que conta como sucesso da entrega | TRANSCRICAO | [09:08] Diego; [09:42] Diego |
| RFC-OPEN-18 | `docs/RFC.md` | Ponto em Aberto | Prazo da Atlas citado como fim do trimestre e como fim de novembro | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos; [09:47] Marcos |
| RFC-IMP-01 | `docs/RFC.md` | Impacto | `changeStatus` passa a gravar na outbox dentro da transação | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-IMP-02 | `docs/RFC.md` | Impacto | Novo processo `npm run worker` separado da API | TRANSCRICAO | [09:11] Diego; [09:11] Larissa |
| RFC-IMP-03 | `docs/RFC.md` | Impacto | Clientes deduplicam por `X-Event-Id` e verificam HMAC; documentação no portal | TRANSCRICAO | [09:20] Sofia; [09:26] Marcos; [09:40] Marcos |
| RFC-IMP-04 | `docs/RFC.md` | Restrição | Três sprints com ao menos dois dias úteis de revisão de segurança; Atlas quer para fim de novembro | TRANSCRICAO | [09:45] Marcos; [09:46] Sofia; [09:47] Larissa |
| RFC-RISK-01 | `docs/RFC.md` | Risco | Falha na inserção da outbox bloqueia a mudança de status, aceito como trade-off | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| RFC-RISK-02 | `docs/RFC.md` | Risco | Acúmulo de eventos na outbox, mitigado por índices, batch pequeno e arquivamento | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| RFC-RISK-03 | `docs/RFC.md` | Risco | Eventos duplicados no cliente, mitigado por `X-Event-Id` e portal | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Marcos |
| RFC-RISK-04 | `docs/RFC.md` | Risco | Cliente fora além de quase 15 horas, mitigado por DLQ e replay | TRANSCRICAO | [09:17] Diego; [09:17] Marcos; [09:18] Diego |
| RFC-RISK-05 | `docs/RFC.md` | Risco | Cliente lento, mitigado por timeout de 10 segundos com retry | TRANSCRICAO | [09:42] Sofia; [09:42] Diego |
| RFC-RISK-06 | `docs/RFC.md` | Risco | Vazamento de secret, mitigado por secret por endpoint e rotação | TRANSCRICAO | [09:21] Sofia; [09:22] Diego |
| RFC-RISK-07 | `docs/RFC.md` | Risco | Perda de ordenação com múltiplos workers, mitigado por single-worker | TRANSCRICAO | [09:12] Diego; [09:14] Marcos |
| RFC-RISK-08 | `docs/RFC.md` | Risco | Atraso na entrega com risco de migração da Atlas, mitigado pela estimativa de três sprints | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos; [09:47] Larissa |
| RFC-REL-01 | `docs/RFC.md` | Referência | Seção 7 do RFC vincula ADR-001 a ADR-008 | TRANSCRICAO | [09:48] Larissa; [09:52] Bruno |
| FDD-META-01 | `docs/FDD.md` | Contexto | Responsável pelo FDD é Larissa; versão de rascunho para revisão com Bruno e Diego antes de codar | TRANSCRICAO | [09:50] Larissa |
| FDD-CTX-01 | `docs/FDD.md` | Contexto | Clientes B2B consultam `GET /orders` periodicamente e querem notificação abaixo de 10 segundos (ver RFC-CTX-01, RFC-CTX-02) | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| FDD-CTX-02 | `docs/FDD.md` | Integração | `changeStatus` valida transição, movimenta estoque, atualiza `orders` e insere histórico em um `prisma.$transaction` | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-CTX-03 | `docs/FDD.md` | Contexto | Módulos atuais não incluem webhooks (ver RFC-CTX-06) | CODIGO | `src/modules/` |
| FDD-CTX-04 | `docs/FDD.md` | Contexto | Atores: clientes B2B, usuários autenticados que representam o cliente, ADMIN e worker | TRANSCRICAO | [09:32] Marcos; [09:36] Sofia; [09:11] Diego |
| FDD-CTX-05 | `docs/FDD.md` | Restrição | Ordenação garantida só por `order_id` e com single-worker (ver RFC-OPEN-02) | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| FDD-OBJ-01 | `docs/FDD.md` | Requisito Não Funcional | Entrega abaixo de 10 segundos, com latência de polling de até 2 segundos aceita (ver ADR-002) | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| FDD-OBJ-02 | `docs/FDD.md` | Requisito Não Funcional | Timeout de 10 segundos na chamada HTTP do worker | TRANSCRICAO | [09:42] Sofia; [09:42] Diego |
| FDD-OBJ-03 | `docs/FDD.md` | Requisito Não Funcional | Limite de payload de 64KB, com erro se ultrapassado | TRANSCRICAO | [09:23] Sofia; [09:24] Diego; [09:24] Larissa |
| FDD-OBJ-04 | `docs/FDD.md` | Decisão | Reuso de `AppError`, Pino, `errorMiddleware`, padrão de módulos e Zod, prefixo `WEBHOOK_` (ver RFC-PROP-07) | TRANSCRICAO | [09:29] Bruno; [09:29] Larissa; [09:30] Larissa |
| FDD-ESC-01 | `docs/FDD.md` | Decisão | Filtro por lista de status na inserção da outbox; sem webhook interessado, nada é inserido | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno; [09:34] Diego |
| FDD-ESC-02 | `docs/FDD.md` | Requisito Funcional | CRUD de configuração de webhook para qualquer role autenticada | TRANSCRICAO | [09:33] Bruno; [09:36] Marcos; [09:37] Sofia |
| FDD-ESC-03 | `docs/FDD.md` | Decisão | Lógica do worker em `webhook.worker.ts` ou `webhook.processor.ts` dentro do módulo | TRANSCRICAO | [09:28] Bruno; [09:28] Diego |
| FDD-ESC-04 | `docs/FDD.md` | Restrição | Fora do escopo: webhooks de entrada, email ao cliente, dashboard visual, arquivamento e exactly-once | TRANSCRICAO | [09:02] Marcos; [09:37] Larissa; [09:40] Larissa; [09:08] Diego; [09:25] Diego |
| FDD-FLUXO-01 | `docs/FDD.md` | Integração | Fluxo inicia em `PATCH /orders/:id/status`, que delega a `OrderService.changeStatus` | CODIGO | `src/modules/orders/order.routes.ts` |
| FDD-FLUXO-02 | `docs/FDD.md` | Requisito Funcional | Worker busca pendentes mais antigos em batch pequeno a cada 2 segundos, em ordem de `created_at` | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:12] Diego |
| FDD-FLUXO-03 | `docs/FDD.md` | Requisito Funcional | Replay de DLQ recoloca o evento na outbox como pendente e registra o autor em log | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-FLUXO-04 | `docs/FDD.md` | Requisito Funcional | Na rotação, a secret antiga vale 24 horas em paralelo e depois deixa de valer | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | Função `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebendo o tx da transação corrente | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | `POST /api/v1/webhooks` com url e lista de status; secret gerada e devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-03 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks` lista os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | `docs/FDD.md` | Contrato | `PATCH /api/v1/webhooks/:id` edita a configuração | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | `docs/FDD.md` | Contrato | `DELETE /api/v1/webhooks/:id` remove a configuração | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks/:id/deliveries` com sucesso/falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | `POST /api/v1/admin/webhooks/dead-letter/:id/replay` com role ADMIN | TRANSCRICAO | [09:18] Diego; [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| FDD-CONTRATO-09 | `docs/FDD.md` | Contrato | Entrega ao cliente com payload `event_id`, `event_type`, timestamp, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id`, `total_cents`, sem items | TRANSCRICAO | [09:43] Diego; [09:44] Bruno |
| FDD-CONTRATO-10 | `docs/FDD.md` | Contrato | Headers `Content-Type`, `X-Event-Id`, `X-Signature`, `X-Timestamp` e `X-Webhook-Id` | TRANSCRICAO | [09:44] Diego; [09:44] Sofia; [09:45] Diego |
| FDD-CONTRATO-11 | `docs/FDD.md` | Integração | Rotas montadas sob `/api/v1` | CODIGO | `src/routes/index.ts` |
| FDD-CONTRATO-12 | `docs/FDD.md` | Integração | Status de sucesso 201, 200 e 204 seguem os controllers existentes | CODIGO | `src/modules/orders/order.controller.ts` |
| FDD-CONTRATO-13 | `docs/FDD.md` | Integração | Campos da API de gestão em camelCase, como nos schemas existentes | CODIGO | `src/modules/orders/order.schemas.ts` |
| FDD-CONTRATO-14 | `docs/FDD.md` | Integração | Formato `ORD-000123` de `order_number` gerado por `reserveOrderNumber` | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-CONTRATO-15 | `docs/FDD.md` | Integração | Valores de status do payload e da lista do webhook vêm do enum `OrderStatus` | CODIGO | `prisma/schema.prisma` |
| FDD-ERRO-01 | `docs/FDD.md` | Decisão | Códigos `WEBHOOK_NOT_FOUND` e `WEBHOOK_INVALID_URL` na matriz de erros, com prefixo `WEBHOOK_` | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno; [09:29] Larissa |
| FDD-ERRO-03 | `docs/FDD.md` | Integração | Erros `WEBHOOK_*` estendem `AppError` | CODIGO | `src/shared/errors/app-error.ts` |
| FDD-ERRO-04 | `docs/FDD.md` | Integração | `NotFoundError` fixa `NOT_FOUND`; erros 404 do módulo precisam estender `AppError` diretamente | CODIGO | `src/shared/errors/http-errors.ts` |
| FDD-ERRO-05 | `docs/FDD.md` | Integração | `errorMiddleware` reutilizado sem alteração; erro desconhecido vira 500 `INTERNAL_SERVER_ERROR` | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-ERRO-06 | `docs/FDD.md` | Integração | `validate` converte `ZodError` em `ValidationError` 400 | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-ERRO-07 | `docs/FDD.md` | Integração | `authenticate` gera 401 e `requireRole('ADMIN')` gera 403 no replay | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-RES-01 | `docs/FDD.md` | Decisão | Resiliência por timeout de 10s, no máximo 5 tentativas com backoff exponencial, processo separado e leitura em batch (ver ADR-003, ADR-004) | TRANSCRICAO | [09:42] Diego; [09:17] Larissa; [09:11] Diego; [09:08] Diego |
| FDD-RES-02 | `docs/FDD.md` | Decisão | Fallback por DLQ e replay manual; email como fallback na próxima fase | TRANSCRICAO | [09:18] Diego; [09:48] Larissa |
| FDD-INV-01 | `docs/FDD.md` | Restrição | Invariantes: status commitado com evento, payload snapshot, url `https`, secret única, payload até 64KB | TRANSCRICAO | [09:41] Diego; [09:52] Larissa; [09:23] Sofia; [09:21] Sofia; [09:24] Larissa |
| FDD-OBS-01 | `docs/FDD.md` | Requisito Não Funcional | Métricas a partir de tempo de resposta e resultado por entrega, status da outbox e registros da DLQ | TRANSCRICAO | [09:34] Marcos; [09:08] Diego; [09:18] Diego |
| FDD-OBS-02 | `docs/FDD.md` | Integração | Logs com Pino existente, redaction e mensagens em snake_case | CODIGO | `src/shared/logger/index.ts` |
| FDD-OBS-03 | `docs/FDD.md` | Integração | Correlação por `requestId` via `X-Request-Id` na API, somada ao `X-Event-Id` | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-DEP-01 | `docs/FDD.md` | Restrição | Node 20, Prisma 5.22.0, Express 4.21.1, Zod 3.23.8, Pino 9.5.0, jsonwebtoken 9.0.2, uuid 11.0.3 | CODIGO | `package.json` |
| FDD-DEP-02 | `docs/FDD.md` | Restrição | MySQL 8.0 | CODIGO | `docker-compose.yml` |
| FDD-ACEITE-01 | `docs/FDD.md` | Critério de Aceite | Critérios cobrem atomicidade, filtro, latência, headers, HMAC, timeout, retry, DLQ, replay ADMIN, https, rotação, 64KB e deliveries | TRANSCRICAO | [09:06] Diego; [09:34] Bruno; [09:44] Diego; [09:17] Larissa; [09:36] Sofia; [09:23] Sofia; [09:21] Sofia |
| FDD-ACEITE-02 | `docs/FDD.md` | Critério de Aceite | Testes com Vitest e Supertest contra MySQL real, no padrão atual | CODIGO | `tests/orders.test.ts` |
| FDD-ACEITE-03 | `docs/FDD.md` | Restrição | Revisão de segurança de ao menos dois dias úteis antes do deploy (ver RFC-IMP-04) | TRANSCRICAO | [09:46] Sofia; [09:47] Larissa |
| FDD-RISCO-01 | `docs/FDD.md` | Risco | Riscos consolidados do RFC (ver RFC-RISK-01 a RFC-RISK-06 e RFC-RISK-08) | TRANSCRICAO | [09:40] Bruno; [09:07] Bruno; [09:24] Diego; [09:17] Diego; [09:42] Diego; [09:22] Diego; [09:45] Marcos |
| FDD-INT-01 | `docs/FDD.md` | Integração | `changeStatus` chama `publishWebhookEvent` com o mesmo `tx`, sem injetar repository no `OrderService` | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-INT-02 | `docs/FDD.md` | Integração | Mapa de transições permanece inalterado e delimita os pares de status publicáveis | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-INT-03 | `docs/FDD.md` | Integração | Novos modelos de outbox, dead letter e configuração com id UUID `Char(36)` e `@@map` em snake_case | CODIGO | `prisma/schema.prisma` |
| FDD-INT-04 | `docs/FDD.md` | Integração | `buildControllers()` recebe a montagem do módulo de webhooks | CODIGO | `src/app.ts` |
| FDD-INT-05 | `docs/FDD.md` | Integração | `buildApiRouter` monta `/webhooks` e `/admin/webhooks/dead-letter/:id/replay` | CODIGO | `src/routes/index.ts` |
| FDD-INT-06 | `docs/FDD.md` | Integração | `src/server.ts` é o modelo para `src/worker.ts` | CODIGO | `src/server.ts` |
| FDD-INT-07 | `docs/FDD.md` | Integração | `createPrismaClient()` permite a instância própria de `PrismaClient` do worker | CODIGO | `src/config/database.ts` |
| FDD-INT-08 | `docs/FDD.md` | Integração | Importar `env.ts` no worker torna `DATABASE_URL` e `JWT_SECRET` obrigatórios para ele | CODIGO | `src/config/env.ts` |
| FDD-INT-09 | `docs/FDD.md` | Integração | Novo script `worker` | CODIGO | `package.json` |
| FDD-INT-10 | `docs/FDD.md` | Integração | Novas tabelas precisam entrar na limpeza do `beforeEach` dos testes | CODIGO | `tests/setup.ts` |
