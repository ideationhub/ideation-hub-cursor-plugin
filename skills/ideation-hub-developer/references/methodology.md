# Methodology: phases A–F

Use this file to run the design loop. Each phase lists the inputs, the steps, what to show the user, and when to stop. The phases are a loop, not a pipeline: go back to any phase when the user challenges, adds or changes something.

Before Phase B, run the user-need trace in `implementation.md`. Do not draft a design for a Product Requirement that trace flagged until the user resolves it.

Implementation and assessment are separate from this design loop. When the user asks you to implement, follow `implementation.md` (it starts with the same user-need trace). When the user gives you Product Requirements to assess, follow `assessment.md`. Do not add assessment rules here.

How to read a requirement, a link, a neighbor, or a status word is in the platform skill (`../../ideation-hub-platform/references/schema.md` and `mcp.md`). Which slice to retrieve is the requirements skill's `references/retrieval.md`. This file does not restate them.

## Labels used everywhere

Tag every statement you make about the design with one of these labels, in your messages and in record bodies:

- **Fact:** stated by a record you read in this conversation. Cite it as `PRD-012@3`.
- **Assumption:** something you take as true to make progress. Mark it *provisional* until the user confirms it.
- **Recommendation:** your engineering judgement on structure, scope, or risk. Give the reason. It is not a license to pick a language, framework, platform, cloud, or vendor; those come from applicable Principles or from the human.
- **Open question:** something you cannot decide without the user or more information.

Never present an assumption or recommendation as a fact.

## Phase A — Understand the problem

**Inputs:** the scope the user agreed (objective, tag, PRD section or named IDs).

**Steps:**

1. Start from the user-need summary in `implementation.md` (who, what, and why, with IDs). Then retrieve the rest of the scope with the smallest read that answers the question, using the retrieval reference above:
   - Search the objective or tag with pending rows excluded.
   - Walk down from each in-scope PRD for existing designs, implementations, and tests.
   - Read the whole product chain only when the scope is the whole product, and exclude obsolete rows.
   - Do a separate read that includes pending proposals. Label them "proposal" and never treat them as accepted.
2. Find the non-functional requirements and principles. An NFR is a Product Requirement tagged `NFR`. Search by tag (`tag: "NFR"`). A Principle is a `PRN` record. Read the ones that apply with `ih_list_applicable_principles`. This skill does not author a Principle. Also look for `STPA` records; if any are in scope, follow the requirements skill's `references/stpa.md`.
3. Read existing SD, CD and UXD records in scope (`types: ["SD","CD","UXD"]`). An existing design is the starting point, not something to replace.
4. Build a short **problem frame**:
   - Purpose: one or two sentences from the User and Business Needs.
   - Users and actors: from the profiles.
   - Functional scope: the in-scope PRDs, grouped by capability.
   - Quality drivers: the NFRs, and any quality the PRDs imply. Label implied ones as assumptions.
   - Constraints: those stated in records, including applicable Principles, or stated by the user in this conversation.
   - External dependencies: systems the requirements name.
   - Missing information: what you need but no record states. If the missing item is a language, framework, platform, cloud, or vendor, and no applicable Principle covers it, list it as an open question. Do not assume a stack.
5. Use the status vocabulary in the retrieval reference. Do not invent a second set of status words.

**Show the user:** the problem frame, in under a screen, with IDs.

**Stop and ask** only if the scope is unclear or the requirements contradict each other in a way that changes the whole design. Otherwise carry on to Phase B with provisional assumptions.

## Phase B — Generate a first-pass design

**Steps:**

1. **Choose the proportionality rung.** Start at the lowest rung that covers the scope and move up only for a cited reason.

   | Rung | When | Records |
   | --- | --- | --- |
   | 1. Small | One team, one deployable, modest load, few integrations | 1 context SD holding style, key decisions and a component list inline. 0–3 CDs, only for components with real interfaces or data |
   | 2. Typical | Several capability areas, a few integrations, real NFRs | 1 context SD, 1–4 container SDs, decision SDs only for hard-to-reverse choices, CDs for components with interfaces or data |
   | 3. Complex | Multiple teams or deployables, demanding NFRs, distribution forced by requirements | Rung 2, plus interface and data CDs and a quality-attribute scenario for each driving NFR |

