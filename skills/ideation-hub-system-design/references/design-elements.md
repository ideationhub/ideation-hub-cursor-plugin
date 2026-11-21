# Design elements: how a design is stored in Ideation Hub

Ideation Hub has three design record types: **System Design (SD)**, **Component Design (CD)** and **UX Design (UXD)**. Each has a title, a Markdown statement, tags, revisions and trace links. A UXD also has a `mockupUrl` field. There are no dedicated types for decisions, interfaces or data models, and no link type for "depends on". This skill stores every design artifact in those three types using the conventions below. Do not ask for new types or links; they are listed as future enhancements in the package README.

## Mapping

| Artifact | Record | Tag | Links (source → target) |
| --- | --- | --- | --- |
| System context and boundary, architectural style | SD | `arch.context` | `PRD realized_by SD` for each in-scope PRD, including NFRs |
| Subsystem / container (a separately deployed or run unit) | SD | `arch.container` | `SD(context) refined_by SD(container)`, and `PRD realized_by SD(container)` for the PRDs it delivers |
| Architecture decision | SD | `arch.decision` | `PRD realized_by SD(decision)` for its drivers, and `SD(decision) refined_by SD/CD` for each element it shapes |
| Component | CD | `arch.component` | `SD realized_by CD`, and `CD refined_by CD` for sub-components |
| Interface or API contract | CD | `arch.interface` | `CD(owner) refined_by CD(interface)`, or `SD realized_by CD(interface)` when it belongs to a container |
| Data model | CD | `arch.data` | `SD or CD refined_by CD(data)` |
| Screens and flows | UXD | `arch.ux` | `PRD realized_by UXD`; `mockupUrl` when a mockup exists |
| Product-level constraint, NFR or principle | Stays a PRD tagged `NFR` or `principle`. Never authored by this skill | `NFR` | `PRD(NFR) realized_by` each SD/CD that responds to it |
| Technical constraint found during design | "Constraints" section in the affected SD; its own SD only if it cuts across the system | `arch.constraint` | as the record it sits in |
| Safety constraint | Follow the requirements skill's `references/stpa.md` | `STPA` | per STPA |
| Assumption | "Assumptions" section in the affected record, each marked *provisional* or *confirmed* | `arch.assumption` while any is provisional | — |
| Open question | "Open questions" section in the affected record | `arch.open-question` while any is open | — |
| Interaction between components | "Interacts with" section in the CD, naming the other record by title, with direction and style (sync/async) | — | none (no link type exists) |
| Validation finding | Not stored. Reported in the conversation or a generated report. A finding that needs a fix becomes a proposal on the affected record | — | — |
| Design verification test | VER or UT, authored through the requirements skill | — | see `traceability.md` |
| Diagram, Markdown document, handoff brief | Generated view, labelled "Derived from Ideation Hub on <date>; not authoritative" | — | — |

Tag rules: 1–32 characters from letters, numbers, spaces and `_-./` (no colon), at most 20 per record. Add the `arch.*` tags alongside any tags the user already uses. Pass the full tag list on an edit, because `tags` replaces the list; omit `tags` to leave them unchanged.

## Granularity

Do not split a design into more records than it needs.

- A component with no separate interface or data worth reviewing on its own stays a bullet in its parent SD or CD.
- An interface or data model gets its own CD only when another component or an implementation agent needs to read it on its own, or when it changes on a different schedule.
- A decision gets its own SD only when it is hard to reverse or contested. Small decisions are a line in the "Key decisions" section of the context SD.

## Titles

- SD and CD titles name the thing: "Order service", "Checkout API", "Order data model", "System context: ShopTrack".
- Decision titles state the choice: "Use PostgreSQL for order data", "Run as one deployable web app".
- No requirement IDs, no type names ("SD:"), no profile names in titles.
- Run the requirements skill's editor pass on titles and bodies before filing.

## Body templates

Write bodies in plain Markdown. Leave out a section that does not apply. Do not put requirement IDs in bodies; the trace links carry the justification. Refer to other design records by title.

### Context SD (`arch.context`)

```markdown
**Purpose.** <one or two sentences on what the system is for>

**Boundary.** Inside: <…>. Outside: <actors and external systems>.

**Architectural style.** <style> because <reasons tied to the requirements>. Not chosen: <alternative> because <reason>.

**Building blocks.** <list of containers or components, one responsibility each>

**Key decisions.** <small decisions, one line each; larger ones are decision records>

**Quality responses.** <for each driving quality: the design response>

**Deployment and operations.** <where it runs, how it is released and observed>

**Constraints.** <stated constraints and their source>

**Assumptions.** <each marked provisional or confirmed>

**Risks.** <risk, likelihood, impact, mitigation>

**Open questions.** <question, why it matters, current provisional answer>
```

### Container SD (`arch.container`)

```markdown
**Responsibility.** <one sentence>

**Technology.** <runtime, framework, store> — <build or buy reason>

**Provides.** <interfaces it exposes, by title>

**Uses.** <other containers or external systems, sync or async>

**Data owned.** <data models it owns, by title>

**Scaling and reliability.** <only against stated or confirmed targets>

**Assumptions / Open questions.** <as above>
```

### Decision SD (`arch.decision`) — MADR-lite

Structure adapted from MADR (https://adr.github.io/madr/, MIT or CC0-1.0).

```markdown
**Status.** Proposed | Accepted (follows the record's review state) | Supersedes "<old decision title>"

**Context.** <the problem and forces, in two to four sentences>

**Decision drivers.** <requirements and qualities that matter, by name>

**Options considered.**
1. <option> — <pros> / <cons>
2. <option> — <pros> / <cons>

**Decision.** <chosen option> because <reasons tied to the drivers>.

**Consequences.** Good: <…>. Bad: <…>. Follow-up: <…>.

**Confirmation.** <how we will know the decision holds: a test, a metric, a review>
```

### Component CD (`arch.component`)

```markdown
**Responsibility.** <one sentence; one reason to change>

**Behaviour.** <the main behaviours, as short numbered rules>

**Interacts with.** <record title> — <calls / is called by / publishes to / reads> — <sync or async> — <why>

**Data.** <data it owns or reads, by title>

**Errors and limits.** <failure handling, timeouts, idempotency>

**Security.** <authn/authz rules it enforces>

**Assumptions / Open questions.** <as above>
```

### Interface CD (`arch.interface`)

```markdown
**Provider.** <component title>   **Consumers.** <titles>

**Style.** <REST/JSON, GraphQL, gRPC, events, function call> — <why>

**Operations.**
| Operation | Input | Output | Errors | Notes |
| --- | --- | --- | --- | --- |

**Rules.** <auth, idempotency, pagination, rate limits, versioning>
```

Keep the contract precise enough for an implementation agent, but do not write code. A short schema fragment (JSON Schema or a table) is fine.

### Data model CD (`arch.data`)

```markdown
**Owner.** <component or container title>   **Store.** <database or storage>

**Entities.**
| Entity | Key fields | Relationships | Notes |
| --- | --- | --- | --- |

**Rules.** <invariants, uniqueness, retention, privacy, migration approach>
```

### UX Design (`arch.ux`)

Use only when the scope includes user-facing flows and the user wants them designed here. State the screens or flows, the states each handles, and the PRDs' observable behaviour they deliver. Put the mockup link in `mockupUrl`.

## Filing order

File parents before children so links can name them. In one `ih_propose_bulk_push`, give each new item a `ref` and use the refs in `links`. See `write-protocol.md`.
