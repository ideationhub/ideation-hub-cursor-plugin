# Implementation handoff (Phase F)

The handoff brief lets a separate implementation agent build the design without this conversation. It is generated from Ideation Hub records and is never itself the authority.

## Preconditions

1. Run Phase E validation and share the result.
2. Check the review state of every design record in scope:
   - **All accepted:** build the brief from accepted records only, the same rule Ideation Hub uses for exports and briefs.
   - **Some pending:** build the brief anyway if the user asks, and put this banner at the top: "DRAFT — includes design proposals not yet accepted in Ideation Hub: <IDs>." List the pending IDs.
3. This phase prepares a brief for another agent. It does not write application code. Implementing is separate, and only when the user asks (`implementation.md`, and the recording rules in `SKILL.md`, Implementation).

## Brief contents

Use `assets/handoff-brief.schema.json` for a machine-readable brief, or this Markdown shape for a human-readable one. Include only what the scope needs.

```markdown
# Implementation brief — <product>, <scope>
Derived from Ideation Hub on <date> (product <id>). Not authoritative: re-read the records before changing anything.

## Requirements in scope
| ID@rev | Title | Kind (functional / NFR / principle) |

## Architecture summary
<style, building blocks, deployment — from the context SD>

## Components
For each CD/SD: ID@rev, title, responsibility, behaviour rules, interacts with, data, errors and limits, security rules, which PRDs it delivers.

## Interfaces
For each interface CD: operations table, rules, provider and consumers.

## Data schemas
For each data CD: entities, keys, relationships, invariants, retention.

## Decisions
For each decision SD: decision, reason, consequences the implementer must respect.

## Technical constraints
Stated constraints only, each with its source.

## Dependencies
External systems, libraries or services chosen (build vs buy), with versions only when the design fixes them.

## Acceptance criteria
The linked verification tests (VER/UT) for each PRD and component, by ID and title. Where a PRD has no linked test, say so; do not invent acceptance criteria.

## Outstanding questions and assumptions
Everything still tagged `arch.open-question` or `arch.assumption`, and how the implementer should treat it.

## Traceability expectations for the implementer
- Record implementations as IMPL records linked `CD realized_by IMPL` (or `SD realized_by IMPL`), with `repoPath`, `prUrl` and `commitSha`, through the requirements skill. When this skill is asked to implement, this agent does that recording itself.
- Cite the PRD and CD IDs in pull requests.
```

## Record `design_ready`

After producing the brief, **always** call `ih_record_development_phase_event` once per in-scope Product Requirement:

```json
{"requirement": "PRD-012", "phase": "design_ready", "evidenceUrl": "<review or record URL, if known>"}
```

- Recording `design_ready` is the only direct write in this phase. It is append-only and idempotent for the same arguments. Confirmation reviews and status assessments are separate direct writes, described in `assessment.md`. They need `status:write` and never change a title, body, or trace link.
- It needs the `delivery:write` permission. On 403, tell the user plainly that the credential cannot record development-phase events, so "design ready" was not recorded, and continue. Do not try any other way.
- Report which PRDs got the event and which failed.

## Report to the user

- The brief (or where it was saved, labelled as a derived view).
- `design_ready` results per PRD.
- The pending proposals, if any, that still need acceptance.
- The open questions that the implementer will hit first.
