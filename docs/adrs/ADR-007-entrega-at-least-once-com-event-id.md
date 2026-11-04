### ADR-007: Garantir entrega at-least-once com deduplicação por X-Event-Id no cliente

Status: aceito 
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição) 
Supersedes: nenhum 
Superseded by: nenhum 
Related to: ADR-001, ADR-004 
Amends: nenhum 

---

### 1. Contexto
O cliente pode receber o mesmo evento duas vezes e precisa conseguir diferenciar duplicatas ([09:24] Diego; [09:25] Bruno). Exigir exactly-once demandaria coordenação dos dois lados ([09:25] Diego). Qual garantia de entrega a plataforma deve oferecer e como o cliente trata duplicatas?

---

### 2. Decisão
Opção escolhida: "At-least-once com `X-Event-Id`, um UUID único por evento gerado na entrada da outbox, para deduplicação do lado do cliente", porque é o padrão de mercado, usado por Stripe e GitHub segundo Diego, e resolve 99% dos casos sem a complexidade de exactly-once ([09:25] Diego). A decisão foi fechada em [09:26] Larissa e confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### Exactly-once

- **Descrição:** garantir que cada evento seja recebido exatamente uma vez ([09:25] Diego).
- **Prós:** nenhum discutido na reunião.
- **Contras:** exigiria coordenação dos dois lados e fica muito mais complexo ([09:25] Diego).
- **Motivo do descarte:** a complexidade, diante de uma alternativa que resolve 99% dos casos ([09:25] Diego).

---

### 4. Consequências

**Positivas**
- O cliente tem um identificador único por evento para deduplicar ([09:25] Diego).
- A plataforma não precisa de coordenação com o cliente para a entrega ([09:25] Diego).

**Negativas**
- O cliente pode receber o mesmo evento duas vezes e tem que estar preparado ([09:24] Diego).
- A responsabilidade pela deduplicação passa para o cliente ([09:25] Sofia), o que será documentado com destaque no portal de desenvolvedor ([09:26] Marcos).

**Trade-off**
- Aceita-se transferir a deduplicação para o cliente ([09:25] Sofia) em troca de evitar a complexidade de exactly-once ([09:25] Diego).

---

### 5. Referências
- Transcrição: [09:24] Diego; [09:25] Bruno; [09:25] Diego; [09:25] Sofia; [09:26] Marcos; [09:26] Larissa; [09:44] Diego; [09:48] Larissa
- Código: nenhum arquivo existente referenciado.
- RFC, FDD ou documentação externa: nenhum referenciado. Stripe e GitHub foram citados por Diego sem link ([09:25] Diego).
