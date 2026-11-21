# Research summary

What was considered when building this skill (November 2026), what was reused, and what had to be original. Only public sources are listed.

## Skill format and harness support

| Source | What it established for this skill |
| --- | --- |
| Agent Skills specification — https://agentskills.io/specification (Apache-2.0) | Required `name` (matches the folder) and `description`; optional `license`, `compatibility`, `metadata`; `references/` and `assets/` folders; progressive disclosure (short SKILL.md, references loaded on demand, one level deep). **Followed exactly.** Version lives in `metadata.version`. |
| Anthropic public skills — https://github.com/anthropics/skills (per-skill licences, mostly Apache-2.0) | Patterns for skill structure. No architecture or system-design skill. Reference only. |
| Claude Code skills and plugins — https://code.claude.com/docs/en/skills/, https://code.claude.com/docs/en/plugins/overview | Skill folders, plugin manifest `.claude-plugin/plugin.json`, root `.mcp.json`. Claude-only features (command injection, extra frontmatter) avoided. |
| Cursor skills and MCP — https://cursor.com/docs/skills, https://cursor.com/docs/mcp | Loads `.cursor/skills`, `.agents/skills` and Claude/Codex folders; MCP headers with `${env:VAR}`. |
| OpenAI Codex skills — https://learn.chatgpt.com/docs/build-skills; MCP config — https://learn.chatgpt.com/docs/config-file/config-reference | `.agents/skills` from the working directory up to the repo root and `~/.agents/skills`; skill descriptions may be shortened, so key trigger words come first; `env_http_headers` keeps secrets out of config. |
| Grok Build — https://docs.x.ai/build/features/skills-plugins-marketplaces, https://docs.x.ai/build/features/mcp-servers | `.grok/skills`, `~/.agents/skills`, plugins; reads Claude Code plugins and `.mcp.json`; ignores unknown frontmatter keys. |
| Model Context Protocol — https://modelcontextprotocol.io/specification/2025-06-18 | Transport expectations for the adapters (Streamable HTTP). |

**Conclusion:** one spec-compliant folder works across Cursor, Claude Code, Codex, Grok Build and Grok Bot. Only MCP credential setup and plugin manifests differ, so those live in `adapters/` and the methodology stays host-neutral.

## Community skills

| Source | Licence | Verdict |
| --- | --- | --- |
| wshobson/agents (architecture-patterns, api-design-principles, microservices-patterns, C4 agents, architect-review) — https://github.com/wshobson/agents | MIT | Broad pattern catalogue, pattern-first and leaning towards microservices, with no traceability. Used only as a checklist of pattern names; text is original. |
| muthub-ai/c4-skills (c4designer, adr-scribe) — https://github.com/muthub-ai/c4-skills | MIT | Good idea of separate files per operating mode and a review checklist. Not adopted: it stores ADRs and diagrams as files, which would compete with Ideation Hub as the record. |
| sbrudz/agent-skills, architecture-decision-records — https://github.com/sbrudz/agent-skills | MIT | Small; reference only. |
| obra/superpowers (brainstorming, writing-plans, verification-before-completion) — https://github.com/obra/superpowers | MIT | Collaboration habits adapted in spirit: present a design in sections, ask few focused questions, evidence before claims. Text is original. |
| github/spec-kit — https://github.com/github/spec-kit | MIT | Spec → plan → tasks mirrors requirements → design → handoff; principles as gates mirror Ideation Hub NFR/principle requirements. Its file-based specs conflict with a system of record, so reference only. |

No public skill was found that turns requirements into designs against a live, approval-gated traceability store. That core is original.

## Methods

| Method | Source and licence | How it is used |
| --- | --- | --- |
| Architecture Decision Records, MADR | https://adr.github.io, https://adr.github.io/madr/ — MADR is MIT or CC0-1.0 | Decision SD body ("MADR-lite": context, drivers, options, decision, consequences, confirmation). Structure reused with attribution. |
| C4 model | https://c4model.com — CC BY 4.0 | Context → container → component levels map onto SD and CD records. Diagrams are derived views only. Attributed. |
| arc42 | https://arc42.org — CC BY-SA 4.0 | Used only as a mental coverage checklist; no text copied, to avoid ShareAlike obligations. |
| ATAM (SEI) | https://www.sei.cmu.edu/library/atam-method-for-architecture-evaluation/ | Lightweight quality-attribute scenarios, sensitivity and trade-off points, risks and non-risks, sized for a conversation. Original wording. |
| Domain-driven design | https://www.domainlanguage.com/ddd/reference/; DDD Crew https://github.com/ddd-crew (CC BY-SA 4.0) | Recommended only for complex domains; no text copied. |
| ISO/IEC/IEEE 29148:2018 | https://www.iso.org/standard/72089.html (paid standard) | Concepts only (necessary, unambiguous, verifiable, traceable; bidirectional traceability). Not quoted. |
| Mermaid C4 diagrams | https://mermaid.js.org/syntax/c4.html (MIT) | Optional rendering of derived views. |
| Design controls traceability | Ideation Hub's own model | Design inputs (PRDs) → design outputs (SD/CD/UXD/IMPL) → verification (VER/UT); gaps and impact through trace links. |

## Original work

1. The mapping of every design artifact onto Ideation Hub's existing record types, link directions and tags.
2. A write protocol built on Ideation Hub's contract: the credential decides direct vs proposal, so the skill always passes `mode: "suggest"`; dry-run first; one pending proposal per record; revision checks.
3. Proportionality rungs that size a design to its requirements.
4. Coverage, justification, direction and staleness checks computed from Ideation Hub data.
5. Impact analysis from revision pins on trace links.
6. A handoff brief built from accepted records, plus the `design_ready` development-phase event.
