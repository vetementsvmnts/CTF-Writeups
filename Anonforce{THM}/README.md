```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   ████████╗██╗  ██╗███╗   ███╗                                ║
║   ╚══██╔══╝██║  ██║████╗ ████║                                ║
║      ██║   ███████║██╔████╔██║                                ║
║      ██║   ██╔══██║██║╚██╔╝██║                                ║
║      ██║   ██║  ██║██║ ╚═╝ ██║                                ║
║      ╚═╝   ╚═╝  ╚═╝╚═╝     ╚═╝                                ║
║                                                               ║
║   ROOM   : Anonforce                                          ║
║   TYPE   : Boot2Root / FTP Exploitation                       ║
║   DIFF   : Easy                                               ║
║   STATUS : ROOTED [✔]                                         ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

# Anonforce — TryHackMe Writeup

> **Room:** [Anonforce (bsidesgtanonforce)](https://tryhackme.com/room/bsidesgtanonforce)
> **Category:** Boot2Root — Misconfigured FTP, GPG/PGP Key Cracking, SSH Foothold, Root Escalation
> **Difficulty:** Easy
> **Author:** Kitsana Thuekoh ([`vetementsvmnts`](https://github.com/vetementsvmnts))

---

## 📖 Overview

`Anonforce` is a beginner-friendly boot2root machine built for the FIT / BSides Guatemala CTF. The path to root hinges on an **anonymous FTP misconfiguration** that exposes the entire filesystem, a **leaked PGP private key** protected by a weak passphrase, and a **PGP-encrypted backup** that ultimately yields root credentials.

**Attack chain summary:**

```
Anonymous FTP Access  →  Filesystem Enumeration  →  Exfiltrate private.asc + backup.pgp
       →  gpg2john Hash Extraction  →  John the Ripper Brute Force
       →  GPG Import + Decrypt backup.pgp  →  Recovered Credentials
       →  SSH Login  →  Privilege Escalation  →  Root Flag
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| `nmap` | Port scanning & service enumeration |
| `ftp` | Anonymous FTP interaction & file exfiltration |
| `gpg2john` | Convert PGP private key to John-crackable hash |
| `john` (John the Ripper) | Passphrase brute forcing (rockyou.txt) |
| `gpg` | Key import & PGP backup decryption |
| `ssh` | Remote shell access |

---

##  Phase 1 — Reconnaissance (Nmap Discovery)

An initial `nmap` scan against the target revealed two open ports:

- **Port 21/tcp** — `vsftpd 3.0.3` (FTP)
- **Port 22/tcp** — `OpenSSH` (SSH)

The FTP service flagged `ftp-anon: Anonymous FTP login allowed` — the first and most critical misconfiguration on the box.

![Nmap Discovery 1](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/01-Nmap-Discovery.png)

A follow-up scan confirmed service versions and ruled out additional attack surface, keeping the focus squarely on FTP.

![Nmap Discovery 2](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/02-Nmap-Discovery.png)

---

##  Phase 2 — FTP Enumeration & Exfiltration

Logging in with `anonymous:anonymous` granted full read access to the box's filesystem via FTP. Enumerating `/home` surfaced a user directory (`melodias`) containing the **user flag**, while a suspicious `notread` directory in the filesystem root contained two files of interest:

- `private.asc` — a PGP private key
- `backup.pgp` — a PGP-encrypted archive

Both files were pulled down locally using FTP's `get` command for offline analysis.

![FTP Exfiltrator](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/03-FTP-Exfiltrator.png)

---

##  Phase 3 — Key Extraction & Brute Force

`private.asc` was locked behind a passphrase, so it was converted into a crackable hash format using `gpg2john`:

```bash
gpg2john private.asc > pgp.hash
```

![Key Extract](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/04-KeyExtract.png)

The resulting hash was then brute forced with **John the Ripper** against `rockyou.txt`, recovering the passphrase in seconds:

```bash
john pgp.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

![Key Import Brute Force](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/04-KeyImport-Brute.png)

---

##  Phase 4 — PGP Decryption

With the passphrase recovered, the private key was imported into GPG and used to decrypt `backup.pgp`:

```bash
gpg --import private.asc
gpg --decrypt backup.pgp > backup.txt
```

Decrypting the backup revealed a set of credentials tied to the box's user account.

![PGP Decryptor](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/05-PGP-Decryptor.png)

---

##  Phase 5 — Foothold, Privilege Escalation & Root

The recovered credentials were used to authenticate over **SSH**, providing an initial foothold on the box. From there, privilege escalation enumeration led to root access.

![Root Access](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/06-RootAccess.png)

---

## 🏁 Flags

**Root Flag:**

![Root Flag](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/07-Root-Flag.png)

**User Flag:**

![User Flag](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/08-User-Flag.png)

**Room Completion:**

![Room Completion](https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Anonforce%7BTHM%7D/Assets/09-Room-Completion.png)

---

##  Key Takeaways

- **Anonymous FTP with world-readable filesystem access** is a critical exposure — it should never allow traversal beyond an intended upload/share directory.
- **Weak GPG passphrases** are just as crackable as weak account passwords; `gpg2john` + `john`/`hashcat` makes short work of dictionary-guessable keys.
- **Sensitive material (private keys, encrypted backups) left on an accessible service** creates a full compromise chain even without a "real" exploit — misconfiguration alone was sufficient to reach root.

---

##  MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Reconnaissance | Active Scanning | T1595 |
| Initial Access | Exploit Public-Facing Application (Anonymous FTP) | T1190 |
| Credential Access | Brute Force — Password Cracking | T1110.002 |
| Credential Access | Unsecured Credentials — Private Keys | T1552.004 |
| Lateral Movement | Remote Services — SSH | T1021.004 |
| Privilege Escalation | Valid Accounts | T1078 |

---

##  Room Link

🔗 [TryHackMe — Anonforce](https://tryhackme.com/room/bsidesgtanonforce)

---

<p align="center">
<i>Writeup by Kitsana Thuekoh — vetementsvmnts</i>
</p>
