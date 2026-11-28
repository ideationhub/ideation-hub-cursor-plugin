# Implementation

Load this file before you design and before you implement. The user-need trace below is required for both. The rest of the file is how to build the change once that trace is done and the user has asked you to implement.

Entity types, trace-link types, revision and status words, MCP tool arguments, and REST routes are not restated here. They live in the Ideation Hub platform skill. From this file, when that skill is a sibling folder:

- `../../ideation-hub-platform/references/schema.md`
- `../../ideation-hub-platform/references/mcp.md`
- `../../ideation-hub-platform/references/rest-api.md`

If a relative path is missing, open the installed `ideation-hub-platform` skill and read the same file. How to read a small slice is the requirements skill's `references/retrieval.md` (`../../ideation-hub-requirements-agent/references/retrieval.md`).

## User-need trace (before design and before implementation)

Do this for every Product Requirement in scope before Phase B (first-pass design) and before any code. If you already did it for this scope in this conversation, use that summary. Do not design or write code for a requirement you have flagged until the user resolves it.

For each in-scope Product Requirement:

1. Read the accepted current revision only. How to call the read, including `includePending: false`, is in the platform `mcp.md`. What each status word means is in the platform schema. Do not use `pendingProposal` text as the requirement.
2. Trace **upward** through trace links. From the Product Requirement, walk `direction: "up"` (`ih_get_neighbors`, or REST neighbors with `direction=up`) to the User Needs it addresses. From each User Need, walk up again to the User Profiles it addresses and the Business Needs it addresses. Read each record you will rely on (`ih_get_requirement`, or `GET /api/requirements/{id}`). A Product Requirement never links straight to a User Profile; the profile is reached only through a User Need.
3. Summarize, with IDs and revisions (`PRD-012@3`, `UN-004@1`):
   - **Who** the user is: the User Profile's name, role, and context. Nothing else belongs on a profile.
   - **What** they are trying to do: the User Need, which is the objective, not a design.
   - **Why**: the Business Needs those User Needs address.
4. Let that summary shape the design and the implementation. It decides the UX, the defaults, the edge cases, the error messages, and what to cut. A behavior that serves no cited User Need is out of scope unless the user adds it.
5. **Flag gaps and conflicts. Do not guess.** Stop on a Product Requirement that has no linked User Need, or whose text conflicts with the User Needs it traces to (the requirement would make the user do something the need does not ask for, or would block what the need asks for). Tell the user what you found, cite the IDs, and propose a trace link or a wording change for them to accept. Do not invent the user, the need, or the wording. Do not edit the requirement directly. After the user agrees, file that proposal through the requirements skill's authoring workflow with `mode: "suggest"`.

Show the summary before the first design section or the first code change. Keep it short enough to read in one pass.

## How to implement

The Product Requirement says what the product must do. This is how you build it. Cite the PRD and design IDs you are implementing.

1. **Follow the codebase.** Read the existing patterns first: structure, naming, data access, API shape, UI components, error handling, and tests. Match them. Do not introduce a new pattern for one requirement. Do not introduce a language, framework, platform, cloud, or vendor that the codebase and the applicable Principles do not already use. If the work needs one and no Principle applies, ask the human.
2. **Smallest thin slice.** Pick the smallest change that satisfies the requirement end to end, one path a user can complete through the data model, the API, and the UI. Cut anything the cited User Needs and Product Requirements do not need.
3. **Consistent layers.** Design the data model, API, and UI changes so they agree with each other and with the rest of the system. A field, an error, or a screen means the same thing in every layer that has it.
4. **Simple and reversible.** Prefer a choice you can replace later without migrating users or stored data. A hard-to-reverse choice is a design decision: record it as `design-elements.md` describes before you build it.
5. **Tests with the change.** Write unit tests and verification tests alongside the change. Write or update those tests before a major refactor. The tests check the behavior the Product Requirement states.
6. **Small pull requests.** One requirement, or one thin slice of it, per pull request. Cite the requirement IDs (`PRD-xxx`, plus the design IDs you implemented) in the commit message and the pull request.
7. **Record the traces where this skill already says how.** Record the Implementation as `SKILL.md` (Implementation) describes: an IMPL linked `CD realized_by IMPL` or `SD realized_by IMPL`, with `repoPath`, `prUrl`, and `commitSha` when you have them. Record Verification Tests and Unit Tests on the design as `traceability.md` (Test links) describes, using the test fields in the platform schema (`testPath`, `lastResult`). File both through the platform write rule, with `mode: "suggest"` unless the user explicitly asks for a direct write on that product in this conversation. Do not invent a second write path.
8. **Record development status.** Follow [Record development status](#record-development-status) for each in-scope Product Requirement. Those writes are direct, not proposals.

## Record development status

For each in-scope Product Requirement you implement, update its development status with direct writes. Do not invent a phase name. Pass `evidenceUrl` (the pull request or deploy URL) when you have one. If a call returns 403, tell the user which permission is missing and continue. Never work around it.

1. There is no agent-writable phase for starting implementation. When the implementation for that requirement is finished, call `ih_record_development_phase_event` with `phase: "implementation_complete"`. It needs `delivery:write`.
2. When the pull request merges, record `phase: "merged_to_main"`. When production deployment completes, record `phase: "deploy_completed"`. You may also record `tests_passing`, `pushed_to_git`, and `deploy_started` at those moments. Events are append-only; the same phase may appear more than once.
3. After shipping, record a status assessment of the current revision with `ih_record_status_assessment` as `assessment.md` describes. It needs `status:write` and cannot change a title, body, or trace link.

## When not to implement

A handoff brief is for another agent. Do not stop at handoff when you were asked to implement.

If the behavior you would write contradicts a Product Requirement, stop and report the difference. Do not change the requirement.

A full assessment of other requirements remains a separate task (Phase H). After you ship a requirement you implemented, record its status assessment as this file describes, following `assessment.md` for how to judge and record the verdicts.
