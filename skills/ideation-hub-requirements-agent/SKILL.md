---
name: ideation-hub-requirements-agent
description: Requirements skill of the Ideation Hub plugin. Use when asked to write, revise, trace, or structure Business Needs, User Profiles, User Needs, and Product Requirements, or to retrieve requirements for an objective. Proposes changes for human review. Uses the Ideation Hub platform skill for the schema, REST API, and MCP tools.
metadata:
  version: "1.7.2"
---

# Ideation Hub — Requirements

**Product:** `https://www.ideationhub.me` · **API:** `https://api.ideationhub.me` · **MCP:** `https://api.ideationhub.me/api/mcp`  
**Audience:** Any coding agent working on a customer's Product in Ideation Hub. Tenant-agnostic: this file holds no secret, tenant id, or product id.

This skill writes and structures requirements. Record types, trace links, revisions, statuses, tags, the REST API, and the MCP tool catalog live in the **Ideation Hub platform skill**. Do not restate them.

From this file, with `ideation-hub-platform` as a sibling skill folder:

- `../ideation-hub-platform/references/schema.md`
- `../ideation-hub-platform/references/mcp.md`
- `../ideation-hub-platform/references/rest-api.md`
- `../ideation-hub-platform/references/credentials.md` — credential setup

All three skills ship together in the one Ideation Hub plugin, so the platform skill is always installed beside this one. If a relative path is missing because only this folder was copied, open the `ideation-hub-platform` skill and read the same file.

## Schema and traces

Read `schema.md` in the platform skill before you create or link a record. It defines the types, the link types, stored direction, allowed pairs, revision pins, status words, and durable IDs. A Product Requirement never links directly to a User Profile. Flag a record with no upward trace. Do not treat that gap as a defect by itself.

**Profile capability rule (authoring):** a proposed Profile that is really one capability (for example "issues scoped agent credentials") is not a Profile. Propose that capability as a Product Requirement that traces to a User Need of an existing Profile, not to the Profile itself. A one-to-one trace needs an explicit stated reason.


## Reference files (load only when the task needs them)

- `references/retrieval.md` — retrieve requirements for an objective: search, vertical slice, horizontal view, or full export; token budget. ID, link, and status words are in the platform schema.
- `references/document-structure.md` — structure a document: point first, non-overlapping sections in task order; derive and order PRD sections; write section summaries.
- `references/rest-api.md` — pointer to the platform skill's REST reference.
- `references/stpa.md` — STPA safety and hazard analysis.
- `references/changelog.md` — version history.

## Working with the user (conversational workflow)

1. **Have a conversation with the end user or parse a document from them.**
2. **Be interactive and ask for disambiguation of anything you hear.** Pair open questions with suggested interpretations ("Did you mean A or B, or something else?"); never guess.
3. **Disseminate all information from the conversation with the end user** — break everything the user said into candidate requirement elements (BN, Profile, UN, PRD, acceptance criteria, designs, tests), losing nothing.
4. **Retrieve what already exists for the objective.** Start with the smallest read that answers it: `ih_search` for one thing, `ih_get_neighbors` for one area, `ih_get_chain` only for a whole-product view. See `references/retrieval.md`.
5. **Have a dialogue with the user about what currently exists in the requirements** — matches, overlaps, gaps.
6. **Determine how best to update those requirements or add to them, keeping the design control structure in place:** TraceLinks up to UNs/business needs, the PRD rubric (testable product behavior, no implementation), and propose-only via Review (§4) unless direct edit is explicitly granted. Apply the layered pattern (see Writing rules): Profile = the person, UN = the WHAT, PRD = the HOW as testable product behavior.
7. **Recommend, don't just transcribe.** If the user is misguided (e.g. needs in a Profile, a HOW in a User Need), still recommend how to define the requirements clearly: explain the better structure and offer a rewritten version.

## Upward Traceability & Discovery Guidelines (Required When Authoring Requirements)

1. **Trace each PRD up to every User Need and Business Need it satisfies (`PRD --addresses--> UN` or `BN`).** Allowed pairs are in the platform schema. Link it via `parentId` on create and/or `ih_link_requirements` (`sourceId: "<PRD-ID>", type: "addresses", targetId: "<UN-ID>"`). Tool arguments are in the platform `mcp.md`.
2. **Search existing User Needs whenever defining a new requirement.** Before creating or updating a requirement, search all existing User Needs in the Product (`ih_search`, or `ih_get_chain` for the whole set) and consider which existing User Needs that new requirement satisfies.
3. **Suggest candidate new User Needs or Business Objectives and ask before creating them.** Also consider which *new* User Needs or Business Objectives (Business Needs) the new requirement would satisfy that are not yet in the Product. Suggest those to the user as candidate proposals and ask whether they are reasonable before creating them.

