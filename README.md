<div align="center">

# 🔒 TermChat

### Zero-Knowledge, End-to-End Encrypted Terminal 1-on-1 Chat

[![Go Version](https://img.shields.io/badge/Go-1.23%2B-00ADD8?style=for-the-badge&logo=go)](https://go.dev)
[![Global Install](https://img.shields.io/badge/Global%20Install-1--Liner-purple?style=for-the-badge&logo=terminal)](https://github.com/viveksec/termchat)
[![Crypto](https://img.shields.io/badge/Crypto-X25519%20%2B%20AES--256--GCM-violet?style=for-the-badge&logo=letsencrypt)](pkg/crypto/crypto.go)
[![License](https://img.shields.io/badge/License-MIT-emerald?style=for-the-badge)](LICENSE)
[![Tests](https://img.shields.io/badge/Tests-100%25%20Passing-success?style=for-the-badge)](cmd/server/main_test.go)

<p align="center">
  <b>Chat securely with anyone in Surat, Mumbai, New York, or Tokyo — straight from your terminal.</b><br>
  <i>No accounts · No phone numbers · No data collection · Zero-Knowledge Relay</i>
</p>

```
  ████████╗███████╗██████╗ ███╗   ███╗ ██████╗██╗  ██╗ █████╗ ████████╗
  ╚══██╔══╝██╔════╝██╔══██╗████╗ ████║██╔════╝██║  ██║██╔══██╗╚══██╔══╝
     ██║   █████╗  ██████╔╝██╔████╔██║██║     ███████║███████║   ██║   
     ██║   ██╔══╝  ██╔══██╗██║╚██╔╝██║██║     ██╔══██║██╔══██║   ██║   
     ██║   ███████╗██║  ██║██║ ╚═╝ ██║╚██████╗██║  ██║██║  ██║   ██║   
     ╚═╝   ╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝ ╚═════╝╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝  
```

---

### ⚡ Run Anywhere in 1 Command (Zero Setup)

```bash
go run github.com/viveksec/termchat/cmd/client@latest
```

</div>

---

## ✨ Features at a Glance

<table>
  <tr>
    <td width="50%">
      <h3>🔒 Zero-Knowledge Architecture</h3>
      The central relay server routes raw JSON packets using anonymous 6-character short IDs. It never possesses private keys, shared secrets, or unencrypted message payloads.
    </td>
    <td width="50%">
      <h3>🔑 Ephemeral Perfect Forward Secrecy</h3>
      Every chat session negotiates an ephemeral <b>X25519 Curve25519</b> Diffie-Hellman key pair. Even if past keys were compromised, prior session plaintexts remain mathematically unrecoverable.
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>🛡️ SAS Safety Number Verification</h3>
      Derives a 6-digit Short Authentication String (<code>XXX-XXX</code>) per session (<code>/verify</code> or <code>Ctrl+V</code>) for out-of-band verification against active Man-in-the-Middle (MITM) attacks.
    </td>
    <td width="50%">
      <h3>🕵️ Stealth Panic Camouflage Mode</h3>
      Instant emergency hotkey (<code>Ctrl+P</code> or <code>/panic</code>) wipes session visuals and renders a realistic system shell prompt (<code>user@macbook-air:~$</code>) to protect user privacy.
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>📦 Encrypted File Sharing</h3>
      Transfer images, documents, and archives via <code>/sendfile &lt;PATH&gt;</code>. Chunks files into 32 KiB payloads, encrypts each chunk locally with AES-256-GCM, and streams with TUI progress indicators.
    </td>
    <td width="50%">
      <h3>🌐 Multi-Node Global Fallback</h3>
      Connects out-of-the-box across cities (Surat ↔ Mumbai ↔ Worldwide) with automated multi-node failover cluster support.
    </td>
  </tr>
</table>

---

## 📐 Architecture & Key Exchange Flow

```mermaid
sequenceDiagram
    autonumber
    actor Alice as 👩 Alice (Surat)
    participant Relay as 🌐 Global Relay Server
    actor Bob as 👨 Bob (Mumbai)

    Note over Alice,Bob: 1. Connection & Discovery
    Alice->>Relay: WebSocket Connect
    Relay-->>Alice: MSG_HELLO (Assigns ID: "ZURSDJ")
    Bob->>Relay: WebSocket Connect
    Relay-->>Bob: MSG_HELLO (Assigns ID: "J93XNR")

    Note over Alice,Bob: 2. Session Initiation & Acceptance
    Alice->>Relay: /connect J93XNR
    Relay->>Bob: Forward Request
    Bob-->>Relay: Accept Request (Y)
    Relay-->>Alice: Forward Acceptance

    Note over Alice,Bob: 3. Ephemeral X25519 Diffie-Hellman Key Exchange
    Alice->>Relay: MSG_KEY_EXCHANGE (Alice Public Key)
    Relay->>Bob: Forward Alice Public Key
    Bob->>Relay: MSG_KEY_EXCHANGE (Bob Public Key)
    Relay->>Alice: Forward Bob Public Key

    Note over Alice,Bob: Both Derive: SharedSecret = SHA256(X25519(PrivKey, PeerPubKey))
    Note over Alice,Bob: Both Derive: SafetyCode = SHA256(AlicePubKey || BobPubKey) [000-000]

    Note over Alice,Bob: 4. End-to-End Encrypted Messaging & File Transfer
    Alice->>Relay: MSG_CHAT / MSG_FILE_CHUNK (AES-256-GCM Ciphertext)
    Note over Relay: Server sees ONLY opaque base64 ciphertext blob!
    Relay->>Bob: Forward Packets
    Note over Bob: Decrypts payload locally using SharedSecret
```

---

## 🚀 Quick Start Guide

### Option 1: Instant One-Liner (No cloning required)

```bash
go run github.com/viveksec/termchat/cmd/client@latest
```

### Option 2: Clone & Build

```bash
# Clone the repository
git clone https://github.com/viveksec/termchat.git
cd termchat

# Build release binaries
go build -o bin/relay-server ./cmd/server
go build -o bin/termchat     ./cmd/client

# Launch local relay server (optional)
./bin/relay-server -addr :8080

# Launch client
./bin/termchat
```

---

## ⌨️ Command & Keybinding Reference

### Slash Commands

| Command | Argument | Description |
|:---|:---|:---|
| `/connect` | `<USER_ID>` | Initiate a 1-on-1 encrypted session with a peer by short ID |
| `/verify` | — | Open SAS 6-digit Safety Number verification modal |
| `/panic` | — | Toggle Stealth Panic Camouflage Mode screen |
| `/sendfile` | `<FILE_PATH>` | Chunk, encrypt (AES-256-GCM) and transfer a file to peer |
| `/disconnect` | — | Leave current chat session and securely erase session key |
| `/clear` | — | Clear current chat history viewport |
| `/whoami` | — | Display your assigned 6-character short ID |
| `/help` | — | Toggle interactive full-screen help modal |

### Keyboard Shortcuts

| Shortcut | Context | Action |
|:---|:---|:---|
| `Enter` | Message Input | Send message / execute command / confirm modal |
| `Y` / `N` | Incoming Request Modal | Accept (`Y`) or Decline (`N`) incoming request |
| `Ctrl+V` | Active Session | Verify 6-digit SAS Safety Number modal |
| `Ctrl+P` | Global | Toggle Stealth Panic Camouflage Mode screen |
| `Ctrl+D` | Active Chat | End current encrypted session |
| `Ctrl+C` | Global | Clean exit & restore terminal state |
| `F1` / `Esc` | Global | Toggle Help overlay / dismiss active modal |
| `PgUp` / `PgDn` | Chat Viewport | Scroll chat history up or down |

---

## 🔒 Security & Threat Model

| Threat Scenario | Risk Level | TermChat Protection |
|:---|:---:|:---|
| **Eavesdropping Relay** | 🔴 High | All messages encrypted with AES-256-GCM before leaving client |
| **Active Relay MITM** | 🟠 Medium | SAS Safety Number Verification (`/verify` or `Ctrl+V`) out-of-band verification |
| **Packet Tampering** | 🔴 High | GCM 128-bit authentication tag verification rejects altered payloads |
| **Replay Attack** | 🟡 Low | Fresh random 96-bit nonces per packet + UTC timestamp validation |
| **Subgroup Attacks** | 🟠 Medium | RFC 7748 Curve25519 scalar clamping + zero-point verification |

---

## 🛠️ Project Structure

```
termchat/
├── cmd/
│   ├── client/
│   │   ├── main.go          # Client bootstrapper, WebSocket loop & crypto wiring
│   │   └── ui.go            # Bubbletea TUI model, views, modals & keybinds
│   ├── demo/
│   │   └── main.go          # Live end-to-end trace & demonstration script
│   └── server/
│       ├── main.go          # Zero-knowledge relay server
│       └── main_test.go     # Relay integration test suite
├── pkg/
│   ├── crypto/
│   │   ├── crypto.go        # X25519 DH, SHA-256 KDF, AES-GCM & SAS Safety Code
│   │   └── crypto_test.go   # Crypto unit test suite
│   └── protocol/
│       ├── protocol.go      # Protocol envelope & JSON payload structs
│       └── protocol_test.go # Protocol unit test suite
├── Dockerfile
├── docker-compose.yml
├── render.yaml
├── fly.toml
├── go.mod
├── go.sum
└── README.md
```

---

## 🧪 Testing & Verification

```bash
# Run unit & integration test suite across all packages
go test -v ./...

# Run live trace demonstration
go run ./cmd/demo/main.go
```

---

## 📄 License

Distributed under the [MIT License](LICENSE).
