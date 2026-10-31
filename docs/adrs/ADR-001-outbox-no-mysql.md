### ADR-001: Registrar eventos de webhook com outbox no MySQL, na mesma transação da mudança de status

Status: aceito
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição)
Supersedes: nenhum
Superseded by: nenhum
Related to: ADR-002, ADR-003, ADR-007, ADR-008
Amends: nenhum

---

### 1. Contexto
Três clientes B2B pediram notificação em tempo real quando o status dos pedidos muda ([09:00] Marcos). A transação de mudança de status já é pesada, e uma chamada HTTP dentro dela faria um cliente lento travar mudanças de status de outros pedidos ([09:04] Bruno). Como registrar o evento de webhook sem acoplar a mudança de status à disponibilidade do cliente e sem perder eventos?

---

### 2. Decisão
Opção escolhida: "Padrão outbox no MySQL existente, com inserção na `webhook_outbox` dentro da transação de `changeStatus`", porque, se a transação principal commitou, o evento foi registrado, e, se ela deu rollback, o evento some junto ([09:06] Diego), sem subir infraestrutura nova para um time pequeno ([09:07] Diego). A decisão foi fechada em [09:08] Larissa e confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### Disparo síncrono no service de orders

- **Descrição:** fazer a chamada HTTP ao cliente dentro do service de orders, no momento da mudança de status ([09:03] Larissa).
- **Prós:** nenhum discutido na reunião.
- **Contras:** um cliente lento travaria mudanças de status de outros pedidos ([09:04] Bruno), e não há como dar rollback na mudança de status quando o cliente está fora do ar ([09:04] Bruno).
- **Motivo do descarte:** "Síncrono está fora de questão" ([09:06] Diego).

### Redis Streams ou similar

- **Descrição:** usar Redis Streams ou algo parecido no lugar de uma tabela no MySQL ([09:07] Larissa).
- **Prós:** nenhum discutido na reunião.
- **Contras:** exigiria subir mais infraestrutura ([09:07] Larissa).
- **Motivo do descarte:** para um time pequeno, subir Redis Cluster para isso é overengineering ([09:07] Diego).

---

### 4. Consequências

**Positivas**
- Status e evento têm o mesmo destino na transação: commit registra o evento e rollback o remove ([09:06] Diego).
- As chamadas HTTP passam a ser disparadas por um worker separado que lê a tabela ([09:06] Diego).
- O MySQL existente é suficiente, sem nova infraestrutura ([09:07] Diego).

**Negativas**
- `changeStatus` passa a depender da outbox: se a inserção falhar, a mudança de status sofre rollback ([09:40] Bruno).
- A tabela pode acumular eventos ([09:07] Bruno). A resposta combinada são índices em status e `created_at`, leitura só de pendentes em batch pequeno e arquivamento posterior das linhas entregues, este fora do escopo da feature ([09:08] Diego).

**Trade-off**
- Aceita-se acoplar a mudança de status à inserção na outbox e deixar o arquivamento da tabela para depois em troca de nunca haver status alterado sem evento registrado ([09:40] Bruno; [09:41] Diego) e de não adicionar infraestrutura ([09:07] Diego).

---

### 5. Referências
- Transcrição: [09:03] Larissa; [09:04] Bruno; [09:06] Diego; [09:07] Larissa; [09:07] Diego; [09:07] Bruno; [09:08] Diego; [09:08] Larissa; [09:40] Bruno; [09:41] Diego; [09:48] Larissa
- Código: `src/modules/orders/order.service.ts` (método `changeStatus`, que executa a mudança de status em `prisma.$transaction`, onde a inserção na outbox será feita)
- Código: `src/modules/orders/order.status.ts` (`shouldDebitStock` e `shouldReplenishStock`, que definem quando a transação movimenta estoque)
- Código: `prisma/schema.prisma` (modelo `OrderStatusHistory`, gravado na mesma transação)
- RFC, FDD ou documentação externa: nenhum referenciado.
