---
name: secure-by-default
description: Use whenever creating or changing any app (including "vibe coded" ones) — login, database, API, uploads, payments, AI, screens. Makes the AI build everything secure by default, at the level of a real audited SaaS, and refuses to ship a feature without its controls and security test.
---

# Secure by default

You are building an app for the user. One rule: **every feature is born secure**. Security isn't a final step; it's how each line is written. Be persistent: don't skip any item below out of haste or because the user didn't ask. If something can't be done, say what's missing and the risk.

## 1. First time in the project

1. Create (or complete) the project's `CLAUDE.md` with a short **"Security rules"** section summarizing this skill, so every future session follows the same rules.
2. Set up from day one: pre-commit secret scanner, `.env` in `.gitignore`, `.env.example` without values, pinned dependencies, tests runnable with one command, private git email.
3. If the project already exists, run the `full-security-audit` skill before adding features.

## 2. Mandatory recipe per feature type

| When creating… | It ships with… | Test that comes with it |
|---|---|---|
| **Any DB query** | Driver/ORM parameters (never string concatenation); `ORDER BY`/columns from an allow-list; filter by the session's account/org | `' OR 1=1 --` leaks nothing |
| **Route with an ID** | Lookup by `id` **and** session owner; random IDs (UUID) | Account B can't read/edit/delete/download anything of account A (IDOR) |
| **Sign-up and login** | Argon2id/bcrypt; 12+ char passwords rejecting breached ones (HIBP k-anonymity); same message for unknown user and wrong password; attempt limits per user+IP, account and IP; session cookie rotated at login | Brute force blocked; same message for both errors |
| **Session/cookies** | `__Host-`, `HttpOnly`, `Secure`, `SameSite=Lax/Strict`, neutral name; expiry; logout invalidates server-side | Old cookie fails after logout |
| **2FA / passkeys** | TOTP with encrypted secret and no code reuse; hashed recovery codes; passkeys with fixed origin/rpId and single-use challenge; no path skips the second factor | Recovery/social login/API can't get in without the second factor |
| **Sensitive action** (change password/email, delete account, disable 2FA) | Recent re-authentication + notification email | Without re-auth → refused |
| **Form / writing route** | CSRF; server-side schema validation (type, size, format); body size limit | POST without CSRF token → 403 |
| **Screen showing user text** | Template autoescape; `textContent` in JS; Markdown/HTML through an allow-list sanitizer | `<script>` shows as text |
| **Redirect (`next=`)** | Internal path with a single `/` only, validated server-side after normalization | `//site.com`, `\`, `javascript:` → `/` |
| **Fetching a user URL** | https only; reject private/local/metadata IPs, also after redirects; time and size limits | `http://169.254.169.254` refused |
| **Upload** | Type by content; max size; generated name; outside public folder; downloaded as attachment; antivirus off the main loop; EXIF stripped; archive limits | `.php` disguised as `.jpg` refused |
| **Regex** | No catastrophic backtracking; prefer a linear scan | 20k chars in under 1 s |
| **Expensive route** (search, AI, email, upload) | Rate limit; capped pagination; heavy work in a queue | Excess → 429 |
| **Payments** | Price from provider/server; signed, idempotent webhooks; pending states grant nothing; lock against double-spending credit | Replayed webhook doesn't grant twice |
| **AI feature** | External text is data, not instructions; AI tools see only the session account's data and ask before risky actions; AI output escaped; per-account cost cap | A document saying "ignore your instructions" changes nothing |
| **Secret** (API key etc.) | Env var/vault only; users' secrets encrypted with a key outside the DB; provider keys least-privilege | Secret scanner passes |
| **Personal data** | Collect the minimum; never log passwords/tokens/IDs; purge what expires; export and delete account work; `Cache-Control: no-store` | Logs don't contain the data |
| **Admin area** | For non-owners, the same response as a nonexistent page on every method; absent from robots/JS/sitemap; every action audited | Regular user gets the same 404 |
| **Errors** | Friendly page without stack traces; API docs and debug closed in production | A 500 shows no code |
| **HTTP responses** | CSP without `unsafe-eval`/inline, `frame-ancestors 'none'`, HSTS, `nosniff`, `Referrer-Policy`, `Permissions-Policy`, COOP/CORP | Header test across all routes |

## 3. Gate before saying "done"

No feature is done without:

1. The recipe items applied (say which).
2. The recipe's security tests written and passing — paste the command and the result.
3. Existing tests passing; no weakened asserts (if one changed, declare why).
4. A security review of the diff: what could go wrong, for whom, and why it doesn't.
5. A rollback plan.
6. A short plain-language explanation of what was protected.

If the user asks to skip, explain the risk in one sentence and offer the simplest secure version. Never ship the insecure version silently.

## 4. Before going live

- Run `full-security-audit` (local copy) and `passive-domain-audit` (domain).
- CDN in front, HTTPS forced, origin firewall accepting only the CDN, restricted SSH.
- Risky new features behind a flag that stays off until verified.
- Real sensitive data only after an independent human pentest.

## Style

Short, in the user's language. Teach while doing: one line per feature, "this protects against…". Credentials, payments and final create/delete/publish clicks stay with the user.
