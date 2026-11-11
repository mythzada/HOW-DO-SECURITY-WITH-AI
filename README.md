# HOW DO SECURITY WITH AI

🇺🇸 **English** · 🇧🇷 [Português](README.pt-BR.md)

> How to build, run and protect a real system using AI as your team — from the code to the network edge.

This repository is the field notebook of a real project: a Brazilian legal SaaS built and secured with AI agents, Cloudflare and a lot of offensive and defensive security study. Everything here was done in practice, tested and verified.

**Status:** open source, continuously growing. Every lesson studied and every defense applied becomes a new guide here.

---

## ⚖️ First: ethics and law

- Test **only** what you own or have **written authorization** to test.
- Unauthorized access is a crime almost everywhere (in Brazil: **art. 154-A of the Penal Code**; personal data under **LGPD**; elsewhere CFAA, Computer Misuse Act, GDPR…).
- Active tools (Nmap, Nikto, sqlmap, gobuster, hydra, pentest agents) run **only against a local copy** or a lab. In production, passive reconnaissance only.
- Found a flaw in someone else's system? Look for their `security.txt` and report it responsibly.

---

## 🗺️ Repository map

| English | Português | What's inside |
|---|---|---|
| [EN-01-DEFENSE-LAYERS.md](EN-01-DEFENSE-LAYERS.md) | [01-CAMADAS-DE-DEFESA.md](01-CAMADAS-DE-DEFESA.md) | Defense in depth: DNS, edge (Cloudflare), server, application, data |
| [EN-02-PASSIVE-RECON.md](EN-02-PASSIVE-RECON.md) | [02-RECONHECIMENTO-PASSIVO.md](02-RECONHECIMENTO-PASSIVO.md) | Commands to audit your own domain the way an attacker would see it |
| [EN-03-AI-AGENTS-METHODOLOGY.md](EN-03-AI-AGENTS-METHODOLOGY.md) | [03-METODOLOGIA-AGENTES-IA.md](03-METODOLOGIA-AGENTES-IA.md) | Coordinating several AI agents (code, security, front-end, explainer) without chaos |
| [EN-04-AI-IN-THE-BROWSER.md](EN-04-AI-IN-THE-BROWSER.md) | [04-NAVEGADOR-COM-IA.md](04-NAVEGADOR-COM-IA.md) | Using an AI Chrome extension for manual panel work, with safety rules |
| [EN-05-STUDY-PATH.md](EN-05-STUDY-PATH.md) | [05-TRILHA-DE-ESTUDO.md](05-TRILHA-DE-ESTUDO.md) | What to study, in which order, and where |
| [EN-CHECKLIST.md](EN-CHECKLIST.md) | [CHECKLIST.md](CHECKLIST.md) | Short checklist for any project |

---

## 🧠 The mindset

1. **Look at your system through an attacker's eyes.** Run the same tools an attacker would, against your own domain.
2. **Defense in layers.** No single protection is enough. If one fails, the next one holds.
3. **AI accelerates, humans decide.** Agents write, test and audit; decisions about risk, money and data stay with the owner.
4. **Verify everything.** Nothing reaches production without pasted local test results, a security review and a clear verdict.
5. **Every mistake becomes a rule.** Agent (and human) errors become numbered lessons that never repeat.
6. **Low cost is a design decision.** Local tests, manual pipelines and a well-configured free plan go a long way.

---

## 🧰 Tools used

| Front | Tools |
|---|---|
| Code and review | Claude Code, specialized Grok agents, GitHub (PRs, review) |
| Coordination | A coordinator agent that assigns, verifies and merges |
| Edge / network | Cloudflare (DNS, WAF, rules, rate limiting, HSTS, TLS, DNSSEC, CAA, DMARC) |
| Server | DigitalOcean (cloud firewall) |
| Pentest / study | Kali Linux on VirtualBox, `dig`, `whois`, `curl`, `openssl`, Google dorks |
| AI testing | Strix (open-source AI agents for security testing) — **only in an authorized local environment** |
| Audit | Cloudflare's security audit skill running in an AI agent |
| Browser | Claude in Chrome extension to operate dashboards (human in control) |

---

## 🚀 Install the skills in your Claude Code

Paste into your project terminal:

```
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill secure-ai-methodology
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill passive-domain-audit
```

Prefer Portuguese answers? Use the `-pt` versions:

```
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill metodologia-seguranca-com-ia
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill auditoria-passiva-dominio
```

| Skill (EN / PT) | What it does |
|---|---|
| **secure-ai-methodology** / metodologia-seguranca-com-ia | Makes Claude Code work with you the way this guide does: teaches while doing, defense in layers, every change with tests, security review and rollback — no unverified "vibe code" |
| **passive-domain-audit** / auditoria-passiva-dominio | Audits your domain the way an attacker would in recon (DNS, email, TLS, headers, subdomains, exposed files), passive commands only, explaining each finding |

---

## ⭐ Recommendations

- **Claude Code** — terminal coding agent; use it with the skills above.
- **Claude in Chrome** (official extension) — lets the AI operate dashboards (Cloudflare, registrar, GitHub) while you watch. Rules in [EN-04-AI-IN-THE-BROWSER.md](EN-04-AI-IN-THE-BROWSER.md).
- **Cloudflare (free plan)** — DNS, WAF, HSTS, DNSSEC, CAA, DMARC.
- **Cloudflare security audit skill** — `npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit`
- **Kali Linux on VirtualBox** — a study lab.
- **An intro hacking & pentest course** (we used Solyd's, in Portuguese).

---

## 🛣️ Next guides

- [ ] Authenticated Origin Pulls (mTLS between CDN and server)
- [ ] Cloudflare Access / Zero Trust for admin areas
- [ ] Local lab: Nmap, Netcat and pentest agents against a copy of your system
- [ ] Monitoring and incident response
- [ ] More skills

Contributions welcome: open an issue or PR.

---

MIT License. Made with study, coffee and AI in Brazil. 🇧🇷
