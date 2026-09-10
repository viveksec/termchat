# 🎯 TermChat - Complete Installation & Usage Guide

## Overview

Your TermChat application is now available as **cross-platform standalone executables**. No source code or build tools needed—just download and run!

## 📥 Download & Extract

### Option 1: Download the ZIP Package
- **Location**: `/tmp/termchat-v1.2.0-cross-platform.zip`
- **Size**: ~15 MB (compressed)
- **Extract**: All binaries and documentation

```bash
unzip termchat-v1.2.0-cross-platform.zip
cd termchat-v1.2.0-cross-platform
```

### Option 2: Use Files from Dist Directory
Files are ready in:
```
/home/common/Documents/terminal project/terminal project/dist/
```

## 🖥️ Platform-Specific Instructions

### Linux

**1. Make executable (if not already)**
```bash
chmod +x termchat-linux-amd64  # For Intel/AMD
chmod +x termchat-linux-arm64  # For ARM (Raspberry Pi, etc.)
chmod +x termchat.sh            # Launcher script
```

**2. Run using launcher (Recommended)**
```bash
./termchat.sh wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**3. Or run directly**
```bash
./termchat-linux-amd64 -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**4. Save to PATH for system-wide access**
```bash
sudo cp termchat-linux-amd64 /usr/local/bin/termchat
chmod +x /usr/local/bin/termchat
termchat -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**5. Desktop shortcut (Linux)**
Create `~/.local/share/applications/termchat.desktop`:
```ini
[Desktop Entry]
Type=Application
Name=TermChat
Comment=Encrypted Terminal Chat
Exec=/path/to/termchat-linux-amd64 -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
Icon=text-editor
Terminal=true
Categories=Utility;
```

---

### macOS

**1. First run - grant permission**
```bash
chmod +x termchat-macos-amd64  # Intel
chmod +x termchat-macos-arm64  # Apple Silicon
chmod +x termchat.sh
```

**2. If you get a security warning:**
- Right-click the app
- Select "Open" → "Open" again
- Or run: `xattr -d com.apple.quarantine ./termchat-macos-amd64`

**3. Run using launcher**
```bash
./termchat.sh wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**4. Or run directly**
```bash
# Intel Mac
./termchat-macos-amd64 -server wss://termchat-relay.meetkhamar3501.workers.dev/ws

# Apple Silicon (M1/M2/M3)
./termchat-macos-arm64 -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**5. Install to /usr/local/bin**
```bash
sudo cp termchat-macos-amd64 /usr/local/bin/termchat
chmod +x /usr/local/bin/termchat
termchat -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**6. Create an App (using Automator)**
- Open Automator → New → Application
- Run Shell Script: `/path/to/termchat-macos-amd64 -server wss://termchat-relay.meetkhamar3501.workers.dev/ws`
- Save as "TermChat.app"

---

### Windows

**1. Using the launcher (Easiest)**
Simply **double-click** `termchat.bat` and enter the server URL when prompted.

**2. Command line usage**
```cmd
termchat-windows-amd64.exe -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

**3. Create a shortcut**
- Right-click → New → Shortcut
- Target: `cmd.exe /c termchat.bat`
- Name: TermChat
- Click the shortcut to run

**4. Create a simple launcher script**
Save as `run-termchat.bat`:
```batch
@echo off
cd /d "%~dp0"
termchat-windows-amd64.exe -server wss://termchat-relay.meetkhamar3501.workers.dev/ws
pause
```

**5. Add to Windows Terminal (Windows 11)**
Edit `settings.json`:
```json
{
  "profiles": {
    "defaults": {},
    "list": [
      {
        "name": "TermChat",
        "commandline": "C:\\path\\to\\termchat-windows-amd64.exe -server wss://termchat-relay.meetkhamar3501.workers.dev/ws",
        "icon": "⌨️"
      }
    ]
  }
}
```

**6. Set server as environment variable**
```cmd
setx TERMCHAT_SERVER "wss://termchat-relay.meetkhamar3501.workers.dev/ws"
```
Then just run `termchat-windows-amd64.exe`

---

## 🔧 Advanced Usage

### Environment Variables

```bash
# Set default server (all platforms)
export TERMCHAT_SERVER="wss://termchat-relay.meetkhamar3501.workers.dev/ws"

# Enable logging
export TERMCHAT_LOG_FILE="/tmp/termchat.log"
```

### Command-line Options

```bash
-server URL    WebSocket server URL (required unless TERMCHAT_SERVER is set)
-log FILE      Write debug logs to FILE
```

### Examples

```bash
# Connect with logging
./termchat.sh wss://... -log debug.log

# Connect to local server
./termchat.sh ws://localhost:8080/ws

