### ADR-002: Ler a outbox com worker em polling a cada 2 segundos

Status: aceito 
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição) 
Supersedes: nenhum 
Superseded by: nenhum 
Related to: ADR-001, ADR-003 
Amends: nenhum 

---

### 1. Contexto
Com a outbox decidida, faltava definir como o worker lê a tabela ([09:08] Larissa). Para os clientes, qualquer entrega abaixo de 10 segundos já é "tempo real" ([09:02] Marcos), e o MySQL não tem listener nativo como o NOTIFY/LISTEN do Postgres ([09:09] Diego). Como o worker deve descobrir novos eventos na outbox?

---

### 2. Decisão
Opção escolhida: "Polling em loop a cada 2 segundos, buscando os eventos pendentes mais antigos", porque atende com folga o requisito de ficar abaixo de 10 segundos ([09:09] Diego), sem o improviso que seria necessário para avisar o worker a partir do banco ([09:09] Diego). A decisão foi registrada em [09:10] Larissa e confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### Trigger do banco

- **Descrição:** usar trigger do banco para tornar a leitura mais reativa ([09:09] Bruno).
- **Prós:** seria mais reativo que o polling ([09:09] Bruno).
- **Contras:** a trigger só executa SQL e não notifica processo externo; avisar o worker exigiria improvisar algo como escrever em arquivo ou bater em um endpoint ([09:09] Diego).
- **Motivo do descarte:** "fica esquisito", e o polling de 2 segundos já atende ao requisito ([09:09] Diego).

---

### 4. Consequências

**Positivas**
- O intervalo de 2 segundos atende ao requisito de negócio, validado por Marcos ([09:10] Marcos).
- O worker lê só os pendentes, em batch pequeno ([09:08] Diego).

**Negativas**
- A latência de 2 segundos passa a ser inerente à solução e foi aceita, nas palavras da reunião: "A latência mínima vai ser 2 segundos no pior caso" ([09:10] Larissa).

**Trade-off**
- Aceita-se a latência do intervalo de polling ([09:10] Larissa) em troca de não improvisar um mecanismo de notificação a partir do banco ([09:09] Diego).

---

### 5. Referências
- Transcrição: [09:02] Marcos; [09:08] Diego; [09:08] Larissa; [09:09] Bruno; [09:09] Diego; [09:10] Marcos; [09:10] Larissa; [09:48] Larissa
- Código: `prisma/schema.prisma` (`provider = "mysql"`, banco sobre o qual a limitação de listener foi discutida)
- RFC, FDD ou documentação externa: nenhum referenciado.
