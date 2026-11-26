# Changelog

## 0.4.1 (2026-11-26) — beta

- Shopping Price Tally worked example (`examples/01-first-pass-design.md`) follows ADR-002 (2026-11-24): native Swift for iOS and Kotlin for Android, iPhone-first development and user testing, both platforms designed side by side. That replaces the one-cross-platform-codebase recommendation and the earlier Flutter direction.
- The same example no longer forces a price confirmation or sells the first 10 tallies as a credit pack. The app takes the lowest price on the scanned tag by default, the $1.99 download includes the first 10 tallies, a credit is used on the first scan of a trip, and later packs are consumable in-app purchases.
- The example adds download budgets (30 MB on iPhone, 25 MB on Android), the Android vitals startup flags from the 23 November 2026 research (cold 5 seconds, warm 2 seconds, hot 1 second), and a short store-rules list with official links. The design method is unchanged.

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
