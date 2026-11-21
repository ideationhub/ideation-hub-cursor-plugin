# Changelog

## 0.1.0 (2026-11-21) — beta

- First release.
- Phases A–F (understand, first pass, collaborate, iterate, validate, handoff) as an iterative loop.
- Design artifacts mapped onto existing SD, CD and UXD records with `arch.*` tags; no server or schema changes.
- Write protocol: always `mode: "suggest"` unless the user explicitly asks for a direct write; dry-run before filing; amend the one pending proposal; `expectedRevision` on edits.
- Handoff always records the `design_ready` development-phase event per in-scope Product Requirement, and reports plainly when the credential lacks `delivery:write`.
- Test links follow Ideation Hub's rule: designs are verified by VER or UT.
- Adapters for Cursor, Claude Code, OpenAI Codex, Grok Build, Grok Bot and generic hosts.
