---
name: ideation-hub-platform
description: Shared Ideation Hub reference for record types, trace links, revisions, the REST API, and Model Context Protocol tools. Use when designing, writing, or assessing Ideation Hub requirements and you need the schema or the tool catalog. Requires no other plugin.
license: MIT
compatibility: Reference files only. No scripts. The Model Context Protocol server is https://api.ideationhub.me/api/mcp.
metadata:
  version: "1.1.0"
  author: Ideation Hub
---

# Ideation Hub platform

This folder is the single description of the Ideation Hub schema, the REST API, and the Model Context Protocol tools. The developer and requirements plugins point here. Do not copy these files into another plugin.

Load only the file you need:

- `references/schema.md` — record types, durable identifiers, revisions, status words, and trace links
- `references/mcp.md` — Model Context Protocol tools, permissions, and the suggest-versus-edit rule
- `references/rest-api.md` — REST routes and lifecycle behavior

Credential setup for a person is in the platform plugin `README.md`, one directory above this skill when the plugin is installed as a whole. The secret is never written in these files.

When this skill folder is copied next to `ideation-hub-developer` and `ideation-hub-requirements-agent` in one skills directory, those plugins look for `../ideation-hub-platform/references/`. When the three plugins are installed as sibling directories, they look for `../../../ideation-hub-platform/references/`, which is the same directory through the `references` link at the platform plugin root.

Current: 1.1.0 (2026-11-27) — Principle (`PRN`) and the `respects` link are in `references/schema.md`. Applicable principles, product adoption, principle conflicts, and the tenant-admin adoption note are in `references/mcp.md`. Full history: `CHANGELOG.md`.
