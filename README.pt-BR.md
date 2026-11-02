# Estudos de Caso de IA em Operação

Estudos de caso práticos e anônimos de **Gustavo Nogueira**, baseados no trabalho com automações e integrações de IA na **MauveHub**.

[English](README.md) · [Arquitetura / Architecture](architecture/README.md) · [Lições aprendidas / Lessons learned](lessons-learned/README.md)

## Case 01 — AI-Powered Appointment Scheduling

### Contexto e objetivo

Maia é uma agente conversacional utilizada em um fluxo de atendimento veterinário. O desafio operacional era conectar conversas de WhatsApp/CRM à consulta de disponibilidade e à criação de eventos, respeitando as regras do serviço e evitando confirmações sem respaldo da integração.

O objetivo era apoiar o agendamento com validações explícitas no fluxo e alternativa de atendimento humano. O caso descreve meu trabalho prático de configuração de instruções e habilidades da agente, ajustes em fluxos n8n e investigação de comportamentos das integrações. É um estudo documental, não uma aplicação independente nem uma distribuição do sistema de produção.

### Arquitetura

**WhatsApp/CRM → Agente LLM → n8n/Webhooks → Google Calendar**

O LLM coleta a intenção e os dados faltantes. O n8n aplica regras determinísticas e chama a integração da agenda. O Google Calendar fornece disponibilidade e resultados dos eventos. Uma proteção de reserva em PostgreSQL foi adicionada para serializar confirmações e preservar gravações incertas para reconciliação.

Consulte a [explicação bilíngue e o diagrama Mermaid renderizado](architecture/README.md) e o [arquivo do diagrama](architecture/appointment-scheduling.mmd). O desenho é uma reconstrução simplificada, sem exportação dos nós proprietários.

### Fluxo de disponibilidade e confirmação

1. Ler o histórico e identificar o serviço solicitado e um paciente.
2. Coletar apenas as informações faltantes. Consulta clínica geral e vacinação são categorias distintas; ambiguidades e questões clínicas exigem esclarecimento ou atendimento humano, sem inferência médica.
3. Enviar uma solicitação estruturada de disponibilidade pela habilidade de integração da agente ao n8n.
4. Validar serviço, data, horário local, duração e janela comercial; então consultar o Google Calendar. Falha na consulta não significa agenda vazia.
5. Oferecer os horários retornados pela integração. Oferecer um horário não o reserva.
6. Após escolha e confirmação explícita do usuário, enviar uma solicitação separada de confirmação.
7. Revalidar a disponibilidade, serializar a confirmação com a proteção de reserva e criar o evento.
8. Confirmar ao usuário somente com resultado de sucesso conhecido. Uma gravação incerta exige reconciliação, sem repetição cega.

As etapas descrevem o comportamento integrado esperado; os limites de validação abaixo se aplicam.

### Classificação e regras de horário

A correção de duração implementada passou a calcular o tempo conforme serviço e data no n8n, sem confiar na duração enviada pela agente:

| Serviço | Segunda a sexta | Sábado |
| --- | --- | --- |
| Consulta clínica geral | 30 minutos | 60 minutos |
| Vacinação | 30 minutos | 30 minutos |

Uma solicitação/evento representa um pet. Vários pets exigem solicitações separadas e sequenciais, com nova consulta de disponibilidade.

O fluxo verifica a janela comercial. Solicitação fora da janela é uma rejeição de regra, não um horário disponível. Este caso público omite os horários de abertura, identificadores de agenda e exceções do cliente.

**Exemplo exclusivamente fictício:** uma clínica de demonstração abre de segunda a sexta das 09:00 às 17:00 e sábado das 09:00 às 13:00, em um fuso local configurado. Uma consulta de 60 minutos às 12:30 do sábado ultrapassa o fechamento e deve ser recusada. Esses horários não são os de produção. Feriados e cenários de limite ainda exigem testes de regressão explícitos.

### Problemas, soluções e evidências

| Problema ou risco | Resposta técnica | Limite da evidência |
| --- | --- | --- |
| Duração fixa encurtava consultas clínicas de sábado | Calcular duração por serviço/data no n8n | Aplicado; validação de API retornou 60 minutos para consulta clínica de sábado e 30 para vacinação |
| Agendamentos fora da janela válida | Validar janela comercial antes da reserva | Uma requisição fora de horário foi rejeitada pela API; cobertura completa dos limites não foi demonstrada |
| Vários pets agrupados em um agendamento | Exigir um pet por solicitação e evento | Aplicado; requisição com pacientes agrupados foi rejeitada |
| Falha na consulta ou confirmação da agenda | Diferenciar falha de consulta, rejeição e gravação incerta; não inventar sucesso | Tratamento de erros documentado; cobertura de todas as falhas não foi demonstrada |
| Confirmações concorrentes ou perda de resposta após criação | Proteção em PostgreSQL; manter reserva incerta e reconciliar o resultado | Proteção aplicada; sem teste completo de concorrência pelo WhatsApp registrado |
| Necessidade de atendimento humano, inclusive remarcação não suportada | Encaminhar para a equipe em vez de prometer ação não integrada | Política/instrução da agente; transferência completa ainda requer validação ponta a ponta |

### Estado de implementação

- **Implementado/configurado:** duração por serviço, um pet por solicitação, proteção de reserva e instruções/descrições de habilidades atualizadas.
- **Correções verificadas em consultas registradas à API:** duração de sábado, duração de vacinação, rejeição fora de horário e de pets agrupados.
- **Não comprovado por esses testes:** jornada completa WhatsApp → criação do evento → confirmação ao usuário, conversas reais simultâneas ou todos os erros de API.
- **Validações/melhorias planejadas:** regressão ponta a ponta, simulação de falhas, monitoramento de reconciliação, checagem de operações duplicadas e fluxo dedicado de remarcação.
- **Fora do escopo:** alterar automaticamente agendamentos existentes e integrar sistema de gestão/CSV. Existe configuração posterior de cancelamento no histórico, mas não foi demonstrado cancelamento bem-sucedido ponta a ponta; ele não é apresentado como validado.

Este é um relato retrospectivo baseado em registros de implementação, não uma auditoria atual do sistema. Não são alegados resultados quantitativos de desempenho, custos ou impacto comercial.

### Decisões técnicas e lições

A conversa fica com o LLM e as regras de reserva ficam no fluxo. Separar consulta de confirmação, preservar integrações funcionais durante ajustes e distinguir gravação incerta de falha definitiva são decisões centrais. Ao validar alterações, comparar parâmetros e conexões funcionais; identificadores gerados pelo editor e posições visuais não comprovam comportamento.

Consulte as [lições e a lista de validação](lessons-learned/README.md).

### Privacidade e escopo

O cliente é anônimo. Não há credenciais, URLs privadas, telefones, dados de tutores/clientes, identificadores de produção, código proprietário ou exportações dos fluxos. Os exemplos usam rótulos fictícios, como **Pet Demo A**, e entradas sintéticas. O repositório documenta decisões de engenharia; não oferece orientação médica nem se apresenta como publicação oficial da MauveHub.
