# Install Ideation Hub in Grok Bot

These steps were checked on 22 November 2026 against https://docs.x.ai/grok-bot/computer-and-apps and https://docs.x.ai/grok-bot/skills-routines-and-automations.

The server address in this repository is https://api.ideationhub.me/api/mcp. On 22 November 2026 a request to https://www.ideationhub.me/api/mcp also returned a successful response. The files in this repository use the api host.

Create a credential before you connect the server. Sign in to Ideation Hub at https://www.ideationhub.me. Open Settings, then Agent credentials, at https://www.ideationhub.me/settings/agents. Create a credential and copy the client identifier and the client secret. The secret is shown only once. Do not paste the secret into the chat.

## Connect the server

The Grok Bot documentation says to open Connect apps from the sidebar. In the mobile app, tap your avatar at the top left and choose Connect apps.

On the desktop app, the same page says to choose Add a custom connector at the bottom of Connect apps when the server is not already in the catalog.

Enter a name such as Ideation Hub.

Choose the type Remote HTTPS.

Enter the server address https://api.ideationhub.me/api/mcp.

The same page says optional Hypertext Transfer Protocol headers are part of that form. Add a header named `X-IH-Client-Id` with the client identifier, and a header named `X-IH-Client-Secret` with the client secret.

Choose Add plugin.

The documentation says the tools are ready from the next message, and that a team policy can block a server so it cannot be saved. If the button does not save, that policy is the documented reason to check.

Not verified: a catalog listing named Ideation Hub inside Connect apps. Use the custom connector steps above until a catalog entry is documented.

## Skills

The skills documentation says a skill is a reusable set of instructions, and that you ask the bot to save a process as a skill. It also says private skills are enabled for the current bot from Connect apps, under Private skills.

Not verified: uploading this repository's skill folders through a skill library screen, or installing this GitHub repository as a Grok Bot plugin. Those steps are not on the two pages above.

What is verified as a way to give the bot the instructions without a plugin installer: the plain instruction files in `docs/instructions/` are generated from the skill folders. You can paste or attach `docs/instructions/ideation-hub-platform.md`, `docs/instructions/ideation-hub-developer.md`, `docs/instructions/ideation-hub-requirements.md`, and `docs/instructions/ideation-hub-mobile-store-distribution.md` in a conversation. That uses the files in this repository. It is not a Grok Bot install command from the official pages.
