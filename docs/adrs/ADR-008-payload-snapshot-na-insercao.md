### ADR-008: Guardar na outbox o payload renderizado no momento da inserção

Status: aceito
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição)
Supersedes: nenhum
Superseded by: nenhum
Related to: ADR-001
Amends: nenhum

---

### 1. Contexto
Entre a inserção na outbox e o envio, o pedido pode mudar ([09:52] Larissa). O tema surgiu depois do resumo final da reunião e da saída de Marcos e Sofia, entre Larissa, Diego e Bruno ([09:50] Larissa; [09:51] Bruno). O evento da outbox deve guardar o payload já renderizado ou só o `order_id`, renderizando na hora do envio ([09:51] Bruno)?

---

### 2. Decisão
Opção escolhida: "Payload renderizado na inserção (snapshot)", porque o evento continua refletindo o estado de quando o status mudou, mesmo que o pedido mude depois ([09:52] Larissa). A decisão foi fechada em [09:52] Bruno, com a concordância de [09:52] Diego.

---

### 3. Alternativas consideradas

### Guardar só o order_id e renderizar no envio

- **Descrição:** a outbox guarda apenas o `order_id`, e o payload é montado na hora do envio ([09:51] Bruno).
- **Prós:** nenhum discutido na reunião.
- **Contras:** se o pedido mudar depois, o evento deixa de refletir o estado da mudança de status, o que Larissa chamou de "caso esquisito" ([09:52] Larissa).
- **Motivo do descarte:** o mesmo contra acima ([09:52] Larissa).

---

### 4. Consequências

**Positivas**
- O evento preserva o estado de quando o status mudou ([09:52] Larissa).

**Negativas**
- Nenhuma foi discutida na reunião.

**Trade-off**
- Nenhum custo foi explicitado na reunião; o ganho registrado é a fidelidade do evento ao momento da mudança de status ([09:52] Larissa).

---

### 5. Referências
- Transcrição: [09:50] Larissa; [09:51] Bruno; [09:52] Larissa; [09:52] Diego; [09:52] Bruno
- Código: `src/modules/orders/order.service.ts` (`changeStatus`, ponto em que a mudança de status ocorre)
- RFC, FDD ou documentação externa: nenhum referenciado.
