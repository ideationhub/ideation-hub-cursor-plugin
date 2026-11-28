# Ideation Hub credential setup

How a person creates the agent credential that the Ideation Hub MCP server needs. The server is `https://api.ideationhub.me/api/mcp`; it reads the credential from the `X-IH-Client-Id` and `X-IH-Client-Secret` headers. The plugin's `mcp.json` fills those from `IH_CLIENT_ID` and `IH_CLIENT_SECRET`.

## Credential setup

Relay these steps in plain words. Never ask for a secret in chat.

1. Open `https://www.ideationhub.me/settings/agents?setup=grokbot` (a `next` query param is kept through sign-in) and sign in as an **Admin** of the Product.
2. On **Agent credentials** (`/settings/agents`; also Product → Agents) choose **Create**.
3. Enter a **Client ID**: letters, digits, `.` `_` `-`, starting with a letter, no spaces (for example `GrokBot`; `Grok Bot` fails). Pattern `^[A-Za-z][A-Za-z0-9._-]{2,63}$`.
4. Pick the permission for this product:
   - **Read** — `scopes: ["read"]`, `editDirectlyProductIds: []`. The agent can look, not write.
   - **Suggest** — `scopes: ["read", "propose"]`, `editDirectlyProductIds: []`. Writes land in the issuer's Review queue until a human Accepts. `suggest` is the same permission as `propose`.
   - **Edit** — `scopes: ["read", "propose"]` and `editDirectlyProductIds` set to the products this agent may change directly, or an `edit` scope on those products. Writes apply to the living record unless the call passes `mode: "suggest"` or `"propose"`.
   - Add `delivery:write` so a handoff can record `design_ready`. Add `status:write` so an assessment can be recorded. `status:write` never edits a title, body, or trace link.
5. Copy the **Client ID** and **Client Secret** from the masked secret card right away. The secret is shown **only once**.
6. Paste them into the host's credentials dialog as `IH_CLIENT_ID` and `IH_CLIENT_SECRET`. A host opens that dialog itself.
7. Confirm with `initialize` on `/api/mcp` or `ih_list_products`, echoing nothing.

Lost secret: create a new credential or rotate it on the same screen, then repeat step 5. The Admin who created the credential is the issuer and Accepts or Rejects proposals.

**Secrets rule:** never paste, echo, log, or print the secret or an access token. Reference env vars only. If a secret leaks, tell the human to rotate or revoke on the same screen.
