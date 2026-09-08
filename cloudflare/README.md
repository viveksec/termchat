# TermChat Cloudflare Worker Relay

Deploy the TermChat zero-knowledge relay to Cloudflare Workers. Clients connect with the **same WebSocket JSON protocol** as the Go relay server — no client changes beyond the `-server` URL.

## Deploy

```bash
cd cloudflare
npm install
npx wrangler login          # once
npx wrangler deploy
```

After deploy, Wrangler prints your worker URL, e.g.:

```
https://termchat-relay.<your-subdomain>.workers.dev
```

Clients connect with:

```bash
export TERMCHAT_SERVER=wss://termchat-relay.<your-subdomain>.workers.dev/ws
go run ../cmd/client

# or explicitly:
go run ../cmd/client -server wss://termchat-relay.<your-subdomain>.workers.dev/ws
```

## Local dev (test before deploy)

```bash
cd cloudflare
npm install
npm run dev
# Worker runs at http://127.0.0.1:8787
```

Terminal 1 — relay:

```bash
cd cloudflare && npm run dev
```

Terminal 2 & 3 — clients:

```bash
go run ./cmd/client -server ws://127.0.0.1:8787/ws
```

Use `/connect <PEER_ID>` in each client to start an encrypted chat (same as local mode).

## Endpoints

| Path | Description |
|------|-------------|
| `/ws` | WebSocket relay (same protocol as Go server) |
| `/health` | JSON health check |
| `/` | Plain-text status page |

## Architecture

- **Worker** — routes HTTP/WebSocket to a single global Durable Object
- **Durable Object (`RelayServer`)** — holds all active WebSocket sessions, assigns 6-char IDs, routes packets (zero-knowledge, same logic as `cmd/server`)

## Remote host with Go server (alternative)

If you prefer a VPS/Docker relay instead of Cloudflare:

```bash
# On the host (open port 8080 or set PORT)
PORT=8080 PUBLIC_HOST=relay.example.com PUBLIC_TLS=true ./relay-server

# On clients
go run ./cmd/client -server wss://relay.example.com/ws
```

Or use the included `Dockerfile` / `render.yaml` for Render.com deployment.
