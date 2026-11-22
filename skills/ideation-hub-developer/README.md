# ideation-hub-developer (0.2.0 beta)

A portable Agent Skill that helps any coding agent do system design, implementation, and assessment of implementation and test coverage status for Product Requirements in [Ideation Hub](https://www.ideationhub.me). The agent reads the accepted requirements over MCP, proposes a proportionate design, iterates, validates coverage and traceability, implements when asked, and assesses each Product Requirement it is given.

Ideation Hub stays the system of record. Designs are stored there as System Design (SD), Component Design (CD) and UX Design (UXD) records, traced to the Product Requirements that justify them, and filed as Review proposals that a person accepts. Implementation and test-coverage assessments are recorded on the current revision (PRD-180, PRD-026, PRD-172, PRD-179).

`ideation-hub-system-design` is a deprecated alias of this skill. Its folder holds only a `SKILL.md` that points here, so existing installs that still invoke the old name keep resolving.

## What is in the package

```text
ideation-hub-developer/
├── SKILL.md                  entry point the agent loads
├── references/               loaded on demand, one level deep
│   ├── methodology.md        phases A–F, proportionality ladder
│   ├── design-elements.md    how each artifact maps onto SD/CD/UXD, tags and templates
│   ├── traceability.md       link direction, coverage, impact, staleness
│   ├── tradeoff-analysis.md  quality scenarios, options tables, build vs buy
│   ├── architecture-patterns.md  styles and when not to use them
│   ├── validation.md         review checklist and report
│   ├── implementation-handoff.md handoff brief and the design_ready event
│   ├── assessment.md         implementation and test-coverage assessment (PRD-180)
│   └── write-protocol.md     exact MCP write rules and error handling
├── assets/                   JSON Schemas for scratch plans, handoff briefs, validation reports
├── examples/                 four illustrative conversations
├── adapters/                 install and MCP setup per harness
├── RESEARCH.md               skills and methods considered, licences
├── NOTES.md                  assumptions, open questions, future enhancements
├── CHANGELOG.md
└── README.md
```

There are no scripts, so the skill needs no runtime.

## Requirements

- The Ideation Hub MCP server and an agent credential (Suggest recommended). Add `delivery:write` if handoffs should record the `design_ready` development-phase event. Add `status:write` to record confirmation reviews and implementation or test-coverage assessments. `status:write` never edits a title, body, or trace link.
- The `ideation-hub-requirements-agent` skill, which owns credentials, MCP usage, requirement types and proposal rules. Both ship in this plugin.

## Install

| Harness | Quick path | Details |
| --- | --- | --- |
| Cursor | Install the Ideation Hub plugin | `adapters/cursor.md` |
| Claude Code | Enable the plugin (`.claude-plugin/plugin.json`) or copy to `.claude/skills/` | `adapters/claude-code.md` |
| OpenAI Codex | Copy the requirements skill, this skill, and the deprecated alias to `~/.agents/skills/` or the repo's `.agents/skills/`; add the server to `~/.codex/config.toml` | `adapters/codex.md` |
| Grok Build | `grok plugin install ideationhub/ideation-hub-cursor-plugin --trust` | `adapters/grok-build.md` |
| Grok Bot | Install the plugin, or save the skills and add the connector | `adapters/grok-bot.md` |
| Other | Any Agent Skills loader plus an MCP client | `adapters/generic.md` |

## Use

Ask for a design, an implementation, a review, a handoff, or an assessment, naming the product and scope:

- "Design the system for the requirements tagged `checkout` in ShopTrack."
- "Implement the accepted design for PRD-030."
- "We now need offline support. What changes in the design?"
- "Review the design for PRD section Billing."
- "Hand the search design to a coding agent."
- "Assess whether PRD-030 and PRD-091 are implemented and covered by tests."

The agent always files design changes as proposals (`mode: "suggest"`) unless you explicitly ask it to write directly, never changes an accepted design without your confirmation, and never edits requirements itself. When the code and a Product Requirement disagree, it reports the difference.

## Status

0.2.0 is a beta. The conventions (tags, record templates, link patterns) may change before 1.0 as real designs are recorded. See `NOTES.md` for known gaps. The previous name, `ideation-hub-system-design`, remains as a deprecated alias.

## Licence

MIT, as the rest of this repository. The decision-record structure is adapted from MADR (MIT or CC0-1.0); C4 level guidance follows the C4 model (CC BY 4.0, https://c4model.com). See `RESEARCH.md`.
