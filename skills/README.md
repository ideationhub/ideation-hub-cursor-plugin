# Compatibility paths

These entries are symlinks so existing paths keep resolving inside a checkout of this repository:

| Old path | Resolves to |
| --- | --- |
| `skills/ideation-hub-developer` | `plugins/ideation-hub-developer/skills/ideation-hub-developer` |
| `skills/ideation-hub-system-design` | `plugins/ideation-hub-developer/skills/ideation-hub-system-design` |
| `skills/ideation-hub-requirements-agent` | `plugins/ideation-hub-requirements/skills/ideation-hub-requirements-agent` |

`ideation-hub-system-design` remains a deprecated alias of the developer plugin. The skill names `ideation-hub-developer` and `ideation-hub-requirements-agent` are unchanged.

Copying only this `skills/` directory with a tool that does not follow symlinks will not include the instructions. Install the plugin directories under `plugins/` as siblings instead.
