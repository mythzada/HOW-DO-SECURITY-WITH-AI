# 07 — AI security tools

Three tools that complement each other. **Use only on what you own or have written authorization for.**

| Tool | What it does | Where it runs |
|---|---|---|
| **Cloudflare audit skill** (`security-audit`) | Reads the **code** and hunts flaws in phases, independently verifying each finding | In your repo, via your coding agent |
| **Strix** | AI agents that **run** the system and try to exploit flaws, with proof of concept | **Local copy only** or your own test environment |
| **Our skills** (`passive-domain-audit` / `secure-ai-methodology`) | Passive domain recon and the secure working method | Production domain (passive) and daily coding |

## 1. Cloudflare audit skill

MIT license. Repo: `github.com/cloudflare/security-audit-skill`.

```
npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
```

Then, inside the project, ask Claude Code: `security audit this codebase`.

- Maps the architecture, hunts per attack class (web, auth, AI/LLM, supply chain, cloud, resource exhaustion, data isolation…), tries to **disprove** each candidate and writes `findings.json` and `REPORT.md`.
- Repeated runs on the same repo are additive.
- Needs Node.js and, to execute target code, a sandbox; without one, findings stay "needs validation".

**How we use it:** each finding becomes a task, fixed **one at a time**, with a test proving the fix and a security review. Unproven findings stay marked "unconfirmed".

## 2. Strix

Apache-2.0 license. Repo: `github.com/UseStrix/strix`.

```
curl -sSL https://strix.ai/install | bash
export STRIX_LLM="provider/model"   # see Strix docs
export LLM_API_KEY="your-key"       # never commit it
strix --target ./your-app           # local code
```

- Needs **Docker** and a model API key.
- **It really attacks** the target. Point it **only** at your local copy (e.g., `http://localhost:8000` or the project folder). **Never** at production or third-party systems.
- Results in `strix_runs/`. In automation: `strix -n --target ...` (exits non-zero on findings).

**How we use it:** run a local copy with fake data, run Strix, and treat findings like audit ones: fix one at a time, with tests.

## 3. Recommended order

1. `secure-ai-methodology` on from the first commit.
2. `passive-domain-audit` once the domain is live (then monthly).
3. Cloudflare audit on the code at every major delivery.
4. Strix against the local copy before launch and after big changes to login, payments or data.
5. Repeat. Security is a process, not an event.