# Use saved environment variable
export TERMCHAT_SERVER="wss://termchat-relay.meetkhamar3501.workers.dev/ws"
./termchat.sh
```

---

## 🎮 Using TermChat

### Interface

```
┌─────────────────────────────────────────┐
│ Online Users │          Chat History     │
│              │                          │
│ • Alice      │ 14:23 Alice: Hey!        │
│ • Bob        │ 14:24 You: Hi there!    │
│ • Charlie    │ 14:25 Charlie: Hello    │
│              │                          │
├─────────────────────────────────────────┤
│ You: Type your message here...          │
└─────────────────────────────────────────┘
```

### Keyboard Shortcuts

| Key | Function |
|-----|----------|
| `Enter` | Send message |
| `Tab` | Switch panels |
| `Page Up/Down` | Scroll chat |
| `F1` | Show help |
| `Ctrl+V` | Verify security (SAS) |
| `Ctrl+P` | Panic/stealth mode |
| `Ctrl+C` | Exit |

### Security Features

✅ **SAS Verification**: Press `Ctrl+V` to verify you're chatting with the right person
✅ **Panic Mode**: Press `Ctrl+P` for emergency screen
✅ **Encryption**: All messages are end-to-end encrypted
✅ **Zero-Knowledge**: Server cannot see plaintext

---

## 📦 Sharing with Others

### Method 1: Share the ZIP
```bash
# Create ZIP (if not already done)
cd /tmp
zip -r termchat-v1.2.0-cross-platform.zip termchat-v1.2.0-cross-platform/

# Share via email, cloud storage, USB, etc.
```

### Method 2: Create Installation Script
Create `install.sh`:
```bash
#!/bin/bash
INSTALL_DIR="$HOME/.local/bin"
mkdir -p "$INSTALL_DIR"
cp termchat-linux-amd64 "$INSTALL_DIR/termchat"
chmod +x "$INSTALL_DIR/termchat"
echo "Installed to $INSTALL_DIR/termchat"
export TERMCHAT_SERVER="wss://termchat-relay.meetkhamar3501.workers.dev/ws"
termchat
```

### Method 3: Docker Container
Users can run in Docker:
```bash
docker run -it termchat termchat-linux-amd64 -server wss://...
```

---

## ✅ Verification

### Check Binary Integrity
```bash
# Verify SHA256 checksums
sha256sum -c SHA256SUMS

# Or manually verify one file
sha256sum termchat-linux-amd64
```

### Test Run
```bash
# Test with help
./termchat.sh

# Test connection (without server, shows connection attempt)
./termchat.sh wss://termchat-relay.meetkhamar3501.workers.dev/ws
```

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| "Permission denied" (Linux/Mac) | `chmod +x termchat*` |
| "Binary not found" | Check file extension matches OS |
| "Connection refused" | Verify server URL and internet |
| "Cannot execute on this platform" | Download correct binary for your OS |
| Windows security warning | Click "More info" → "Run anyway" |
| Can't type characters | Check terminal UTF-8 encoding |

### Debug Mode
```bash
./termchat.sh wss://... -log /tmp/debug.log
tail -f /tmp/debug.log
```

---

## 📊 File Descriptions

| File | Size | Purpose |
|------|------|---------|
| termchat-linux-amd64 | 6.1 MB | Linux Intel/AMD 64-bit |
| termchat-linux-arm64 | 5.8 MB | Linux ARM 64-bit |
| termchat-macos-amd64 | 6.2 MB | macOS Intel |
| termchat-macos-arm64 | 5.9 MB | macOS Apple Silicon |
| termchat-windows-amd64.exe | 6.2 MB | Windows Intel/AMD |
| termchat-windows-arm64.exe | 5.8 MB | Windows ARM |
| termchat.sh | 2.2 KB | Linux/macOS launcher |
| termchat.bat | 2.0 KB | Windows launcher |
| README.md | 4.9 KB | Full documentation |
| QUICKSTART.md | 2.0 KB | Quick start guide |

---

## 🔐 Security Notes

- **Binaries are statically compiled** - no external dependencies needed
- **End-to-end encryption** - messages encrypted client-side before sending
- **Zero-knowledge architecture** - relay server cannot see plaintext
- **Ephemeral keys** - new key pair generated for each session
- **Open source** - you can verify the code in the repository

---

## 📝 Next Steps

1. **Extract the ZIP**: `unzip termchat-v1.2.0-cross-platform.zip`
2. **Run launcher**: `./termchat.sh wss://termchat-relay.meetkhamar3501.workers.dev/ws`
3. **Invite friends**: Share the server URL with others
4. **Verify security**: Press `Ctrl+V` to see SAS codes
5. **Enjoy encrypted chat!** 🚀

---

**Version**: v1.2.0  
**Built**: 2026-08-16  
**Platforms**: Linux, macOS, Windows (all architectures)  
**Status**: Ready for distribution