## 1. Credentials, auth, and tools

Relay credential setup from the platform skill's `references/credentials.md`. Do not invent a second setup flow, and do not ask for a secret in chat.

Authentication, the suggest-versus-edit write rule, every MCP tool, and trace-link review outcomes are in the platform skill (`mcp.md` and `rest-api.md`). Pass `mode: "suggest"` when a proposal should wait for a person even if the credential may edit directly. Never call a direct-only tool (`ih_create_requirement`, `ih_update_requirement`, `ih_set_tags`) unless the user explicitly asked for a direct write and the credential has edit on that product.

## 4. Propose a change (write)

Follow the platform write rule. Then use this authoring workflow.

Workflow:

1. **Read neighbors first.** Pull the chain (`ih_get_chain`) or neighbors (`ih_get_neighbors`) for the parent you will attach to.
2. **Check duplicates, conflicts, overlap, and related requirements before writing.** Search existing requirements and pending proposals by meaning as well as title and statement, including their profiles, scope, status, and traces. Compare against same-type siblings under the intended parent and same-type records product-wide, and state the one-line scope boundary before proposing (see Scope boundary and overlap). For every proposed new or updated requirement, evaluate other related requirements and all appropriate upward trace targets; link only where the proposed capability genuinely satisfies the need, not merely because wording is similar. Preserve distinctions such as tenant-wide versus product-specific authority.
   - **Extend before you add.** If an existing Product Requirement, accepted or pending, already covers the same subject, amend it to include the new capability rather than creating another. Create a new Product Requirement only when the capability has its own rules or lifecycle, such as its own review, staleness or permissions, that would blur the existing requirement's scope. When two existing requirements overlap, propose folding one into the other and marking the absorbed one obsolete.
   - **Clarify ambiguous definitions or structure before any write.** If duplicate, conflicting, overlapping, or related requirements leave ambiguity about which record to create or update, whether to combine or separate capabilities, or which traces are appropriate, explain the relevant records and recommend a structure, then ask the user to resolve that ambiguity. Do not silently merge, broaden scope, withdraw, or invent relationships. Clear, non-conflicting matches do not require another confirmation. A second `add` of an existing key conflicts.
3. **Consider traces whenever modifying a requirement.** Before any edit, withdrawal, obsolete, reclassify, or link change, read the requirement's inbound and outbound traces (`ih_get_requirement`, `ih_get_neighbors`). Check whether each existing trace still holds after the change, whether any should be added, removed, or redirected, and whether linked requirements need their own updates. Where you are not certain a trace or related change is right, ask the user for guidance rather than guessing. After a trim, split, or merge, re-check that no obligation is unowned or double-owned and that traces still hold.
4. **Explain planned related changes and ask before acting on recommendations.** Before writing, tell the user the main change and every related change you plan (trace additions or removals, edits to linked requirements, withdrawals), with a one-line rationale for each, and ask any open questions. Proceed only with what the user confirms; changes the user explicitly asked for need no second confirmation, but related changes you recommend do.
5. **Editor pass before proposing (required).** Clear, readable requirements are a primary goal. Reread each title, then each body, as a careful human editor would for a reader new to the product: Is the subject a real thing in the product? Is the verb accurate? Could any word or phrase be read two ways? Is any sentence too long, or does it bury its point behind a list? Would the reader stumble or need context? Rewrite until it reads naturally and cannot be misread; titles get the most care.
6. **Explain-back check before proposing.** Launch a fresh subagent and give it only the requirement's title and body, with no other context. The subagent explains in plain words what the Product must do and for whom. Judge whether that matches what you meant. Check the title and the body for clarity, for making sense on their own, and for consistency with each other. Rewrite anything the reader misread or found ambiguous (titles most of all), then repeat with a new fresh reader, up to 3 rounds. If you cannot launch a subagent, do the editor pass in step 5 instead.
7. **Amend a pending proposal instead of filing another.** Before proposing a change to a requirement, read it and check `pendingProposal`. If one exists, read its `payload` and amend it with `ih_propose_edit` plus `proposalId`. Do not edit the requirement directly or file a second proposal. On **409** `pending_proposal_exists`, amend that `proposalId`. Anyone with propose permission can amend any pending proposal. After accept, reject, or withdraw, a new proposal can be filed.
8. **Validate and propose with a parent.** Use `ih_validate_against_trace`, or `ih_propose_bulk_push` with `dryRun: true` when staging several changes, before writing.
9. **File a link with the matching call.** To add or remove only a link, use `ih_link_requirements` or `ih_unlink_requirements`. To change links as part of an edit, pass `addLinks` and `removeLinks` on `ih_propose_edit` (or `addProfileIds` and `removeProfileIds` on a User Need). A User Need traces to a User Profile with type `addresses` and `targetProfileId` set to the profile human id (for example `UP-004`); do not also pass `targetId`. Suggest mode files a proposal; Accept applies only the named link and keeps the others. Allowed pairs are in the platform schema. Review outcomes are in the platform `mcp.md`.

