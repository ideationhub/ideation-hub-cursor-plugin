# Changelog

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
