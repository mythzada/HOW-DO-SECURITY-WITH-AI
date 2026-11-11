# 08 — From attack to defense (path under construction)

Every attack topic we study becomes a guide on **how to defend**. Topics follow the order of an introductory hacking & pentest course (we study with Solyd Offensive Security's *Introdução ao Hacking e Pentest 3.0*, in Portuguese — recommended). Offensive labs stay in the course; here we keep the applied defense.

| # | Attack topic | How we defend | Status |
|---|---|---|---|
| 1 | Career and ethics | Scope, written authorization, law, `security.txt` | ✅ README |
| 2 | Linux and terminal | Least permissions (no `777`), unprivileged users, updates | 🔜 guide |
| 3 | Recon | Shrink what shows: hidden IP, clean DNS, dedicated whois, robots without secrets, headers without versions | ✅ [EN-02](EN-02-PASSIVE-RECON.md) |
| 4 | Scanning and services (Nmap, Netcat) | Firewall with only needed ports, origin accepts only the CDN, restricted SSH, no service banners | 🔜 guide + local lab |
| 5 | Service exploits | Updated dependencies, minimal images, nothing exposed beyond need | 🔜 guide |
| 6 | SQL injection | Parameterized queries/ORM, never concatenate SQL, least DB privilege | 🔜 guide |
| 7 | XSS and cookie theft | CSP without inline/eval, no `innerHTML`, HttpOnly/Secure/SameSite cookies | ✅ partly in [EN-06](EN-06-SECURITY-METHODS.md) · 🔜 guide |
| 8 | LFI / file reads | Never use user-supplied paths; file allow-list; `.env`/`.git` blocked at the edge | 🔜 guide |
| 9 | Shell upload | Validate type by content, rename, store outside web root, AV scan, no execution | 🔜 guide |
| 10 | Privilege escalation and post-exploitation | Non-root processes, secrets off the app disk, logs and alerts | 🔜 guide |
| 11 | Network pivoting | Private network, firewall between services, no public DB | 🔜 guide |
| 12 | IDOR (bug bounty) | Every query filtered by the session's account; two-account tests | ✅ partly in [EN-06](EN-06-SECURITY-METHODS.md) · 🔜 guide |
| 13 | SSRF | Destination allow-list, block internal/metadata IPs, no blind redirect following | 🔜 guide |
| 14 | Account takeover | Passkeys, 2FA, rate limits, safe recovery, email notices | ✅ [EN-06](EN-06-SECURITY-METHODS.md) |

**Lab rule:** all offensive practice runs on a **local copy** of your system or the course labs — never in production.
