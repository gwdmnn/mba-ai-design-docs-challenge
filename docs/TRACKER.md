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
| RFC-PROP-04 | `docs/RFC.md` | Decisão | Worker em processo separado com polling de 2 segundos (consolida ADR-002 e ADR-003) | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:11] Larissa; [09:30] Bruno |
| RFC-PROP-05 | `docs/RFC.md` | Decisão | Retry 1m/5m/30m/2h/12h e DLQ em `webhook_dead_letter` com replay manual (consolida ADR-004 e ADR-005) | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:18] Diego |
| RFC-PROP-06 | `docs/RFC.md` | Decisão | HMAC-SHA256 com secret por endpoint e at-least-once com `X-Event-Id` (consolida ADR-006 e ADR-007) | TRANSCRICAO | [09:22] Sofia; [09:25] Diego; [09:26] Larissa |
| RFC-PROP-07 | `docs/RFC.md` | Decisão | Módulo `src/modules/webhooks` reaproveitando `AppError`, Pino e middleware de erro | TRANSCRICAO | [09:27] Bruno; [09:30] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa Descartada | Disparo síncrono no service de orders (ver ADR-001-ALT-01) | TRANSCRICAO | [09:03] Larissa; [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa Descartada | Redis Streams ou similar (ver ADR-001-ALT-02) | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa Descartada | Trigger do banco para notificar o worker (ver ADR-002-ALT-01) | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa Descartada | Garantia exactly-once (ver ADR-007-ALT-01) | TRANSCRICAO | [09:25] Sofia; [09:25] Diego |
| RFC-OPEN-01 | `docs/RFC.md` | Ponto em Aberto | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:38] Diego; [09:39] Diego; [09:39] Larissa; [09:48] Larissa |
| RFC-OPEN-02 | `docs/RFC.md` | Ponto em Aberto | Escala para múltiplos workers e perda de ordenação, "problema do futuro" | TRANSCRICAO | [09:12] Diego; [09:13] Diego; [09:13] Larissa |
| RFC-OPEN-03 | `docs/RFC.md` | Ponto em Aberto | Se as 5 tentativas incluem o envio inicial (ver LAC-01) | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| RFC-OPEN-04 | `docs/RFC.md` | Ponto em Aberto | Papel do status "falhou" da outbox ao lado da DLQ (ver LAC-02) | TRANSCRICAO | [09:08] Diego; [09:18] Diego |
| RFC-OPEN-05 | `docs/RFC.md` | Ponto em Aberto | Armazenamento da secret e qual secret assina no grace period (ver LAC-03) | TRANSCRICAO | [09:21] Sofia; [09:46] Sofia |
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
