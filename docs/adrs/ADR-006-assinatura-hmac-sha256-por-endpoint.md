### ADR-006: Assinar webhooks com HMAC-SHA256 e secret rotacionável por endpoint

Status: aceito
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição)
Supersedes: nenhum
Superseded by: nenhum
Related to: nenhum
Amends: nenhum

---

### 1. Contexto
Os eventos expõem dados de pedidos para endpoints fora da infraestrutura da plataforma, e o cliente precisa validar que a requisição veio da plataforma e que o payload não foi adulterado ([09:19] Sofia). Além disso, já houve cliente que vazou secret em log da própria aplicação ([09:22] Diego). Como autenticar as entregas e limitar o impacto de um vazamento?

---

### 2. Decisão
Opção escolhida: "HMAC-SHA256 sobre o corpo do request, enviado em `X-Signature`, com secret única por endpoint e rotação com grace period de 24 horas", porque é o padrão de mercado, com biblioteca disponível para todo cliente sério ([09:20] Sofia), e uma secret por endpoint impede que um vazamento comprometa todos ([09:21] Sofia). A decisão foi fechada em [09:22] Sofia e confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### Secret global da plataforma

- **Descrição:** usar uma única secret da plataforma para todos os endpoints ([09:21] Sofia).
- **Prós:** nenhum discutido na reunião.
- **Contras:** "se vaza uma, vaza tudo" ([09:21] Sofia).
- **Motivo do descarte:** o risco de vazamento total ([09:21] Sofia).

---

### 4. Consequências

**Positivas**
- O cliente consegue verificar do lado dele a origem e a integridade da requisição ([09:19] Sofia; [09:20] Sofia).
- O vazamento de uma secret fica restrito a um endpoint ([09:21] Sofia).
- O cliente pode pedir nova secret pela API e tem 24 horas para migrar os sistemas ([09:21] Sofia).

**Negativas**
- A configuração de webhook passa a armazenar a secret junto de url, `customer_id` e estado ativo ([09:21] Bruno; [09:21] Sofia).
- Durante 24 horas, a secret antiga continua válida em paralelo com a nova ([09:21] Sofia).
- HMAC e geração de secret exigem revisão de segurança de ao menos dois dias úteis antes do deploy ([09:46] Sofia).
- Pontos não fechados na reunião: a forma de armazenar a secret e qual secret assina os envios durante o grace period não foram definidas.

**Trade-off**
- Aceita-se manter duas secrets válidas por 24 horas ([09:21] Sofia) em troca de o cliente ter tempo de migrar os sistemas dele ([09:21] Sofia).

---

### 5. Referências
- Transcrição: [09:19] Sofia; [09:20] Bruno; [09:20] Sofia; [09:21] Sofia; [09:21] Bruno; [09:22] Diego; [09:22] Sofia; [09:44] Diego; [09:46] Sofia; [09:48] Larissa
- Código: nenhum arquivo existente referenciado.
- RFC, FDD ou documentação externa: nenhum referenciado.
