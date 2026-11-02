# Real-World AI Case Studies

Practical, anonymized case studies by **Gustavo Nogueira**, based on AI automation and integration work at **MauveHub**.

[Português](README.pt-BR.md) · [Architecture / Arquitetura](architecture/README.md) · [Lessons learned / Lições aprendidas](lessons-learned/README.md)

## Case 01 — AI-Powered Appointment Scheduling

### Context and objective

Maia is a conversational agent used in a veterinary service workflow. The operational challenge was to connect a WhatsApp/CRM conversation to calendar availability and appointment creation while respecting service rules and avoiding unsupported confirmations.

The goal was to support appointment scheduling with explicit workflow validation and a human fallback. This case describes my practical work configuring agent instructions and skills, adjusting n8n workflows, and investigating integration behavior. It is a documentation case study, not a standalone application or a release of the production system.

### Architecture

**WhatsApp/CRM → LLM agent → n8n/Webhooks → Google Calendar**

The LLM collects intent and missing information. n8n applies deterministic rules and calls the calendar integration. Google Calendar supplies availability and event results. A PostgreSQL booking guard was added to serialize confirmations and preserve uncertain writes for reconciliation.

See the [bilingual architecture explanation and rendered Mermaid diagram](architecture/README.md) and [diagram source](architecture/appointment-scheduling.mmd). The diagram is a simplified reconstruction, not an export of proprietary workflow nodes.

### Availability and confirmation flow

1. Read conversation context and identify the requested service and one patient.
2. Collect only missing scheduling information. General clinical consultation and vaccination are separate service categories; ambiguous or clinical questions require clarification or human assistance, not medical inference.
3. Send a structured availability request through the agent's integration skill to n8n.
4. Validate service, date, local time, duration and business window, then query Google Calendar. A failed query must not be treated as an empty calendar.
5. Offer slots returned by the integration. Offering a slot does not reserve it.
6. After the user selects and explicitly confirms, submit a separate confirmation request.
7. Revalidate the slot, serialize confirmation using the booking guard, and create the calendar event.
8. Confirm to the user only when the integration provides a known successful outcome. An uncertain write requires reconciliation rather than a blind retry.

Steps describe the intended integrated behavior; the validation limits below apply.

### Service and business-hour rules

The implemented duration correction derived appointment length from service and date in n8n rather than trusting a duration supplied by the agent:

| Service | Monday–Friday | Saturday |
| --- | --- | --- |
| General clinical consultation | 30 minutes | 60 minutes |
| Vaccination | 30 minutes | 30 minutes |

One request/event represents one pet. Multiple pets require separate sequential requests and refreshed availability.

Business windows are checked in the workflow. A rejected out-of-window request is a rule outcome, not an available appointment. This public case omits the client's opening hours, calendar identifiers and exceptions.

**Fictional example only:** a demo clinic opens Monday–Friday 09:00–17:00 and Saturday 09:00–13:00 in a configured local timezone. A 60-minute visit at 12:30 on Saturday extends beyond closing and should be rejected. These opening hours are not the production schedule. Holiday coverage and boundary scenarios still need explicit regression tests.

### Problems, changes and evidence

| Problem or risk | Technical response | Evidence boundary |
| --- | --- | --- |
| Fixed duration made Saturday clinical appointments too short | Calculate duration from service/date in n8n | Applied; API validation returned 60 minutes for Saturday clinical and 30 for vaccination |
| Requests outside a valid scheduling window | Validate the business window before booking | A recorded out-of-window API request was rejected; comprehensive boundary coverage is not established |
| Several pets grouped into one appointment | Enforce one pet per request and event | Applied; grouped-patient API request was rejected |
| Calendar availability or confirmation failure | Distinguish query failure, rejection and uncertain write; do not invent success | Error-handling design documented; complete failure-path coverage is not established |
| Competing confirmations or a lost response after creation | PostgreSQL booking guard; retain uncertain reservation and reconcile event outcome | Guard applied; no complete WhatsApp concurrency test is evidenced |
| Requests requiring a person, including unsupported rescheduling | Route to human service instead of promising an unsupported action | Agent instruction/routing policy; complete handoff behavior still requires end-to-end validation |

### Implementation status

- **Implemented/configured:** service-dependent duration, one pet per request, booking guard, and updated agent instructions/skill descriptions.
- **Corrections verified through recorded API checks:** Saturday duration, vaccination duration, out-of-window rejection and rejection of grouped pets.
- **Not established by those checks:** a complete WhatsApp → event creation → customer confirmation journey, simultaneous real conversations, or every API failure path.
- **Planned validation/improvements:** end-to-end regression tests, failure injection, reconciliation monitoring, duplicate-operation checks, and a dedicated rescheduling flow.
- **Outside this case:** automatic changes to existing appointments and external practice-management/CSV integration. A later cancellation configuration exists in the historical work, but a successful end-to-end cancellation was not demonstrated; it is not presented here as verified.

This is a retrospective based on implementation records, not a fresh audit of the live system. No quantitative performance, cost or business impact claims are made.

### Technical decisions and lessons

Keep the LLM responsible for conversation and the workflow responsible for booking rules. Separate availability from confirmation, preserve working integrations during changes, and treat an uncertain write differently from a definite failure. Compare functional workflow parameters and connections when validating edits; editor-generated identifiers and layout are not behavioral evidence.

Read the [lessons and validation checklist](lessons-learned/README.md).

### Privacy and scope

The client is anonymized. No credentials, private URLs, phone numbers, guardian/customer records, production identifiers, proprietary code or workflow exports are included. Examples use fictional patient labels such as **Demo Pet A** and synthetic inputs. This repository documents engineering decisions; it does not provide medical advice or claim to be an official MauveHub publication.
