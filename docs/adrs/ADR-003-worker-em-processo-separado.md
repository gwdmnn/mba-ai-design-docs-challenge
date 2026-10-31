### ADR-003: Executar o worker de webhooks em processo separado da API

Status: aceito
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição)
Supersedes: nenhum
Superseded by: nenhum
Related to: ADR-001, ADR-002
Amends: nenhum

---

### 1. Contexto
O worker precisa ler a outbox continuamente, e hoje o projeto tem como entry-point `src/server.ts` ([09:11] Larissa). Se o worker rodar na mesma instância da API, um reinício da API o derruba ([09:11] Diego). Onde o worker deve rodar?

---

### 2. Decisão
Opção escolhida: "Processo Node separado, com entry-point `src/worker.ts` e script `npm run worker`, usando o mesmo banco e a mesma stack", porque, se a API reiniciar, o worker não é perdido ([09:11] Diego). O processo usa instância própria de `PrismaClient` com a mesma `DATABASE_URL`, já que `PrismaClient` é por processo ([09:30] Bruno). A decisão foi confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### Worker dentro da instância da API

- **Descrição:** executar o worker no mesmo processo da API ([09:11] Diego).
- **Prós:** nenhum discutido na reunião.
- **Contras:** se a API reinicia, perde o worker ([09:11] Diego).
- **Motivo do descarte:** "Só não pode ser o mesmo processo" ([09:11] Diego).

---

### 4. Consequências

**Positivas**
- Reinícios da API não derrubam o worker ([09:11] Diego).
- A stack e o banco continuam os mesmos ([09:11] Diego).

**Negativas**
- O projeto ganha um novo entry-point e um novo script ([09:11] Larissa).
- O worker abre sua própria instância de `PrismaClient` ([09:30] Bruno).

**Trade-off**
- Aceita-se manter um processo adicional, com instância própria de `PrismaClient` ([09:30] Bruno), em troca de o worker não depender do ciclo de vida da API ([09:11] Diego).

---

### 5. Referências
- Transcrição: [09:11] Diego; [09:11] Larissa; [09:11] Bruno; [09:28] Bruno; [09:28] Diego; [09:29] Diego; [09:30] Bruno; [09:48] Larissa
- Código: `src/server.ts` (entry-point atual da API, citado como modelo para `src/worker.ts`)
- Código: `src/config/database.ts` (criação do `PrismaClient` usado pela API)
- Código: `package.json` (scripts do projeto, onde `worker` será adicionado)
- RFC, FDD ou documentação externa: nenhum referenciado.
