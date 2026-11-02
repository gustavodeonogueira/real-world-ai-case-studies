# Case 01 — Lessons Learned / Lições Aprendidas

[English case](../README.md) · [Caso em português](../README.pt-BR.md) · [Architecture / Arquitetura](../architecture/README.md)

## English

### Applied changes and verified corrections

- Derive duration in the workflow from service/date. Recorded API checks confirmed Saturday clinical duration of 60 minutes and vaccination duration of 30 minutes despite a conflicting incoming duration.
- Enforce one patient per request/event. Grouped-patient input was rejected in a recorded API check.
- Enforce the business window. A recorded out-of-window request was rejected. That is targeted evidence, not proof of every opening-hour, holiday or timezone boundary.
- Add a PostgreSQL booking guard and update agent instructions/skill descriptions. These were applied changes; full concurrency behavior through WhatsApp was not demonstrated.
- Preserve functional paths during edits. Compare node parameters and connections rather than editor-generated identifiers or visual layout.

### Lessons from the work

1. **Conversation is not the source of booking authority.** Prompts help collect and explain information; workflow validation must enforce scheduling constraints.
2. **Availability does not equal reservation.** Recheck before committing; do not guarantee a slot merely because it was offered.
3. **A failed lookup does not mean an empty calendar.** Separate a free result from an unavailable integration.
4. **An uncertain write is not a definite failure.** Reconcile the event outcome before repeating the operation; blind retries risk duplicate bookings.
5. **Human fallback must be explicit.** Unsupported rescheduling and unresolved outcomes require a person. A configured transfer instruction still needs a real conversation test.
6. **Saved configuration is not end-to-end success.** API checks can prove specific rule outcomes without proving customer-facing behavior.
7. **Existing appointments need separate review.** Do not silently rewrite previously created events when correcting future scheduling rules.

### Planned regression checks — not completed results

Use synthetic patients, a dedicated test calendar and authorized test conversations.

- [ ] Complete availability → user confirmation → event creation → response through WhatsApp.
- [ ] Verify weekday clinical, Saturday clinical and vaccination duration in the actual event start/end.
- [ ] Test opening, closing, appointment end, timezone and holiday boundaries.
- [ ] Reject a partially occupied candidate; refresh stale availability.
- [ ] Process two pets sequentially as separate events.
- [ ] Run simultaneous confirmations for one slot; inspect the guard and calendar outcome.
- [ ] Simulate lookup failure, known creation failure and lost response after a write.
- [ ] Reconcile uncertain creation without creating a duplicate or prematurely releasing the reservation.
- [ ] Verify handoff reaches the human queue and that automated replies stop as intended.
- [ ] Evaluate a dedicated rescheduling flow before enabling it.
- [ ] Clean up only test events after inspection.

### Planned improvements

Repeatable integration tests, structured error categories, reconciliation monitoring and duplicate-operation checks would strengthen reliability. They are proposed engineering work, not delivered features in this repository. No uptime, conversion, cost-saving or latency figures are claimed.

## Português

### Mudanças aplicadas e correções verificadas

- Calcular duração no fluxo a partir de serviço/data. Consultas registradas à API confirmaram 60 minutos para consulta clínica de sábado e 30 para vacinação, mesmo com duração conflitante na entrada.
- Exigir um paciente por solicitação/evento. Entrada com pacientes agrupados foi rejeitada em uma consulta registrada à API.
- Validar a janela comercial. Uma requisição fora da janela foi rejeitada. É evidência pontual, não prova de todos os limites de expediente, feriados ou fuso.
- Adicionar proteção de reserva em PostgreSQL e atualizar instruções/descrições das habilidades. As mudanças foram aplicadas; o comportamento completo de concorrência pelo WhatsApp não foi demonstrado.
- Preservar caminhos funcionais durante ajustes. Comparar parâmetros e conexões dos nós, não identificadores gerados pelo editor ou posições visuais.

### Lições do trabalho

1. **A conversa não é autoridade para reservar.** Prompts ajudam na coleta e explicação; as restrições precisam ser impostas pelo fluxo.
2. **Disponibilidade não é reserva.** Revalidar antes de gravar; não garantir um horário apenas porque foi oferecido.
3. **Falha de consulta não significa agenda vazia.** Separar resultado livre de integração indisponível.
4. **Gravação incerta não é falha definitiva.** Reconciliar o evento antes de repetir; tentativas cegas podem duplicar reservas.
5. **A alternativa humana precisa ser explícita.** Remarcações não suportadas e resultados não esclarecidos exigem uma pessoa. Instrução de transferência configurada ainda precisa de teste real.
6. **Configuração salva não comprova sucesso ponta a ponta.** Testes de API validam regras específicas sem comprovar toda a experiência do usuário.
7. **Agendamentos existentes exigem revisão separada.** Corrigir regras futuras não autoriza reescrever silenciosamente eventos anteriores.

### Validações de regressão planejadas — não são resultados concluídos

Usar pacientes sintéticos, agenda dedicada a testes e conversas de teste autorizadas.

- [ ] Completar disponibilidade → confirmação do usuário → criação do evento → resposta pelo WhatsApp.
- [ ] Conferir duração de consulta clínica em dia útil, sábado e vacinação no início/fim do evento.
- [ ] Testar abertura, fechamento, término do atendimento, fuso e feriados.
- [ ] Recusar horário parcialmente ocupado e atualizar disponibilidade antiga.
- [ ] Processar dois pets sequencialmente em eventos distintos.
- [ ] Executar confirmações simultâneas para um horário e inspecionar reserva e agenda.
- [ ] Simular falha de consulta, falha definitiva de criação e perda da resposta após gravação.
- [ ] Reconciliar criação incerta sem duplicar evento nem liberar a reserva antes da hora.
- [ ] Verificar chegada à fila humana e interrupção das respostas automáticas.
- [ ] Avaliar fluxo dedicado de remarcação antes de habilitá-lo.
- [ ] Remover somente eventos de teste após a conferência.

### Melhorias planejadas

Testes repetíveis de integração, categorias estruturadas de erro, monitoramento da reconciliação e checagem de operações duplicadas podem fortalecer a confiabilidade. São propostas de engenharia, não funcionalidades entregues por este repositório. Não são alegados números de disponibilidade, conversão, economia ou latência.
