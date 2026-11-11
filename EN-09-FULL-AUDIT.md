# 09 — Full staged audit (the way we did it)

You don't have to find the flaws by hand. The **`full-security-audit`** skill makes your AI agent walk the whole system in 14 stages, the same way we audited a real SaaS: every finding proven with a test, fixed in a small PR and explained in plain language.

```
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill full-security-audit
```

Then, in Claude Code, inside a **local copy** of your project: *"run the full staged security audit"*.

## The stages

| # | Stage | What it hunts |
|---|---|---|
| 0 | System map | Every route, who can reach it, sensitive zones |
| 1 | SQL injection and injection | Queries built from text, free `ORDER BY`, OS commands, templates, NoSQL |
| 2 | IDOR and isolation | Change the ID, see another account's data; two-fake-account test on every route |
| 3 | Input, sanitization, XSS | `innerHTML`, `|safe`, unfiltered Markdown, browser-only validation, Unicode |
| 4 | Login, session, 2FA | Password hashing, breached passwords, attempt limits, paths that skip 2FA, passkeys, recovery |
| 5 | CSRF, CORS, redirects | Writes without CSRF, open CORS, open redirects via `//`, `\`, `%09` |
| 6 | SSRF | Server fetching a user URL that lands on internal IPs or cloud metadata |
| 7 | Uploads | Type by extension, executable files, zip bomb, zip slip, EXIF, antivirus freezing the server |
| 8 | Denial of service | Slow regex (ReDoS), expensive routes without limits, a limiter that reveals hidden routes |
| 9 | Secrets and dependencies | Secrets in code or history, over-privileged keys, vulnerable dependencies |
| 10 | Business logic and payments | Unsigned or replayed webhooks, price from the form, credit race conditions, reconciliation |
| 11 | AI / LLM | Prompt injection, AI reading another account's data, executed AI output, uncapped cost |
| 12 | Personal data and logs | Storing more than needed, logs with passwords/tokens, uncontrolled people search |
| 13 | Headers, admin, edge | CSP, cookies, an admin area that gives itself away, origin firewall, DNS |
| 14 | Sweep and report | Cloudflare skill, Strix on the local copy, permanent tests, final report |

## Rules that make the audit trustworthy

1. **Local copy, never production.** Active attacks only on your machine.
2. **No test, no finding.** The test fails before the fix and passes after.
3. **One fix per PR**, with an explanation and a rollback plan.
4. **Never weaken a test** to go green.
5. **Found secrets are never repeated**: say where, and rotate.
6. **Report in `AUDIT.md`**, so you can resume.

## What AI doesn't replace

Before storing real sensitive data, also get an **independent human pentest**. AI auditing finds a lot, but it doesn't sign off alone.

See also: [06 — security methods](EN-06-SECURITY-METHODS.md) · [08 — attack to defense](EN-08-ATTACK-TO-DEFENSE.md) · [07 — AI tools](EN-07-AI-SECURITY-TOOLS.md)
