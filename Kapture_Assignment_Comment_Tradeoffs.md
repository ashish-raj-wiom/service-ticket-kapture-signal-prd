# Kapture Assignment Comment — tradeoff log

Running log from the interview of 18 Sep 2026. Becomes the tradeoffs register at finalise.
Every row is a decision Ashish made against options that were put to him.

| # | Decision point | Chosen | Rejected options | Why (PM's stated reason) | Date |
|---|---|---|---|---|---|
| 1 | What Kapture is told, given nobody is assigned during the 131-minute blind window | **Assignment only** — push when the CSP takes the job or assigns a technician | Assignment plus "not picked up yet"; technician assignments only; every CSP-side state change | Took the feature as scoped, knowing it lands at the end of the blind window rather than inside it. The unclaimed-ticket signal is left as a separate problem (§1 Boundary). | 18 Sep 2026 |
| 2 | How the agent sees it | **A comment on the Kapture ticket** | Comment plus a sub-status; structured fields; comment now with fields as V2 | Reuses the existing `ADD_COMMENT` push. Leaves status, sub-status, priority and queue untouched, so no ops dashboard or TAT bucket moves. | 18 Sep 2026 |
| 3 | What the comment says | **Name and role of the assignee** | Name, role and contact number; role only; name, role and timestamp | Enough for the agent to answer "who is coming?". A technician's personal number in a CRM many agents read was not wanted. | 18 Sep 2026 |
| 4 | Whether later changes also comment | **Every assignment — swaps and recalls too** | First assignment only; first assignment with swaps as V2 | The thread has to be a true record; a stale name is worse than no name. Costs a new event out of TAS and a new endpoint on SRS, which the first-assignment case would not have. | 18 Sep 2026 |
| 5 | Shifting tickets | **Included** | Excluded to match the chat PRD; included with its own wording | The chat PRD excluded shifting because the customer copy was complaint-framed. An internal agent comment carries no such wording, so the reason did not transfer — and shifting has the worse blind window. | 18 Sep 2026 |
| 6 | The push to Kapture fails | **Best effort — one attempt, dropped, nothing recorded** | Retry until it lands; retry and raise on exhaustion; retry and record against the ticket | An agent-facing comment does not warrant retry and recovery engineering. **Recorded as an Override**: a dropped comment is invisible, and AC-GRD-3 cannot tell one from an action that never happened. | 18 Sep 2026 |
| 7 | One action arriving twice, and the same person assigned again | **Same rule as the chat PRD** — one comment per action; a genuine re-assign of the same person still comments | Suppress by person; no suppression at all | Keeps the two specs consistent, so an engineer reading both finds one rule rather than two. | 18 Sep 2026 |

## Measurements the decisions were made against

Taken 18 Sep 2026 from the TAS execution-candidate data in the warehouse, 30-day window.

| Figure | Value |
|---|---|
| Restore candidates | 49,065 |
| Median blind window — reaching the CSP → first assign action | **131 min** |
| Blind for over 1 hour / over 3 hours | **64.8%** / **43.1%** |
| Resolved within 5 min of the first assign action | **73.3%** |
| Gap over 30 min between first assign action and resolution | **21.9%** — roughly 10,100 tickets a month |
| When a technician is assigned: median to resolution | **35 min**, with **51.1%** over 30 min |
| Technician swaps | 4.17% of assigned jobs — 819 in 30 days |
| Shifting jobs assigned a technician | 94%, a median 4.4 hours in, against a 96-hour TAT |

**What the measurement changed.** The brief said Kapture is blind "between the job landing with the CSP and it being resolved". The data says something sharper: nobody is assigned during the blind window at all. The job sits untouched for a median 131 minutes, then the CSP responds and closes it — 73% within five minutes. So this feature fires at the *end* of the blind period and is genuinely useful for the ~22% with a real gap, not for the callback taken 40 minutes in. That is recorded in §1 Boundary rather than left for someone to discover.

## Code facts the PRD rests on

| Fact | Where |
|---|---|
| **Correction to the brief:** SRS *does* receive a first-response signal. `ES_RESTORE_FIRST_RESPONSE_RECORDED` fires on the first of `ACCEPT_TASK` or `ASSIGN_TECHNICIAN` and stamps `first_response_at` | `csp-support-resolution-service/…/api/InboundEventController.java:175` |
| But it fires **once per job** and carries `triggering_action` with **no person** — so it cannot serve the comment's name, nor any later action | `…/domain/event/inbound/EsRestoreFirstResponseRecorded.java` |
| Only `EsRestoreTechnicianAssigned` carries `ticketId`. The accepted and recalled events carry none | `csp-tas-service/…/restore/domain/event/outbound/*.java` |
| Both assign events have exactly one consumer today — csp-notification-service, which turns them into CleverTap events | `csp-notification-service/…/translate/EventTranslator.java` |
| `ADD_COMMENT` already exists and takes `comment`, `ticket_id` and `sub_status`; an empty `sub_status` leaves the ticket's sub-status untouched | `ticket-service-java/…/service/handler/impl/AddCommentHandler.java` |
| ticket-service has no concept of a restore technician; `ASSIGN_TICKET` is CC→Partner queue routing, not this | `ticket-service-java/…/util/KaptureQueueMessageKey.java` |
