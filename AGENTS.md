# EVPanda documentation

## About this project

- The public documentation site for EVPanda, built on [Mintlify](https://mintlify.com) and deployed to [docs.evpanda.io](https://docs.evpanda.io) from `master`.
- Pages are MDX files with YAML frontmatter. Configuration lives in `docs.json`.
- Use the Mintlify MCP server, `https://mcp.mintlify.com`, to edit content and settings via MCP.
- Use the Mintlify docs MCP server, `https://www.mintlify.com/docs/mcp`, to query information about using Mintlify via MCP.

### Sources of truth

Documentation must match shipped code, not intent. When documenting SDK behavior, read the source:

| Subject | Source |
| --- | --- |
| Go SDK | `../evpanda-go`, the reference implementation the other SDKs track |
| Node SDK | `../evpanda-node` |
| Python SDK | `../evpanda-py` |
| Ingestion API contract | `../evpanda/apispec/ingestion-api.yaml` and `../evpanda/apps/atlas/internal/ingest/` |
| Dashboard surface | `../evpanda/apispec/webapi/` and `../evpanda/apps/aurora/src/router` |
| Validation and issues | `../evpanda/apps/evproto` and `../evpanda/apps/polaris/worker` |

The SDK repos are **read-only** from here. Never edit `evpanda-go` or `evpanda-node` while working on docs.

Where the ingestion API and an SDK disagree, document the behavior a customer will actually observe, and say plainly that it is current behavior. Two such gaps are live today: the ingestion API requires the tenant pair on OCPP messages though the SDKs treat it as optional, and it requires an OCPI `response_status_code` between 100 and 599 though the SDKs allow it to be omitted. Both are rejected server side after a successful delivery, so the SDK never reports the loss.

### The shipped API surface

All three SDKs were aligned on `evpanda-go` v0.3.x. The current shape:

- `startOCPI` / `startOCPP` free functions in Node and Python, `StartOCPI` / `StartOCPP` in Go. A bad config yields an inert client carrying the fault on `.error` (Node, Python) or returned as an error (Go).
- Identity types are `Platform` (`id`, `name`) and `Charger` (`id`), each with an optional all-or-nothing `tenantId` / `tenantName` pair.
- The API key is the only required config. `endpoint` defaults to `https://ingest.evpanda.io`.
- Compression is always zstd; there is no codec option.
- `stats()` returns `captured` plus `droppedInvalid`, `droppedOversize`, `droppedEvicted`, `droppedUndeliverable`, `droppedFault`, and the two buffer gauges.
- `logMode` is `silent`, `errors`, or `debug`, overridable with `EVPANDA_LOG`.

Do not write a snippet from memory. Read the SDK source or its README first.

## Structure

```
index.mdx                        Introduction: what EVPanda is, how it works, issues
concepts/networks.mdx            Networks and API keys
concepts/identity.mdx            Per-message identity and tenants
concepts/capture-model.mdx       Data flow, redaction, limits, retention
integration/getting-started.mdx  Shared SDK surface: install, config, delivery, monitoring, troubleshooting
integration/ocpp.mdx             OCPP CSMS integration, end to end
integration/ocpi.mdx             OCPI server integration, end to end
style.css                        Custom CSS (auto-included by Mintlify)
```

Seven pages, deliberately. The sidebar is a flat list: a bare Introduction plus two groups, with no tabs and no icons.

**One page per protocol, not per language.** `integration/ocpp.mdx` and `integration/ocpi.mdx` each carry the whole integration, with every snippet in a `<CodeGroup>` tabbed Go / Node / Python **in that order**. Where an SDK genuinely differs (Node needs a body parser, Go resolves OCPI identity before the handler runs, the Python WSGI adapter reads the body up front), say so in a named callout or a "Per-SDK notes" accordion rather than forking the page.

Anything shared by both protocols belongs in `integration/getting-started.mdx`, not duplicated into each guide.

## Terminology

- **Network**: the container for one instrumented system. A **charger network** watches an OCPP CSMS; a **roaming network** watches an OCPI server. Not "project", not "server".
- **Platform**: a roaming partner in a roaming network, and the SDK type that names one. Never EVPanda itself.
- **Charger** / **charge point**: prefer "charge point" for the OCPP entity, "charger" for the dashboard row and the SDK type.
- **Issue**: a validation finding, grouped by type. Severity is `error`, `warning`, or `anomaly`.
- **Capture**: what the SDK does. It never "intercepts", "proxies", or "monitors" traffic.
- **Identity**: the per-message attribution, a `Platform` or a `Charger`.
- Say **the SDK**, not "our SDK". Say **EVPanda**, never "we".

## Style preferences

- **No em dashes or en dashes anywhere in published pages.** Use a comma, a colon, a period, or parentheses. Check with `grep -rn "\u2014" *.mdx */*.mdx` before committing.
- Technical, simple, direct. Say the thing, then stop.
- Active voice, second person ("you").
- One idea per sentence. Sentence case for headings.
- Bold for UI elements: Click **Settings**.
- Code formatting for file names, commands, paths, and code references.
- Lead with what the reader must do; put rationale after it, not before.
- Prefer a table to a bulleted list when every item has the same shape.
- Callouts are for consequences, not emphasis. `<Warning>` means "this will lose data or break"; `<Note>` means "this will surprise you"; `<Tip>` means "there is a better way".
- Keep Introduction and Core concepts short. Depth belongs in SDK integration, and only where it changes what the reader types.

## Content boundaries

- Document the shipped API surface only. Internal service names (atlas, aurora, polaris, evproto, mirage) never appear in published pages. Say "ingestion API", "dashboard", "validation engine".
- Don't document unreleased features. Where an SDK is behind, say so explicitly and give the working alternative.
- Don't invent limits, defaults, or endpoints. Every number in these docs is traceable to source.

## Working on the docs

```bash
npx mint@latest dev            # local preview at localhost:3000
npx mint@latest validate       # strict build validation
npx mint@latest broken-links   # link check
npx mint@latest a11y           # contrast and alt-text check
```

Run `validate` and `broken-links` before opening a PR. Changes deploy automatically once merged to `master`.
