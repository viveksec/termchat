# TermChat — End-to-End Encrypted Terminal Chat

A production-ready, 1-on-1 encrypted chat application running entirely in the terminal. Built in Go with zero runtime dependencies beyond the Go standard library and a handful of well-audited packages.

```
┌────────────────────────────────────────────────────────────────┐
│  🔒 X25519 key exchange  ·  AES-256-GCM messages              │
│  Zero-knowledge relay   ·  Ephemeral session keys              │
│  Beautiful TUI          ·  Auto-reconnect                      │
└────────────────────────────────────────────────────────────────┘
```

---

## How it Works

```
Alice (cli-client)          Relay Server           Bob (cli-client)
     │                          │                       │
     │── WebSocket connect ─────▶│                       │
     │◀─ HELLO (ID: "AB3K9Z") ──│                       │
     │                          │◀── WebSocket connect ──│
     │                          │─── HELLO (ID: "PQ7R2X") ──▶│
     │                          │                       │
     │── /connect PQ7R2X ───────▶│── CONNECT_REQUEST ────▶│
     │                          │                       │ [y/n prompt]
     │                          │◀── CONNECT_RESPONSE ───│ (accepted)
     │◀─ accepted ──────────────│                       │
     │                          │                       │
     │── KEY_EXCHANGE (pub_A) ──▶│── KEY_EXCHANGE ────────▶│
     │◀─ KEY_EXCHANGE (pub_B) ──│◀── KEY_EXCHANGE ────────│
     │                          │                       │
     │  [derive shared secret]  │              [derive shared secret]
     │  AES key = SHA256(X25519(priv_A, pub_B))         │
     │                          │                       │
     │── CHAT (AES-GCM blob) ───▶│── CHAT (same blob) ────▶│  [decrypt]
     │                          │  (server sees only    │
     │                          │   opaque ciphertext)  │
```

**The relay server is zero-knowledge** — it only reads the `type` and `target_id` fields of each packet to route it. It never sees private keys, shared secrets, or plaintext messages.

---

## Repository Layout

```
.
├── go.mod
├── go.sum
├── README.md
├── pkg/
│   ├── crypto/
│   │   └── crypto.go          # X25519 keygen, DH derivation, AES-256-GCM
│   └── protocol/
│       └── protocol.go        # Wire protocol structs & JSON serialisation
└── cmd/
    ├── server/
    │   └── main.go            # WebSocket relay server
    └── client/
        ├── main.go            # Client bootstrapper & WebSocket event loop
        └── ui.go              # Bubbletea TUI model
```

---

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Go   | ≥ 1.21  | https://go.dev/dl/ |
| Git  | any     | https://git-scm.com/ |

---

## Build Instructions

### Bash (Linux / macOS)

```bash
# Clone and enter the project
git clone <your-repo-url> termchat
cd termchat

# Download dependencies
go mod tidy

# Build both binaries
go build -o bin/relay-server ./cmd/server
go build -o bin/termchat     ./cmd/client

# Or build for a specific platform (cross-compile)
GOOS=linux  GOARCH=amd64 go build -o bin/relay-server-linux   ./cmd/server
GOOS=darwin GOARCH=arm64 go build -o bin/relay-server-darwin  ./cmd/server
GOOS=windows GOARCH=amd64 go build -o bin/relay-server.exe    ./cmd/server
```

### PowerShell (Windows)

```powershell
# Clone and enter the project
git clone <your-repo-url> termchat
cd termchat

# Download dependencies
go mod tidy

# Build both binaries
go build -o bin\relay-server.exe .\cmd\server
go build -o bin\termchat.exe     .\cmd\client

# Cross-compile for Linux from Windows
$env:GOOS = "linux"
$env:GOARCH = "amd64"
go build -o bin\relay-server-linux .\cmd\server
go build -o bin\termchat-linux     .\cmd\client
Remove-Item Env:\GOOS; Remove-Item Env:\GOARCH
```

---

## Running Locally

### Step 1 — Start the relay server

```bash
# Default: listens on :8080
./bin/relay-server

# Custom port
./bin/relay-server -addr :9000

# With verbose logging
./bin/relay-server -addr :8080 2>&1 | tee server.log
```

Expected output:
```
[relay] TermChat relay server v1.0.0 listening on [::]:8080
[relay] WebSocket endpoint: ws://[::]:8080/ws
```

### Step 2 — Start two clients (in separate terminal windows)

**Terminal 1 (Alice):**
```bash
./bin/termchat
# or specify server:
./bin/termchat -server ws://localhost:8080/ws
```

**Terminal 2 (Bob):**
```bash
./bin/termchat -server ws://localhost:8080/ws
```

### Step 3 — Start chatting

