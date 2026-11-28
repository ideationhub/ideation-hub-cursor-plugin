---
name: ideation-hub-platform
description: Shared Ideation Hub reference for record types, trace links, revisions, the REST API, and Model Context Protocol tools. Use when designing, writing, or assessing Ideation Hub requirements and you need the schema or the tool catalog. Requires no other skill.
license: MIT
compatibility: Reference files only. No scripts. The Model Context Protocol server is https://api.ideationhub.me/api/mcp.
metadata:
  version: "1.1.0"
  author: Ideation Hub
---

# Ideation Hub platform

This folder is the single description of the Ideation Hub schema, the REST API, and the Model Context Protocol tools. The developer and requirements skills point here. Do not copy these files into another skill.

Load only the file you need:

- `references/schema.md` — record types, durable identifiers, revisions, status words, and trace links
- `references/mcp.md` — Model Context Protocol tools, permissions, and the suggest-versus-edit rule
- `references/rest-api.md` — REST routes and lifecycle behavior
- `references/credentials.md` — how a person creates an agent credential

Credential setup for a person is in `references/credentials.md`. The secret is never written in these files.

The developer and requirements skills reach these files at `../ideation-hub-platform/references/` from their own folders, so keep the skill folders side by side. The Ideation Hub plugin ships all three that way.

Current: 1.1.0 (2026-11-27) — Principle (`PRN`) and the `respects` link are in `references/schema.md`. Applicable principles, product adoption, principle conflicts, and the tenant-admin adoption note are in `references/mcp.md`. Full history: `CHANGELOG.md`.
