### ADR-005: Persistir falhas permanentes na tabela separada webhook_dead_letter

Status: aceito 
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição) 
Supersedes: nenhum 
Superseded by: nenhum 
Related to: ADR-001, ADR-004 
Amends: nenhum 

---

### 1. Contexto
Ao atingir o teto de tentativas, o evento é considerado falha permanente e movido para a DLQ ([09:15] Diego). Esses eventos precisam ficar disponíveis para debug e reprocessamento ([09:18] Diego). A DLQ deve ser uma tabela separada ou uma marcação "failed" na própria outbox ([09:17] Larissa)?

---

### 2. Decisão
Opção escolhida: "Tabela separada `webhook_dead_letter`, com payload, motivo da falha e timestamp", porque deixa mais limpa a leitura da outbox principal e serve de evidência para debug e reprocessamento ([09:18] Diego). A decisão foi confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### Marcar como "failed" na própria outbox

- **Descrição:** manter o evento na outbox, marcado como "failed" ([09:17] Larissa).
- **Prós:** nenhum discutido na reunião.
- **Contras:** a leitura da outbox principal fica menos limpa ([09:18] Diego).
- **Motivo do descarte:** a tabela separada foi preferida pelos motivos da decisão ([09:18] Diego; [09:18] Bruno).

---

### 4. Consequências

**Positivas**
- A leitura da outbox principal fica mais limpa ([09:18] Diego).
- Payload, motivo da falha e timestamp ficam guardados como evidência ([09:18] Diego).

**Negativas**
- O reprocessamento é manual: um endpoint admin recoloca o evento na outbox como pendente ([09:18] Diego).
- Ponto não fechado na reunião: a outbox foi descrita com o status "falhou" ([09:08] Diego), e o papel desse status ao lado da DLQ não foi definido.

**Trade-off**
- Aceita-se manter uma tabela adicional e um reprocessamento manual ([09:18] Diego) em troca de uma outbox mais limpa e de evidência dedicada para debug ([09:18] Diego).

---

### 5. Referências
- Transcrição: [09:08] Diego; [09:15] Diego; [09:17] Larissa; [09:18] Diego; [09:18] Bruno; [09:19] Larissa; [09:48] Larissa
- Código: `src/middlewares/auth.middleware.ts` (`requireRole`, que a reunião decidiu reaproveitar no endpoint de replay, [09:36] Larissa)
- RFC, FDD ou documentação externa: nenhum referenciado.
