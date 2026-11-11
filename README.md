# cloak

**Let your agent talk to any model without telling it who you are.**

cloak is a small local proxy for OpenAI- and Anthropic-compatible APIs. It swaps your real names, emails, phone numbers and addresses for a realistic stand-in identity before a request leaves your machine, and swaps them back in the reply. Your agent, your code and you see the real details. The model provider sees only the stand-in.

> You write **"Email Maya Patel at maya@acme.io."** The model sees **"Email Alex Moreno at alex@northwind.dev."**
>
> When it replies **"Here's a draft for Alex,"** you see **"Here's a draft for Maya."**

cloak is the open core of the [Cloak](https://github.com/0307-studio) app, built for terminals, agents and servers.

## How it works

```
  your agent / SDK               cloak (localhost)                        model provider
  ┌──────────────────┐  real    ┌──────────────────────────────┐  stand-in ┌──────────────┐
  │ Claude Code,     │ ───────▶ │ 1. find your details (Alt ID) │ ───────▶ │ Anthropic,   │
  │ Codex, any SDK   │          │ 2. swap for the stand-in      │          │ OpenAI,      │
  │                  │ ◀─────── │ 3. restore them in the reply  │ ◀─────── │ OpenRouter…  │
  └──────────────────┘  real    │ 4. optional UA + proxy        │  stand-in └──────────────┘
                                └──────────────────────────────┘
```

- **Alt ID**: a local file with your real details and the stand-in for each. Matching is case-insensitive and catches common variants (first name alone, email local part, formatted and unformatted phone numbers).
- **Streaming aware**: replacements are restored inside streamed responses (SSE), even when a name is split across chunks.
- **Network cover**: optionally send a different User-Agent and route upstream traffic through an HTTP or SOCKS5 proxy.
- **Nothing leaves your machine except the cloaked request.** No account, no telemetry. Your API keys pass straight through to the provider.

## Install

```bash
go install github.com/october-dev/cloak/cmd/cloak@latest
```

## Quick start

```bash
# 1. Save your real details and their stand-ins
cloak id set name  "Maya Patel"     "Alex Moreno"
cloak id set email "maya@acme.io"   "alex@northwind.dev"
cloak id set phone "+1 415 555 0134" "+1 628 555 0199"

# 2. Preview what a provider would see
cloak check "Email Maya Patel at maya@acme.io"
# → Email Alex Moreno at alex@northwind.dev

# 3. Run any agent through cloak
cloak run -- claude
cloak run -- codex
```

`cloak run` starts the proxy on a free local port and launches the command with `ANTHROPIC_BASE_URL` and `OPENAI_BASE_URL` pointed at it. When the command exits, the proxy stops.

To run the proxy on its own:

```bash
cloak serve --port 8790
# then point any client at http://127.0.0.1:8790
```

## Commands

| Command | What it does |
| --- | --- |
| `cloak id set <kind> <real> <stand-in>` | Add or update a detail (`name`, `email`, `phone`, `address`, `birthday`, `custom`) |
| `cloak id list` | Show saved details |
| `cloak id remove <real>` | Remove a detail |
| `cloak check "<text>"` | Show the cloaked version of some text |
| `cloak run -- <command...>` | Run a command with its model traffic going through cloak |
| `cloak serve` | Run the proxy on its own |

| Option | |
| --- | --- |
| `--upstream <url>` | Where to forward requests (defaults to each provider's own API) |
| `--user-agent <ua>` | User-Agent to present upstream |
| `--proxy <url>` | Upstream `http://` or `socks5://` proxy |
| `--log` | Print each cloaked request to stderr so you can see exactly what was sent |

## With AI Hotspot

cloak and [AI Hotspot](https://github.com/october-dev/ai-hotspot) stack: point cloak's `--upstream` at your hotspot, and everyone on your network gets shared model access with their identity kept to themselves.

## What cloak can't do

cloak replaces the details you give it. It doesn't guess at details it hasn't been told about, and it can miss unusual spellings or details inside images and files. Use `--log` or `cloak check` before sending anything sensitive.

## Security

Your Alt ID is stored at `~/.config/cloak/alt-id.json` with owner-only permissions. The proxy only listens on `127.0.0.1`. To report a vulnerability, see [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE)
