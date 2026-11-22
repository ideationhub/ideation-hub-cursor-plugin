# Ideation Hub Developer plugin

System design, implementation, and assessment for Ideation Hub Product Requirements.

The agent instructions are still an Agent Skill so hosts can load them: `skills/ideation-hub-developer/SKILL.md` (name `ideation-hub-developer`, version 0.4.0). `skills/ideation-hub-system-design/` is the deprecated alias. Both names still resolve.

This plugin requires the **Ideation Hub platform** plugin for the schema, REST API, and MCP tools. Cursor cannot declare that dependency in the manifest. Install `ideation-hub-platform` next to this plugin. Requirement wording changes go through the **Ideation Hub Requirements** plugin (`ideation-hub-requirements-agent`).

Do not install this plugin and the legacy root `ideation-hub` bundle at the same time. That bundle already loads these instructions.
