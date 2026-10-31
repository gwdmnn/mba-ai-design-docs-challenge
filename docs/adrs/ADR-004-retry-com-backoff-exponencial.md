### ADR-004: Retentar entregas com backoff exponencial e teto de 5 tentativas

Status: aceito
Data: quinta-feira, 09:00 (data de calendário não informada na transcrição)
Supersedes: nenhum
Superseded by: nenhum
Related to: ADR-005, ADR-007
Amends: nenhum

---

### 1. Contexto
O cliente pode estar offline no momento da entrega ([09:14] Larissa), e já houve cliente com indisponibilidade de duas horas em manutenção planejada ([09:16] Diego). Retentar pouco descartaria eventos cedo demais, e retentar sem limite deixaria eventos pendurados para sempre ([09:15] Diego; [09:16] Diego). Quantas vezes e em que intervalos a entrega deve ser retentada?

---

### 2. Decisão
Opção escolhida: "5 tentativas com backoff exponencial de 1m, 5m, 30m, 2h e 12h, e depois falha permanente com envio para a DLQ", porque cobre quase 15 horas entre a primeira falha e a última tentativa ([09:17] Diego), janela considerada aceitável ([09:17] Marcos), sem deixar evento pendurado indefinidamente ([09:15] Diego). A decisão foi fechada em [09:17] Larissa e confirmada em [09:48] Larissa.

---

### 3. Alternativas consideradas

### 3 tentativas

- **Descrição:** reduzir o teto para 3 tentativas, mais agressivo ([09:16] Bruno).
- **Prós:** é mais agressivo ([09:16] Bruno).
- **Contras:** um cliente indisponível de manhã teria três retentativas em 30 minutos e o evento seria encerrado ([09:16] Diego).
- **Motivo do descarte:** "3 é pouco" ([09:16] Diego); a reunião fechou em cinco ([09:16] Larissa).

### Retry indefinido com backoff

- **Descrição:** retentar sem teto, aumentando o intervalo ([09:15] Diego).
- **Prós:** nenhum discutido na reunião.
- **Contras:** o evento pode ficar pendurado para sempre se o cliente sumiu ([09:15] Diego).
- **Motivo do descarte:** o problema do evento pendurado; cinco tentativas já cobrem a janela necessária ([09:15] Diego).

---

### 4. Consequências

**Positivas**
- Indisponibilidades como a manutenção planejada de duas horas ficam cobertas pela janela de retentativas ([09:16] Diego; [09:17] Diego).
- Todo evento tem um fim definido: entrega ou falha permanente ([09:15] Diego).

**Negativas**
- Um cliente fora do ar além da janela de quase 15 horas tem o evento movido para a DLQ, situação aceita pelo produto ([09:17] Marcos).
- Ponto não fechado na reunião: não foi dito se as 5 tentativas incluem o envio inicial, embora a progressão tenha 5 intervalos ([09:17] Diego).

**Trade-off**
- Aceita-se levar à falha permanente os eventos de clientes indisponíveis por mais de quase 15 horas ([09:17] Marcos) em troca de evitar eventos pendurados para sempre ([09:15] Diego).

---

### 5. Referências
- Transcrição: [09:14] Larissa; [09:15] Diego; [09:16] Bruno; [09:16] Diego; [09:16] Larissa; [09:17] Diego; [09:17] Marcos; [09:17] Larissa; [09:48] Larissa
- Código: nenhum arquivo existente referenciado.
- RFC, FDD ou documentação externa: nenhum referenciado.
