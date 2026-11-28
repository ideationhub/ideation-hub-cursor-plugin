---
name: ideation-hub-system-design
description: Deprecated alias of the Ideation Hub developer skill. Use when an existing install still invokes ideation-hub-system-design. Follow ideation-hub-developer and do not use this folder as a separate workflow.
license: MIT
compatibility: Requires the ideation-hub-developer skill in a sibling directory, the Ideation Hub platform skill, and the ideation-hub-requirements-agent skill.
metadata:
  version: "0.5.0"
  status: deprecated
  aliasOf: ideation-hub-developer
  author: Ideation Hub
---

# ideation-hub-system-design (deprecated)

This name is deprecated. It is a thin alias of the **Ideation Hub developer skill** so existing installs that still resolve `ideation-hub-system-design` keep working.

Do the work with `ideation-hub-developer`:

- Name: `ideation-hub-developer`
- Directory: `../ideation-hub-developer/`
- Instructions: `../ideation-hub-developer/SKILL.md`

On-demand references, adapters, and examples live in that directory, not in this one. Load that skill and follow it. The schema, REST API, and MCP tools are in the Ideation Hub platform skill.