2. **Cover these topics where the scope needs them.** Skip a topic with one line ("No integrations in scope") instead of inventing content.
   - System context and boundary: what is inside, the actors, the external systems.
   - Architectural style and rationale (see `architecture-patterns.md`).
   - Major subsystems and components, with one responsibility each.
   - Interactions: who calls whom, synchronous or asynchronous, and why.
   - Interfaces and contracts: operations, inputs, outputs, errors, versioning.
   - Data: entities, ownership, persistence, retention.
   - Security and authorization: identities, roles, trust boundaries, secrets.
   - Integration: external systems, failure handling.
   - Scalability, reliability and performance: only against stated or confirmed targets.
   - Deployment and operations: where it runs, how it is observed, how it is released.
   - Significant technical risks.
3. **Build vs buy.** For each significant capability (auth, payments, search, messaging, storage, email, analytics), say whether to build or buy only as applicable Principles require. If none apply, ask the human. Do not name a vendor, product, or library from this skill, and do not prefer buy or build as a default.
4. **Justify every element.** Each element lists the requirement IDs (and decisions) it serves. An element with no justification is removed or flagged as a recommendation for the user to accept.
5. **Draft records, do not file them yet.** Write the record titles, bodies, tags and links following `design-elements.md`. Keep them in the conversation (or in a scratch design plan matching `assets/design-plan.schema.json`).

## Phase C — Collaborate with the user

1. **Present the first pass in sections:** context and style first, then components and interfaces, then data, then cross-cutting concerns, then risks. For each section, say what it is based on (IDs) and what is assumed.
2. **Pick the questions that matter.** Ask a question only if its answer changes the structure, a hard-to-reverse choice, or an NFR response. Ask at most 3–5 at once, most important first. Each question:
   - says why it matters (what changes depending on the answer)
   - offers two or three options
   - gives your recommendation and the provisional assumption you will use if the user does not answer, except when the question is a language, framework, platform, cloud, or vendor: then cite any applicable Principle, or ask the human, and do not offer a skill-level recommendation or a provisional technology assumption of your own
3. **Use trade-off analysis only where a trade-off is real** (`tradeoff-analysis.md`).
4. **Keep moving.** If the user answers some questions and not others, carry the provisional assumptions forward and say so, except you must not carry a technology assumption this skill invented.
5. **When the user agrees**, stage the records with a dry run and file them as proposals (`write-protocol.md`).

Avoid: questionnaires, asking what the requirements already say, and asking about details that do not change the design. Ask the user rather than recommending a language, framework, platform, cloud, or vendor when no applicable Principle covers the choice.

## Phase D — Iterate and refine

The user may challenge a decision, ask for alternatives, change a constraint, add or change requirements, ask for more depth, revisit an earlier decision, or ask for a review.

1. **Locate the impact.**
   - For a changed requirement: `ih_get_neighbors` `direction: "down"` from it, and compare each link's `targetRevision`/`sourceRevision` with the record's current revision (`traceability.md`).
   - For a challenged decision: the decision SD's outbound `refined_by` links list the elements it shaped.
2. **Change only what is affected.** Leave unaffected records alone. List what you are keeping unchanged.
3. **State downstream effects:** other components, interfaces, data, NFR responses, tests and implementations that the change touches.
4. **Revisiting a decision:**
   - Draft a new decision SD that records the new choice, with the old decision's title in its Context and a "Supersedes" line.
   - Propose `obsolete` on the old decision through `ih_lifecycle_action` (`mode: "suggest"`), after the user confirms.
   - Move the old decision's `refined_by` links to the new decision with `addLinks` and `removeLinks`.
5. **More depth:** add CDs under the relevant SD or CD with `refined_by`. Do not rewrite the parent.
6. **A requirement change requested by the user:** hand it to the requirements skill's workflow. The design change follows once that requirement is accepted, or proceeds against the pending proposal labelled as such.
7. **A design review request:** run Phase E.

## Phase E — Validate

Follow `validation.md`. Run the deterministic checks from Ideation Hub data first, then the qualitative review. Label every finding verified, predicted or judgement. Present the findings, and offer fixes as proposals.

## Phase F — Implementation handoff

Follow `implementation-handoff.md`. Build the brief and record `design_ready` for each in-scope Product Requirement, then report. This ends the design loop. When the user asked you to implement, continue with `implementation.md`. When the user asked you to assess Product Requirements, follow `assessment.md`.

## Proportionality reminders

- One deployable is the default. Distribution needs a cited reason: independent scaling, independent release, isolation or team boundaries that the requirements or the user state.
- No queue, cache, CDN, search engine, service mesh or container orchestration without a requirement that needs it.
- Small products deserve small designs. A one-record design for a small app is a good outcome.
