# EVPanda docs

The public documentation site for EVPanda, published at **[docs.evpanda.io](https://docs.evpanda.io)**.

Built with [Mintlify](https://mintlify.com). Pages are MDX; site configuration lives in `docs.json`.

## Structure

| Path | Contents |
| --- | --- |
| `index.mdx` | Introduction — what EVPanda is, how it works, and what issues are |
| `concepts/` | Networks and API keys, identity, what gets captured |
| `integration/` | Getting started, plus one end-to-end guide per protocol |
| `style.css` | Custom CSS, auto-included by Mintlify on every page |

## Local development

```sh
npx mint@latest dev
```

Preview at `http://localhost:3000`.

## Checks

```sh
npx mint@latest validate       # strict build validation
npx mint@latest broken-links   # internal link check
npx mint@latest a11y           # colour contrast and alt text
```

Run `validate` and `broken-links` before opening a PR.

## Publishing

Merges to `master` deploy automatically through the Mintlify GitHub app.

## Related repositories

- [`evpanda-go`](https://github.com/evpanda-labs/evpanda-go) — Go SDK, the reference implementation
- [`evpanda-node`](https://github.com/evpanda-labs/evpanda-node) — Node SDK
- [`evpanda-py`](https://github.com/evpanda-labs/evpanda-py) — Python SDK

Documentation must match shipped code. See `AGENTS.md` for the sources of truth and the house style.