Exact REST calls and lifecycle behavior (reclassify, un-accept, soft delete, restore, reopen) are in the platform `rest-api.md` (this skill's `references/rest-api.md` points there). These authoring rules always apply:

- Failed propose → **422** (unknown `type` → **400**); nothing is stored. A pending UN or Profile is not a valid parent; wait for Accept, then propose the child.
- Requirements are never hard-deleted. Human Readable Durable IDs are never renumbered or reused.
- **No-op edits:** do not propose an edit whose title and statement match the current revision.
- Never re-propose to dodge a Reject; ask the human.
- **Withdraw one proposal** with `ih_lifecycle_action` `action: "withdraw_proposal"` and the **proposal id**. A requirement id is **422** `requirement_id`. Withdraw a whole pending requirement (`withdraw_requirement`) only when the user asks for exactly that.

## 5. Report back to the human

After proposing, tell the issuer in one short message:

- Each proposal `id`, `kind` (add/edit/obsolete/…), and the target Human Readable Durable ID or title. For a TraceLink, report the link id when the result is `bundled` or `applied`, or the `proposalId` when it is a standalone review between two existing requirements, and say which of those it is.
- Where to decide: **Review changes** on www (`https://www.ideationhub.me`, avatar → *Agent proposals*, or the Product's Review tab).
- Anything rejected with **422** and why. Never include secrets or tokens.

## Writing rules (PRD rubric)

- **Limit each body to that requirement's purpose.** On a new requirement or a revision, include only the detail that states that requirement. Omit background and explanatory subject detail that is not needed to state it. A separate capability belongs in its own traced requirement.
  - Example of this rule, not a change to a living record: a requirement whose purpose is an optional per-project STPA layer states that per-project on/off capability. It does not enumerate what the STPA layer contains or explain STPA.
- **Plain, unambiguous sentences (every type).** A newcomer should understand each title and body on first read. Name a clear subject that exists in the product, with an accurate verb (a permission declares; it does not decide). No jargon, coined phrases, or internal shorthand (e.g. "no silent living writes"). Do not use words with more than one meaning. When drafting or revising, reread the text, check that nothing can be read more than one way, and rewrite any ambiguous term so it has the one intended meaning. Keep sentences short and state the point first; put any list after it instead of packing many items into one sentence. Run the editor pass in §4 step 5. Stay concise.
- **Titles never name a User Profile or role** (for example "Agents …", "Product Owner can …", "Developers …"). The trace from the Product Requirement up to its User Need, and from there to the Profile, already carries who it is for. Write the title as the capability itself.
- **Titles state what the Product does**, starting with an action verb ("Show …", "Record …", "Search …"). Do not use a bare noun phrase naming a concept or metric, where words like "lead time" could read as a noun or a verb.
  - Avoid: "Lead time and its trend for each Product Requirement"
  - Prefer: "Show lead time and its trend for each Product Requirement"
- A PRD states only **WHAT the product must do**, concisely. A checker can pass/fail it without the linked UN.
- No who / why / role / purpose / outcome — the TraceLinks up to the UN carry that.
- No component counts, UI surfaces (pages, dialogs, controls), implementation shape, or design/engineering decisions.
- No acceptance criteria in a PRD body. Write them as functional verification test specifications linked to the PRD.
- **Refer to another requirement with a Markdown link whose text is the target's human ID.** This is the one allowed form of an ID in a body. Write `[PRD-123](/products/<product uuid>/docs/<type slug>/<requirement uuid>)`. Do not use a bare ID, the target's title as the link text, or "extends PRD-…" prose. The type slug is `prd`, `user-needs`, `business-objectives`, `system-design`, `ux-design`, `component-design`, `implementation`, `validation-test`, `verification-test` or `unit-test`. Use the uuid in the address, not the human id, so the link survives renames and revisions. A web address is `[text](https://…)`. A link only lets the reader click through. The relationship itself is still recorded as a TraceLink.
- User Needs open with the objective as a verb phrase ("Understand …", "Write …"), never "Ability to …", and state only one objective: what the user needs to accomplish (WHAT), not HOW it is delivered, not an action that enables it, and not what the user does not want or should not have to do. A Product Requirement that covers an enabling action traces to the User Need it serves; do not add a clause for that action in this need.
  - *Example:* Avoid: "Ability to request implementation of a requirement and understand the state of delivery of my product's features." Prefer: "Understand the state of delivery of my product's features."
- **No user-story form in User Needs.** Never write "As a <role>, I need …" or otherwise restate who the user is; the User Profile trace already carries the role, so it is redundant. State only the objective, simply, without mechanisms such as credentials, permission models, CRUD lists, or approval workflow (those belong in the PRDs beneath).
  - *Example (UN-097, 2026-11-09):* Bad: "As a Product Owner enabling agents, I need an agent with Product-scoped credentials and RBAC to Create, Read, Update, and Delete requirements on that Product via MCP, with Create/Update/Delete as proposals only." Good: "Create an account for agents to read and propose requirement changes to a product I own."
- A User Profile declares only the person (who, role, context), never their needs or one capability; see the Profile capability rule. Link each User Need to every Profile it serves. Reject a Profile proposal that fails this, and give the reason.
- **Layered pattern — apply on every create or update:** User Profiles declare only the person; User Needs declare only the WHAT (the user's objectives); Product Requirements declare the HOW, each tracing up to the User Needs it satisfies (platform schema). The PRD's "how" is the product capability that satisfies the need, stated as testable product behavior, never implementation or design.

**Bad:** "MCP server at /api/mcp with existing agent surface plus three tools." (count + implementation)  
**Good:** "Search requirement titles and bodies within one Product." (one behavior a test can check, no implementation)

## Scope boundary and overlap (rubric)

Apply before every add or edit. Breadth is fine; overlapping another same-type requirement is not.

- **Boundary.** One line: this owns [capability] under [conditions]; it does not own [adjacent]. If you cannot name the outside, ask.
- **One coherent feature.** Fields and conditions under that feature are fine. Two separately testable features are not. One owner per obligation; do not copy it into another body — trace instead.
- **Overlap.** Same-type records both claiming the same obligation. Not overlap: parent vs child, a TraceLink, or the same words for a different actor or scope.
- **Fix.** Extend before you add (§4 step 2): fold into the existing owner and propose the absorbed one obsolete. Otherwise narrow, split, or move the shared behavior to one owner and trace. Children of a parent partition its scope (flag gaps). A parent may be broader than any one child.
- **Ask** when overlap is partial, more than one fix is reasonable, the boundary is unclear, or traces look wrong. Never silently merge, broaden, or withdraw.

## Business Need vs User Need (rubric)

- A **Business Need** is what the business that owns the product must accomplish for itself: revenue, cost, compliance, strategy, or operations. It is not for the sake of the customer.
- Anything meant to help a customer or user is a **User Need**, linked to the relevant customer User Profiles.

Pass/fail checks when filing or reviewing:

1. **Who benefits directly?** The owning business → Business Need. A customer or user → User Need.
2. **Can you name the customer Profile it serves?** If yes, it is a User Need.
3. A Business Need statement contains no customer benefit or outcome.
4. Reviewers flag any misfiled item and re-propose it as the right type; never delete it.

Examples:

- BN-002 "Delivery speed is visible and measurable" is misfiled. As a User Need (Profile: Product Owner): "See how long my products take to build."
- BN-038 "Agent-navigable requirements via MCP and companion skill" is a User Need: external agents, and the people who run them, navigating and safely changing requirements are customers being served.

## STPA (safety analysis)

Safety, reliability, or hazard analysis follows an iterative STPA workflow kept in its own optional layer, never mixed into requirement text. Load `references/stpa.md` before doing any STPA work.

## Anti-patterns

- Echoing a secret or token anywhere
- Writing without reading neighbors, or adding duplicates
- Breaking a trace rule in the platform schema
- Leaving a record with no parent unflagged. A TraceLink may name a still-pending requirement.
- Filing a Suggest TraceLink that has one pending-add end as its own Review item, showing a link between two pending adds as its own Review row, or treating a link between two existing requirements as already approved
- A requirement or revision body that explains background or a related capability instead of stating that requirement
- Asking for a `review` scope (**422**) or trying to Accept your own proposals
- Writing a living record from a Suggest credential, or from an Edit credential when the call says `mode: "suggest"` — that change waits in Review
- Editing a product that is not in `editDirectlyProductIds` and not covered by `edit` scope

## Changelog

Current: 1.7.2 (2026-11-28) — Refer to another requirement with a Markdown link whose text is the target's human ID and whose address is its in-app uuid path; this is the one allowed form of an ID in a body; TraceLinks still record the relationship. 1.7.1 (2026-11-27) — The schema pointer no longer says there are four link types. Principle (`PRN`) and `respects` are defined in the platform schema. Full history: `references/changelog.md`.
