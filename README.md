# TermChat

TermChat is an open-source terminal chat client with end-to-end encrypted one-to-one sessions. Clients discover one another through a relay, then exchange ephemeral X25519 public keys and encrypt chat data locally with AES-256-GCM.

The relay assigns temporary six-character IDs and forwards protocol packets. It does not receive private keys, shared secrets, or plaintext chat messages.

## Current relay

The default client relay is the deployed Cloudflare Worker:

```text
wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

Health check: <https://termchat-relay.meetkhamar3501.workers.dev/health>

## Quick start

### Use the included Linux binary

```bash
chmod +x bin/termchat-linux
./bin/termchat-linux
```

The client uses the current Cloudflare relay automatically. To select another relay:

```bash
./bin/termchat-linux -server wss://example.com/ws
```

### Build from source

Requirements: Go 1.23 or newer.

```bash
git clone https://github.com/viveksec/termchat.git
cd termchat
go run ./cmd/client
```

Or build a local binary:

```bash
mkdir -p bin
go build -o bin/termchat ./cmd/client
./bin/termchat
```

### Downloaded binaries

The `bin/` directory contains builds for:

| File | Platform |
| --- | --- |
| `termchat-linux-amd64` | Linux x86_64 |
| `termchat-linux-arm64` | Linux ARM64 |
| `termchat-macos-amd64` | macOS Intel |
| `termchat-macos-arm64` | macOS Apple Silicon |
| `termchat-windows-amd64.exe` | Windows x86_64 |
| `termchat-windows-arm64.exe` | Windows ARM64 |

Run the binary that matches the operating system and architecture. Release builds can be regenerated with:

```bash
./build-cross-platform.sh
```

## Using the client

1. Start the client on two machines.
2. Each client receives a temporary ID shown in the user list.
3. On one client, enter `/connect USER_ID`.
4. Accept the request on the other client.
5. Verify the safety number out of band with `/verify` or `Ctrl+V`.
6. Send messages after the encrypted session is established.

Commands:

| Command | Purpose |
| --- | --- |
| `/connect USER_ID` | Request a session with another online user |
| `/disconnect` or `/leave` | End the current session |
| `/sendfile PATH` | Send an encrypted file to the peer |
| `/verify` | Display the session safety number |
| `/whoami` | Display your temporary user ID |
| `/clear` | Clear chat history |
| `/panic` | Show the privacy screen |
| `/help` | Show built-in help |

Press `F1` for help and `Ctrl+C` to exit. Use `-log FILE` when troubleshooting connection problems:

```bash
./bin/termchat-linux -log /tmp/termchat.log
```

The server can also be selected with `TERMCHAT_SERVER`:

```bash
export TERMCHAT_SERVER=wss://example.com/ws
./bin/termchat-linux
```

## Run a local Go relay

The repository includes a self-hosted Go relay for local development or a server you operate yourself:

```bash
go run ./cmd/server -addr :8080
```

In another terminal:

```bash
go run ./cmd/client -server ws://localhost:8080/ws
```

The Go relay supports `PORT`, `PUBLIC_HOST`, and `PUBLIC_TLS` for deployment environments. For Docker-based deployments, see [Dockerfile](Dockerfile) and [docker-compose.yml](docker-compose.yml). The included [render.yaml](render.yaml) is an optional Render deployment configuration.

## Cloudflare Worker relay

The production relay is implemented with a Cloudflare Worker and Durable Object. Deployment and local Worker development instructions are in [cloudflare/README.md](cloudflare/README.md).

```bash
cd cloudflare
npm install
npx wrangler login
npx wrangler deploy
```

Clients connect to the deployed Worker at its `/ws` path using `wss://`.

## Development

Run all Go tests:

```bash
go test ./...
```

Run the demo:

```bash
go run ./cmd/demo
```

The protocol and cryptography packages have focused unit tests. The relay tests cover connection IDs, user-list broadcasts, routing, and an encrypted session flow.

## Repository layout

```text
cmd/client/       Terminal client and WebSocket connection manager
cmd/server/       Go WebSocket relay
cmd/demo/         Protocol and encryption demonstration
pkg/crypto/       X25519, key derivation, AES-GCM, and safety numbers
pkg/protocol/    Shared JSON packet types
cloudflare/       Cloudflare Worker relay and Durable Object
scripts/test-relay/Relay test utility
bin/              Cross-platform client and relay binaries
```

## Security notes

- Chat payloads are encrypted before they are sent to the relay.
- Each client creates an ephemeral X25519 key pair for its process session.
- AES-GCM authenticates encrypted messages and file chunks.
- The safety number should be compared through a separate trusted channel.
- The relay still sees connection metadata such as temporary IDs, timing, and packet routing fields.

TermChat is provided under the [MIT License](LICENSE).
