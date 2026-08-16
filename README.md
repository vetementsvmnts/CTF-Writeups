<div align="center">

```
██████╗████████╗███████╗
██╔════╝╚══██╔══╝██╔════╝
██║        ██║   █████╗  
██║        ██║   ██╔══╝  
╚██████╗   ██║   ██║     
 ╚═════╝   ╚═╝   ╚═╝     
     W R I T E U P S
```

**A structured archive of Hack The Box, TryHackMe, and VulnHub engagements**

[![HTB](https://img.shields.io/badge/Hack%20The%20Box-9FEF00?style=for-the-badge&logo=hackthebox&logoColor=black)](https://www.hackthebox.com/)
[![THM](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=red)](https://tryhackme.com/)
[![VulnHub](https://img.shields.io/badge/VulnHub-2E7D32?style=for-the-badge&logo=linux&logoColor=white)](https://www.vulnhub.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-00FF41?style=for-the-badge)](LICENSE)

</div>

---

## `~$ whoami`

Kitsana Thuekoh — Penetration Tester / Offensive Security Researcher based in Johannesburg, South Africa.
This repository documents box walkthroughs and CTF-style engagements completed as part of ongoing offensive
security practice, certification prep, and portfolio development.

Each writeup follows a consistent methodology: **reconnaissance → enumeration → exploitation → privilege
escalation → post-exploitation**, mapped where relevant to the MITRE ATT&CK framework.

---

## `~$ ls writeups/`

| # | Box / Room | Platform | OS | Difficulty | Techniques | Writeup |
|---|------------|----------|----|------------|------------|---------|
| 01 | Tech_Supp0rt: 1 | TryHackMe | Linux | Easy | SMB Enum, FTP, Password Reuse, Priv Esc | [`→ view`](./Tech_Supp0rt:%201%7BTHM%7D) |

> Table updates as new writeups are added. Each row links to a self-contained folder with its own README,
> including attack-chain diagrams, MITRE ATT&CK mapping, and supporting evidence.

---

## `~$ cat structure.md`

```
CTF-Writeups/
├── <Box-Name>: <Number>{PLATFORM}/
│   ├── README.md          # Full writeup: recon → exploitation → priv esc
│   ├── images/            # Screenshots / evidence
│   └── scripts/           # Any exploit or automation scripts used
├── README.md               # This file — master index
└── LICENSE
```

Naming convention: `<Box Name>: <Number>{PLATFORM}` — e.g. `Tech_Supp0rt: 1{THM}`, `Breakout: 1{HTB}`,
`Empire: Breakout{VH}`.

---

## `~$ cat methodology.md`

Every writeup in this repository follows the same structure for consistency and readability:

- **Reconnaissance** — Nmap scans, service enumeration, initial footprinting
- **Enumeration** — Directory brute-forcing, SMB/FTP/web enumeration, credential hunting
- **Exploitation** — Vulnerability identification and exploit development/execution
- **Privilege Escalation** — Local enumeration, misconfiguration abuse, kernel/service exploits
- **Post-Exploitation** — Flag capture, persistence notes, lessons learned
- **MITRE ATT&CK Mapping** — Techniques used mapped to tactic IDs where applicable
- **Blue Team Notes** — Detection opportunities and defensive recommendations

---

## `~$ cat platforms.md`

| Platform | Focus |
|----------|-------|
| **Hack The Box** | Realistic enterprise-style networks, Active Directory, web app chains |
| **TryHackMe** | Guided and CTF-style rooms across web, network, and Linux/Windows fundamentals |
| **VulnHub** | Self-hosted vulnerable VMs for offline practice and CTF-style challenges |

---

## `~$ cat certifications.md`

`OSCP` · `HTB CPTS` · `CompTIA PenTest+` · `CompTIA Security+` · `ISC2 CC` · `NASA VDP Letter of Recognition`

---

## `~$ contact --info`

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](#)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/vetementsvmnts)

---

<div align="center">

*Findings are documented for educational and portfolio purposes only. No exploitation was performed against
systems without explicit authorization.*

</div>
