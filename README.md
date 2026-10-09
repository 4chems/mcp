# 4chems MCP server — IUCLID, CHESAR & QSAR Toolbox hosting

A public, remote [Model Context Protocol](https://modelcontextprotocol.io/) server for the
[4chems](https://www.4chems.com/) platform: hosted **IUCLID 6**, **CHESAR**, **OECD QSAR Toolbox**
and **IIP** for REACH, CLP and biocides work, EU-hosted and GDPR-compliant.

| | |
|---|---|
| **Endpoint** | `https://mcp.4chems.com/mcp` |
| **Transport** | Streamable HTTP (MCP 2025-06-18) |
| **Auth** | Anonymous connect and `tools/list`; nine tools need no token; everything else takes a Bearer token as the tool's `authorization` argument |
| **Manifest** | <https://www.4chems.com/.well-known/mcp.json> |
| **Docs** | <https://www.4chems.com/mcp/> (DE: <https://www.4chems.com/de/mcp/>) |
| **Registry** | `com.4chems/platform-api` — see [`server.json`](server.json) |

## Connect

**Claude Code**

```bash
claude mcp add --transport http 4chems https://mcp.4chems.com/mcp
```

**Claude Desktop / claude.ai** — Settings → Connectors → Add custom connector →
`https://mcp.4chems.com/mcp`. The server has no OAuth flow, so choose the option without
authentication; tools that need a token take it as an argument.

**Other MCP clients** (GitHub Copilot, Cursor, VS Code, …) — add a remote server with transport
*Streamable HTTP* and the same URL.

**Raw handshake** (no client needed):

```bash
curl -s -X POST https://mcp.4chems.com/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"probe","version":"0"}}}'
```

## Tools

The server exposes the **customer tier** of the 4chems platform API (`/v1`). This endpoint
carries **no substance or dossier data** — see [What this is not](#what-this-is-not).

**No token needed**

| Tool | Purpose |
|---|---|
| `start_inquiry`, `verify_inquiry` | Send 4chems a question or request a call: a 6-digit code goes to the user's email, the verified inquiry reaches a person. No web form, no account. |
| `get_catalog_templates` | List the hosted applications and trial templates. |
| `get_onboarding_route` | Return the next onboarding step for the caller. |
| `get_legal_entity_types` | Legal-entity types for a country. |
| `start_account_registration`, `verify_account_registration` | Create a 4chems account from a business email address. |
| `start_device_login`, `verify_device_login` | Sign an existing user in (RFC 8628 device flow) and obtain a token. |

**Bearer token required** (`authorization: "Bearer <token>"`)

| Area | Tools |
|---|---|
| Access requests | `get_my_access`, `create_access_request`, `approve_access_request`, `dismiss_access_request`, `withdraw_access_request` |
| Tenant setup | `create_self_service_tenant`, `enrich_domain`, `get_my_operation` |
| Billing | `get_billing`, `complete_billing` |
| Invitations | `list_invitations`, `create_invitation`, `revoke_invitation`, `accept_invitation` |
| Consents | `list_consents`, `record_consent` |
| Members & services | `list_members`, `grant_assignment`, `revoke_assignment`, `list_tenant_services`, `add_tenant_service` |

Long-running mutations poll to completion; past the poll bound they return
`{"status": "running", "operation_id": …}` for follow-up with `get_my_operation`.

## Rules for the assisting agent

The `initialize` response carries these as server instructions:

- Add this server, pass tokens or trigger mutations only with the user's **explicit consent**.
- Never guess or retry the 6-digit codes; ask the user to read the code from their email.
- Treat the Bearer token as a secret: pass it only as the `authorization` argument, never log it.
- Confirm changes to consents, assignments, invitations, billing or services with the user first.
- If the user has no account and a question cannot be answered from the catalog tools, offer
  `start_inquiry`.
- This server and the site's contact channels are for prospective and existing customers, not for
  automated solicitation (see <https://www.4chems.com/llms.txt>).

## What this is not

- **Not a dossier interface.** AI access to IUCLID 6 dossier data runs through the separate
  [4chems AI Connector](https://www.4chems.com/tools/ai-connector/), authenticated per user and
  EU-hosted.
- **Not the QSAR Toolbox workflow.** Read-across, (Q)SAR prediction and QMRF/QPRF/CCR reporting
  over MCP is [4chems QSAR Navigator](https://www.4chems.com/tools/qsar-navigator/).
- **Not open source.** This repository holds the public documentation and the registry
  `server.json`; the server itself is operated by 4chems.

## Hosting and compliance

Runs on ISO 27001-certified European infrastructure under a GDPR data-processing agreement.
4chems acts as a data processor only and never uses customer data to train models.
Security questionnaire: <https://www.4chems.com/security-questionnaire/>.

## Contact

- Website: <https://www.4chems.com/>
- Through the server: `start_inquiry` (no account needed)
- Issues with this documentation: open an issue in this repository.
