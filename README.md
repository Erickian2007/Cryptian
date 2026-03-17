# 🔐 IAN-SEC

> Personal password vault — offline, terminal, no server, no cloud.

```
    ██╗ █████╗ ███╗  ██╗       ███████╗███████╗ ██████╗
    ██║██╔══██╗████╗ ██║       ██╔════╝██╔════╝██╔════╝
    ██║███████║██╔██╗██║ █████╗███████╗█████╗  ██║     
    ██║██╔══██║██║╚████║ ╚════╝╚════██║██╔══╝  ██║     
    ██║██║  ██║██║ ╚███║       ███████║███████╗╚██████╗
    ╚═╝╚═╝  ╚═╝╚══╝ ╚══╝       ╚══════╝╚══════╝ ╚═════╝
```

---

## Why offline?

Online managers like LastPass or Bitwarden are convenient, but they carry structural risks:

- In 2022, LastPass was hacked — data from millions of users was leaked
- You depend on the company continuing to exist
- You depend on the internet to access your own passwords
- You blindly trust their cryptographic implementation

IAN-SEC solves this differently: **the vault lives on your machine**. No server. No account. No external dependencies. If someone steals the `.json` file, they get nothing without your master password — mathematically.

---

## How it works

IAN-SEC uses a three-layer security chain:

```
your master password (human, memorable)
        │
        ▼
  PBKDF2-HMAC-SHA256
  600,000 iterations + random salt
        │
        ▼
  256-bit key (brute-force proof)
        │
        ▼
  AES-256-GCM + random nonce
        │
        ▼
  ciphertext (unreadable without the key)
        │
        ▼
  saved to ~/.iansec.json
```

### PBKDF2 + Salt

Your human password is inherently weak. PBKDF2 applies SHA-256 **600,000 times in a loop**, making each brute-force attempt computationally expensive — what takes you milliseconds would take an attacker years to test billions of combinations.

The salt is a randomly generated 16-byte number, unique to each entry. This ensures that two identical passwords produce completely different results — eliminating rainbow table attacks.

### AES-256-GCM

AES-256 is the standard adopted by the US government for classified data. With a 256-bit key, there are more possible combinations than atoms in the observable universe. GCM mode adds authentication — if someone alters even a single bit of the encrypted file, decryption fails immediately.

The nonce (number used once) is another randomly generated 12-byte number per operation, ensuring that encrypting the same data twice produces completely different results.

### Kerckhoffs' Principle

> *"A cryptographic system should be secure even if everything about it, except the key, is public knowledge."*

IAN-SEC's code is open. The algorithms are public. Security rests **entirely on your master password** — not on the secrecy of the system.

---

## Installation

**Dependency:**
```bash
pip install cryptography
```

**Arch Linux:**
```bash
pip install cryptography --break-system-packages
```

**Automatic clipboard (optional):**
```bash
# Arch / Manjaro
sudo pacman -S xclip

# Ubuntu / Debian
sudo apt install xclip
```

**Run:**
```bash
python iansec.py
```

---

## Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Add password** | Encrypts and saves a password with a service name |
| 2 | **Get password** | Decrypts and copies to clipboard automatically |
| 3 | **List entries** | Shows saved names (never the passwords) |
| 4 | **Delete entry** | Removes an entry (requires master password) |
| 5 | **Generate site password** | Creates a strong, deterministic password from your human password |
| 6 | **Backup** | Copies the vault to configured destinations |
| 7 | **Backup destinations** | Manages backup paths (USB drive, external HDD, cloud) |

### Deterministic password generator

Option 5 is unique: it **saves nothing**. It uses your human password + service name as PBKDF2 input and always produces the same strong password as output.

```
master password + "google"  →  Xk#9mP2qL$nR7vBw4jQs
master password + "github"  →  3Yw@8tFnZe5hCmK1sDxA
```

Same master password, completely different results per service. If one site leaks, the others remain safe. And you never need to store those passwords — you can recreate any of them on the spot.

---

## Backup

The `~/.iansec.json` file **can be copied anywhere without risk**. Without the master password, it's just encrypted garbage. You can throw it on Google Drive, Dropbox, a USB drive, email — it doesn't matter.

```bash
# manual backup
cp ~/.iansec.json /media/your-usb-drive/

# or use menu option 6 to automate it
```

Automatic backup saves with a timestamp:
```
IAN-SEC_backup_20260317_143022.json
```

---

## Security

**What is protected:**
- Each entry uses unique salt and nonce — patterns are impossible
- Key comparison uses constant time (`hmac.compare_digest`) — no timing attacks
- The file has `600` permissions — only you can read and write
- The master password is never saved anywhere

**What you need to protect:**
- Your master password — without it there is no recovery
- The `~/.iansec.json` file — it is your vault

**What doesn't matter if leaked:**
- The source code (it's right here, it's public — Kerckhoffs' principle)
- The `.json` file in isolation (useless without the master password)

---

## Structure

```
iansec.py              ← everything in one file, no extra dependencies beyond cryptography
~/.iansec.json         ← your vault (never goes to the repository)
~/.iansec_config.json  ← backup destinations (never goes to the repository)
```

---

## Philosophy

IAN-SEC was born as an exercise in learning cryptography from scratch — understanding each piece before using it. It's not just a tool, it's the materialization of concepts:

- **XOR** as the foundation of all symmetric cryptography
- **One-way functions** with no way back
- **Salt** against rainbow tables
- **Costly iterations** against brute force
- **Authentication** beyond confidentiality

If you want to understand how IAN-SEC works under the hood, the code is intentionally readable and commented.

---

## License

MIT — use, modify, distribute freely.
