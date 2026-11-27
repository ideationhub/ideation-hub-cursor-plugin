# Ideation Hub Requirements plugin

Authoring workflow and writing rubrics for Business Needs, User Profiles, User Needs, and Product Requirements.

The agent instructions are still an Agent Skill so hosts can load them: `skills/ideation-hub-requirements-agent/SKILL.md`. The skill name `ideation-hub-requirements-agent` is unchanged so existing installs keep resolving it. The user-facing name of this package is the Ideation Hub Requirements plugin (1.7.1).

This plugin requires the **Ideation Hub platform** plugin for the schema, REST API, and MCP tools. Cursor cannot declare that dependency in the manifest. Install `ideation-hub-platform` next to this plugin.

Do not install this plugin and the legacy root `ideation-hub` bundle at the same time. That bundle already loads these instructions.
