---
name: full-security-audit
description: Use when the user asks for a complete security audit of their own system (web/API/SaaS). Runs in 14 stages on a local copy, hunting SQL injection, IDOR, XSS, sanitization, login/session/2FA flaws, CSRF, redirects, SSRF, uploads, ReDoS, secrets, payments, AI/LLM, personal data, headers and edge. Proves every finding with a test, fixes one at a time and explains in plain language.
---

# Full staged security audit

You will audit **the user's own** system the way it was done on a real SaaS built with AI: stage by stage, every finding proven with a test, fixed in a small PR and explained in plain language. The user doesn't need to know where to look: **you** look.

## Rules (first)

- Only the user's system or one with written authorization (computer-crime and data-protection laws, e.g. CFAA, GDPR, Brazil's art. 154-A/LGPD). Otherwise, stop.
- Work **on a local copy** (clone + test database with fake data). Never run active attacks against production. In production, only what a normal browser would do.
- Never read, copy or display real secrets (.env, keys, passwords). If you find one, say **where** it is and that it must be rotated — never repeat the value.
- One finding = one test that fails before the fix and passes after. Never weaken an existing test to go green; if an assert changes, declare and justify it.
- One fix per PR, with: what it was, why it's dangerous, how it was proven, how to roll back.
- Talk to the user briefly, in their language. End each stage with a ✅/⚠️/❌ scoreboard.

## How to run

Run the stages **in order**. After each, show its scoreboard and ask whether to continue (or keep going if the user said "do everything"). Keep progress in `AUDIT.md` (stage, finding, severity, test, PR, status) so you can resume later.

Severity: **CRITICAL** (takes over other accounts/data, runs code) · **HIGH** (leaks data, bypasses login/2FA) · **MEDIUM** (helps an attack, reveals structure) · **LOW** (hardening).

---

## Stage 0 — System map

