# Changelog

## 0.5.3 (2026-11-28) — beta

- When you start implementing a requirement, record `phase: "implementation_started"` with `ih_record_development_phase_event` (`delivery:write`, no requester). The skill already records `implementation_complete`, `merged_to_main`, `deploy_completed`, and a status assessment after shipping.
- `skills/ideation-hub-system-design/SKILL.md` metadata version tracks this release.


## 0.5.2 (2026-11-28) — beta

- While implementing, record development status on each in-scope Product Requirement with `ih_record_development_phase_event`: `implementation_complete` when the work is finished, `merged_to_main` when the pull request merges, and `deploy_completed` when production deployment completes. After shipping, record a status assessment with `ih_record_status_assessment`. There is no agent-writable phase for starting implementation; do not invent one.
- `skills/ideation-hub-system-design/SKILL.md` metadata version tracks this release.

## 0.5.1 (2026-11-28) — beta

- Technology choices (language, framework, platform, cloud, or vendor) come from the product's and tenant's applicable Principles, read with `ih_list_applicable_principles`. If none apply, ask the human. This skill does not recommend a specific technology.
- Examples and references no longer prescribe a stack (for example a cross-platform codebase, a named database, or a managed provider). `skills/ideation-hub-system-design/SKILL.md` metadata version tracks this release.


## 0.5.0 (2026-11-27) — beta

- A Principle is a `PRN` record, the broadest record, not a Product Requirement tagged `principle`. `respects` runs from a requirement to a Principle and is shown the other way as Respected by. This skill does not author a Principle and does not create that link. Applicable principles are read with `ih_list_applicable_principles`. Tenant-wide adoption is a human tenant admin action in the web app.
- An NFR stays a Product Requirement tagged `NFR`.
- `skills/ideation-hub-system-design/SKILL.md` metadata version tracks this release.

## 0.4.0 (2026-11-22) — beta

- This package is the Ideation Hub developer plugin. The agent skill name stays `ideation-hub-developer`, and `ideation-hub-system-design` remains a deprecated alias, so existing names still resolve.
- Schema, REST, and MCP details now live in the Ideation Hub platform plugin (`plugins/ideation-hub-platform`). This plugin points there. The requirements plugin keeps authoring rubrics only.
- Assessment procedure in `references/assessment.md` is unchanged (PRD-180).

## 0.3.0 (2026-11-22) — beta

- Before design or implementation, trace each in-scope Product Requirement up to its User Needs, then to the User Profiles and Business Needs behind them. Summarize who the user is, what they are trying to do, and why, and let that shape the work. A missing or conflicting User Need is flagged; the agent proposes a trace link or wording change and does not guess or edit the requirement. Procedure: `references/implementation.md`.
- Implementation approach in `references/implementation.md`: follow the codebase's patterns, take the smallest end-to-end slice, keep data model, API, and UI consistent, prefer reversible choices, write unit and verification tests with the change, keep pull requests small, and cite `PRD-xxx` in commits and pull requests. Implementation and test trace links are still recorded the way `SKILL.md` and `references/traceability.md` already describe.
- Schema, trace-link types, revision and status vocabulary, the MCP tool list, and REST routes are no longer restated. Those files point at `ideation-hub-requirements-agent` (`SKILL.md`, `references/retrieval.md`, `references/rest-api.md`). Design-link direction, revision pins, design `parentId`, and assessment tools stay here because that skill does not state them.
- Assessment procedure in `references/assessment.md` is unchanged (PRD-180, applying PRD-026, PRD-172, and PRD-179).
- `skills/ideation-hub-system-design/SKILL.md` remains a deprecated alias of this skill. Its metadata version tracks this release so existing installs still resolve the old name.

## 0.2.0 (2026-11-22) — beta

- Renamed the skill from `ideation-hub-system-design` to `ideation-hub-developer`. `skills/ideation-hub-system-design/SKILL.md` remains as a deprecated alias that points here, so existing installs still resolve the old name.
- Scope is system design, implementation, and assessment of implementation and test coverage status for Product Requirements.
- Assessment procedure in `references/assessment.md` (PRD-180). It applies the completeness rules in PRD-026 and the coverage rules in PRD-172, judges the current revision only, and records confirmation reviews and verdicts as PRD-179 describes (`status:write`).

## 0.1.0 (2026-11-21) — beta

- First release.
- Phases A–F (understand, first pass, collaborate, iterate, validate, handoff) as an iterative loop.
- Design artifacts mapped onto existing SD, CD and UXD records with `arch.*` tags; no server or schema changes.
- Write protocol: always `mode: "suggest"` unless the user explicitly asks for a direct write; dry-run before filing; amend the one pending proposal; `expectedRevision` on edits.
- Handoff always records the `design_ready` development-phase event per in-scope Product Requirement, and reports plainly when the credential lacks `delivery:write`.
- Test links follow Ideation Hub's rule: designs are verified by VER or UT.
- Adapters for Cursor, Claude Code, OpenAI Codex, Grok Build, Grok Bot and generic hosts.