1. Note your assigned 6-character ID shown in the status bar (e.g., `AB3K9Z`).
2. In Alice's terminal, type: `/connect <BOB_ID>` and press Enter.
3. In Bob's terminal, press `Y` + Enter to accept.
4. Both terminals perform an automatic X25519 key exchange.
5. You're now in an AES-256-GCM encrypted session. Type messages and press Enter.

### Keyboard Reference

| Key | Action |
|-----|--------|
| `Enter` | Send message / confirm action |
| `F1` | Toggle help overlay |
| `Esc` | Close help / decline request |
| `Ctrl+D` | Leave current session |
| `Ctrl+C` | Quit TermChat |
| `PgUp / PgDn` | Scroll chat history |

### Commands

| Command | Description |
|---------|-------------|
| `/connect <ID>` | Initiate a chat session with a user |
| `/disconnect` | End the current session |
| `/clear` | Clear the chat history |
| `/whoami` | Display your assigned short ID |
| `/help` | Show the help overlay |

---

## Cloud Deployment

### Option A — fly.io (Recommended)

```bash
# Install flyctl
curl -L https://fly.io/install.sh | sh

# Authenticate
fly auth login

# Create the app (from the project root)
fly launch --name termchat-relay --region ord --no-deploy

# Set the Dockerfile (or use Go buildpacks)
# fly.io auto-detects Go projects. Point it at cmd/server:
```

Create `fly.toml`:
```toml
app = "termchat-relay"
primary_region = "ord"

[build]
  [build.args]
    GO_BUILD_TARGET = "./cmd/server"

[[services]]
  protocol = "tcp"
  internal_port = 8080

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

  [[services.ports]]
    port = 80
    handlers = ["http"]

  [services.concurrency]
    type = "connections"
    hard_limit = 1000
    soft_limit = 800
```

```bash
fly deploy
# Your relay is now at: wss://termchat-relay.fly.dev/ws
```

Connect clients to your cloud relay:
```bash
./bin/termchat -server wss://termchat-relay.fly.dev/ws
```

### Option B — Bare VPS (Ubuntu/Debian)

```bash
# On your server
go build -o /usr/local/bin/relay-server ./cmd/server

# Create a systemd service
cat > /etc/systemd/system/termchat-relay.service <<EOF
[Unit]
Description=TermChat Relay Server
After=network.target

[Service]
ExecStart=/usr/local/bin/relay-server -addr :8080
Restart=always
RestartSec=5
User=nobody

[Install]
WantedBy=multi-user.target
EOF

systemctl enable --now termchat-relay
```

### Option C — Docker

Create `Dockerfile` in the project root:
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o relay-server ./cmd/server

FROM scratch
COPY --from=builder /app/relay-server /relay-server
EXPOSE 8080
ENTRYPOINT ["/relay-server", "-addr", ":8080"]
```

```bash
docker build -t termchat-relay .
docker run -p 8080:8080 termchat-relay

# With docker-compose
docker-compose up -d
```

---

## Security Architecture

### Cryptographic Properties

| Property | Implementation |
|----------|---------------|
| **Confidentiality** | AES-256-GCM — 256-bit symmetric encryption |
| **Integrity** | GCM authentication tag — detects any tampering |
| **Authenticity** | Shared secret only derivable by both DH parties |
| **Forward Secrecy** | Fresh ephemeral X25519 key pair per client launch |
| **Nonce uniqueness** | 96-bit random nonce generated per message |
| **Key clamping** | Private scalar clamped per RFC 7748 |
| **Low-order protection** | X25519 output checked for all-zero (invalid DH) |

### What the relay server sees

```json
{
  "type": "CHAT",
  "sender_id": "AB3K9Z",
  "target_id": "PQ7R2X",
  "payload": {
    "ciphertext": "r4nD0mB4s364EncOdEdCiph3rT3xtTh4tM34nsN0thInGt0TheS3rv3r..."
  },
  "timestamp": "2026-08-13T07:00:00Z"
}
```

The server never sees:
- Private keys (generated locally, never transmitted)
- Shared secrets (derived locally via Diffie-Hellman)
- Plaintext messages (encrypted before leaving the client)

### Threat Model

| Threat | Mitigation |
|--------|-----------|
| Passive relay eavesdropping | AES-256-GCM encryption |
| Active relay MITM | X25519 DH — relay cannot forge keys |
| Replay attacks | GCM nonce + timestamp |
| Message tampering | GCM authentication tag verification |
| Key reuse across sessions | Ephemeral key pair per client launch |

> **Note**: For production use, consider adding out-of-band key verification (fingerprint display) to protect against an active MITM who controls the relay. This implementation trusts the relay to forward public keys faithfully.

---

## Development

```bash
# Run tests
go test ./...

# Static analysis
go vet ./...

# Run server with hot-reload (requires air)
air --build.cmd "go build -o bin/relay-server ./cmd/server" --build.bin "bin/relay-server"

# Build all platforms
make build-all   # if Makefile is added
```

---

## License

MIT License — See LICENSE file for details.