1. List language, framework, database, ORM, templates, queue, file storage, external providers (payments, email, AI, OAuth).
2. Produce the **list of every route** (method + path + needs login? + needs role? + accepts files? + writes to DB?). E.g. FastAPI `app.routes`; Express `app._router.stack`; Django `show_urls`; Rails `rails routes`; Laravel `route:list`.
3. Mark **sensitive zones**: login, session, 2FA, recovery, admin, payments, uploads, personal data, AI calls.
4. Run the current test suite and note what already fails (inherited failures aren't yours).

Output: route table + sensitive zones. It drives every later stage.

## Stage 1 — SQL injection and injection in general

Look for queries built from user text:

```
grep -rnE "execute\(|raw\(|text\(|query\(|\.format\(|f\"(SELECT|INSERT|UPDATE|DELETE)|\+ *['\"] *(WHERE|AND|OR)" --include=*.{py,js,ts,php,rb,go,java} .
grep -rnE "ORDER BY .*\{|LIMIT .*\{|LIKE .*%s.*%" .
```

- Every query uses **parameters** (`?`, `:name`, driver `%s`), never concatenation/f-strings.
- `ORDER BY`, column names and direction come from an **allow-list**, not the user.
- `LIKE` escapes `%` and `_`.
- Also: command injection (`os.system`, `subprocess(..., shell=True)`, `exec`, `eval`), template injection (rendering a user string), header/email injection (newlines in `Subject`/`To`), NoSQL operator injection (`$where`, `$ne` from JSON).

Proof: tests with `' OR 1=1 --`, `1; DROP TABLE x`, `ORDER BY (SELECT 1)` and `{"$ne": null}` showing nothing leaks or breaks (same response as invalid input). Optional tool, **local copy only**: `sqlmap -u "http://127.0.0.1:PORT/route?x=1" --batch --level 2`.

## Stage 2 — IDOR and tenant isolation

The most common SaaS flaw: change the ID in the URL and see someone else's data.

1. For every route taking an ID (`/{id}`, `?id=`, JSON body): does the query filter **by the session's account/organization** (`WHERE id = ? AND owner_id = session`)? Or does it fetch by ID and trust it?
2. Sequential IDs exposed? Prefer random identifiers (UUID/token) **in addition to** the filter — never instead of it.
3. Files: downloads only search **the session account's folder**; file names never come from the user (no `../`).
4. Lists, searches, exports, counters and internal webhooks filter too.
5. Roles: a regular user calling an admin/owner route gets the same response as a nonexistent route.

Proof (mandatory): **a test with two fake accounts** A and B. B tries to read, edit, delete and download each of A's resources through every route → all 404/403, nothing changes in the DB. Even better: a **sweep test** that walks every Stage 0 route as B using A's IDs.

## Stage 3 — Input, sanitization and XSS

```
grep -rnE "innerHTML|outerHTML|insertAdjacentHTML|document\.write|dangerouslySetInnerHTML|v-html|\|safe|mark_safe|Markup\(|autoescape false|\{\{\{" .
```

- Templates with **autoescape on**; no `|safe` on user data.
- In JS, text goes in with `textContent`, never `innerHTML`.
- User Markdown/HTML goes through an **allow-list sanitizer** (e.g. bleach/DOMPurify); links only `http(s):`/`mailto:`.
- **Server-side** validation of type, size, format and range (Pydantic/Zod/Joi schema). Request body size limit.
- Normalize Unicode (NFKC) before comparing; beware homoglyphs in domains/names (IDNA).
- JSON embedded in a page: escape `</script>`.

Proof: tests with `<script>alert(1)</script>`, `"><img src=x onerror=alert(1)>`, `javascript:alert(1)` as a link, and huge text → shown as text, nothing runs, no 500.

## Stage 4 — Login, passwords, session and 2FA

- Passwords: slow hash (Argon2id/bcrypt); NIST 800-63B policy (12+ chars, no forced symbols, reject breached passwords via Have I Been Pwned k-anonymity and passwords equal to name/email).
- Same message for "user doesn't exist" and "wrong password"; similar timing.
- Attempt limits per user+IP, per account and per IP.
- Session: cookie rotated at login (no fixation), expires, logout invalidates server-side; changing password/enabling 2FA kills other sessions.
- 2FA/TOTP: encrypted secret, replayed code rejected, hashed single-use recovery codes; **no path skips the second factor** (recovery, social login, API, "remember device", flag on with empty secret).
- Passkeys/WebAuthn: origin and rpId fixed in config (not from the Host header), single-use short-lived challenge bound to the session, user verification required, anti-cloning counter.
- Sensitive actions (change password/email, disable 2FA, add a key, admin area) require **recent re-authentication**.
- Account recovery: random single-use expiring token, doesn't reveal whether the email exists, never downgrades account security.
- OAuth/social login: `state` validated, fixed redirect_uri, email verified by the provider.

Proof: a test for every alternative path trying to skip the second factor.

## Stage 5 — CSRF, CORS and redirects

- CSRF on every writing request (POST/PUT/PATCH/DELETE), JSON included; `SameSite=Strict/Lax` helps but doesn't replace it.
- CORS: never `*` with credentials; fixed origin list.
- Redirects (`next=`, `return_to=`, `url=`): only internal paths starting with a single `/`. Reject `//`, `\`, `http:`, `javascript:`, `data:`, `%2F%2F`, `%5C`, `%09`, newlines — validating **after** decoding and normalizing, **on the server**.
- Domain comparison by allow-list (after the domain only end, `/`, `?`, `#` or a port).

Proof: 20+ open-redirect payloads in a parametrized test, all ending at `/`.

## Stage 6 — SSRF and outbound calls

Where does the server fetch a user-supplied URL (link preview, import by URL, configurable webhook, remote avatar)?

- Only `https`, port 443, domain allow-list when possible.
- Resolve DNS and **reject private/local/metadata IPs** (127.0.0.0/8, 10/8, 172.16/12, 192.168/16, 169.254.169.254, ::1, fc00::/7) — and check again after redirects.
- Timeout, max size, no following redirects without validation.

Proof: tests with `http://127.0.0.1`, `http://169.254.169.254/`, `http://[::1]`, a domain resolving to an internal IP.

## Stage 7 — Uploads and files

- Type checked by **content** (magic bytes), not extension or the sent `Content-Type`.
- Max size; server-generated name; stored **outside** the public folder; served with `Content-Disposition: attachment` and `nosniff`.
- Never execute uploads (no `.php`, `.html`, `.svg` with scripts served inline).
- Antivirus (e.g. ClamAV) **off** the app's main loop (queue/thread) so it can't freeze the server.
- Archives: total size and file-count limits (zip bomb), no `../` paths (zip slip).
- Images: re-encode and strip metadata (EXIF with location).

Proof: tests uploading `shell.php` renamed `.jpg`, SVG with `<script>`, zip with `../`, a huge file.

## Stage 8 — Denial of service and limits

```
grep -rnE "re\.(compile|match|search|findall)\(|new RegExp\(|/.*(\.\*|\+)\)?(\+|\*)" .
```

- Catastrophic-backtracking regex (`(a+)+`, `(.*)*`, overlapping alternations) → replace with a linear scan; test with 20k chars and a time cap.
- Rate limits on expensive routes (login, search, AI, email sending, upload) and capped pagination.
- Heavy work (antivirus, PDF, AI, email) off the synchronous request path.
- The limiter must not **reveal** hidden routes (429 only where the route exists = leak).

## Stage 9 — Secrets, crypto and dependencies

- Secret scanner over the repo and its **history** (e.g. gitleaks/trufflehog/detect-secrets).
- Secrets only in env vars/vault; `.env` in `.gitignore`.
- Users' secrets (e.g. their API keys) encrypted with a key outside the DB; short codes stored only as hashes.
- Provider keys with **least privilege** (restricted keys with only the permissions used).
- Constant-time token comparison; cryptographic randomness.
- Dependencies pinned by version/hash, vulnerability scanner (pip-audit, npm audit, Dependabot), nothing abandoned.
- Git: private email (noreply), no symlinks or machine-local paths in commits.

## Stage 10 — Business logic, payments and webhooks

- Webhooks with **verified signatures** and **idempotency** (same event twice doesn't grant twice).
- Price, plan and quantity come from the server/provider, never from the form.
- Intermediate states (pending payment, abandoned plan change) don't lock the account or grant access.
- Periodic reconciliation between DB and provider, with **non-repeating** alerts (only new divergences; owner marks "explained" with reason and audit trail).
- Real low-value charge-and-refund tests before opening to the public.
- Race conditions: two simultaneous requests can't spend the same credit/coupon twice (transaction + lock).

## Stage 11 — AI / LLM (OWASP Top 10 for LLM)

- **Prompt injection**: text from documents, pages or users is data, never instructions; tools the AI calls have minimal permissions and human confirmation for risky actions.
- The AI only reaches data **of the session's account** (same Stage 2 filter in every tool).
- AI output is untrusted input (escape HTML, never execute, never becomes SQL).
- Costs: per-account limit, monthly cap at the provider, rate limit.
- AI keys: prefer bring-your-own-key stored encrypted; the server keeps the minimum.
- An AI "test configuration" button must make a real minimal call, not just validate the key.
- AI-generated content labeled as such.

## Stage 12 — Personal data, logs and privacy

- Collect the minimum; what isn't needed is **never stored**; purge migrations delete what expires.
- Logs without passwords, tokens, codes, documents or message content; on errors, only an identifier and type.
- Searching personal data (names, IDs) requires login and an accepted purpose.
- Export and delete-my-account actually work.
- JSON responses with personal data: `Cache-Control: no-store`.
- Admin actions write an audit event (who, when, what — without the sensitive data).

## Stage 13 — Headers, admin area and edge

- Headers: CSP without `unsafe-eval` or inline, `frame-ancestors 'none'`, HSTS with preload, `nosniff`, `Referrer-Policy`, `Permissions-Policy`, COOP/CORP; cookies `__Host-`, `HttpOnly`, `Secure`, `SameSite`, **neutral names** (cookie names reveal nothing internal).
- Invisible admin: for non-owners, **the same response as a nonexistent page** for GET/HEAD/OPTIONS/POST, before CSRF and the limiter; absent from robots, JS, CSS, sitemap and error messages.
- API docs and debug pages closed in production; errors without stack traces.
- Server: firewall accepting only the CDN on web ports; restricted SSH; the app trusts the visitor IP from the CDN only when the connection comes from the CDN.
- Edge: run the `passive-domain-audit` skill (DNS, DNSSEC, CAA, SPF/DKIM/DMARC, TLS, subdomains, exposed files).

## Stage 14 — Automated sweep and report

1. Run Cloudflare's official audit skill if installed: `npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit`.
2. Run Strix **on the local copy only**: `strix --target ./` (or `http://127.0.0.1:PORT`). Each of its findings goes through this skill's rules (prove with a test before fixing).
3. Keep as permanent tests: a route sweep (headers, CSRF, tenant isolation on every route) and an HTML "fingerprint" (hash) of login/2FA screens so any change is deliberate.
4. Deliver the final report in `AUDIT.md`:
   - scoreboard per stage ✅/⚠️/❌;
   - findings by severity, each with test, PR and status;
   - what was **not** verified (e.g. independent human pentest, provider infrastructure);
   - next actions in order.

Remind the user: an AI audit doesn't replace an independent human pentest before handling real sensitive data.
