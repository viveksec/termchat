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
      <h3>🔑 Perfect Forward Secrecy</h3>
      Every chat session negotiates an ephemeral <b>X25519 Curve25519</b> Diffie-Hellman key pair. Even if past keys were compromised, prior session plaintexts remain mathematically unrecoverable.
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>🛡️ AES-256-GCM Encryption</h3>
      All message payloads are authenticated and encrypted locally using 256-bit symmetric keys derived via SHA-256 KDF with fresh 96-bit random nonces.
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

    Note over Alice,Bob: 4. End-to-End Encrypted Messaging
    Alice->>Relay: MSG_CHAT (AES-256-GCM Ciphertext)
    Note over Relay: Server sees ONLY opaque base64 ciphertext blob!
    Relay->>Bob: Forward MSG_CHAT
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
| `/disconnect` | — | Leave current chat session and securely erase session key |
| `/clear` | — | Clear current chat history viewport |
| `/whoami` | — | Display your assigned 6-character short ID |
| `/help` | — | Toggle interactive full-screen help modal |

### Keyboard Shortcuts

| Shortcut | Context | Action |
|:---|:---|:---|
| `Enter` | Message Input | Send message / execute command / confirm modal |
| `Y` / `N` | Incoming Request Modal | Accept (`Y`) or Decline (`N`) incoming request |
| `F1` / `Esc` | Global | Toggle Help overlay / dismiss modal |
| `Ctrl+D` | Active Chat | End current encrypted session |
| `Ctrl+C` | Global | Clean exit & restore terminal state |
| `PgUp` / `PgDn` | Chat Viewport | Scroll chat history up or down |

---

## 🔒 Security & Threat Model

| Threat Scenario | Risk Level | TermChat Protection |
|:---|:---:|:---|
| **Eavesdropping Relay** | 🔴 High | All messages encrypted with AES-256-GCM before leaving client |
| **Active Relay MITM** | 🟠 Medium | Ephemeral DH exchange — relay cannot derive symmetric key without private keys |
| **Packet Tampering** | 🔴 High | GCM 128-bit authentication tag verification rejects altered payloads |
| **Replay Attack** | 🟡 Low | Fresh random 96-bit nonces per packet + UTC timestamp validation |
| **Subgroup Attacks** | 🟠 Medium | RFC 7748 Curve25519 scalar clamping + zero-point verification |

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
