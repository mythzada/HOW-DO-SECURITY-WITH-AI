# 01 — Defense layers

Defense in depth: if one layer fails, the next one holds.

## 1. DNS

| Control | Purpose | How to check |
|---|---|---|
| **DNS on Cloudflare, records "proxied"** | The server's real IP never appears in public DNS | `dig domain A` / `AAAA` → only CDN IPs |
| **DNSSEC** | Digital signature on DNS; blocks forged answers | `dig +dnssec domain @1.1.1.1` → `ad` flag |
| **CAA** | Only listed authorities may issue certificates | `dig domain CAA` |
| **Null MX** (`MX 0 .`) | Declares the domain receives no email — nothing to attack | `dig domain MX` |
| **SPF `-all`** | No server may send mail as the main domain | `dig domain TXT` |
| **DMARC `p=reject`** | Spoofed mail in your name is rejected | `dig _dmarc.domain TXT` |
| **Sending from a dedicated subdomain** | The app sends via its own subdomain with DKIM, keeping the main domain closed | Gmail "Show original" → DKIM/DMARC PASS |

> Tip: in some registries (like Brazil's .br) the holder is public. Use a dedicated contact email (e.g., a private relay) and register under a company when possible.

## 2. Edge (Cloudflare)

- **SSL/TLS Full (strict)** + **minimum TLS 1.2** + **Always Use HTTPS**.
- **HSTS** with `includeSubDomains; preload`.
- **Managed WAF** on.
- **Custom rule** blocking common probes (`.env`, `.git`, WordPress, PHP on non-PHP sites).
- **Rate limiting** on sensitive routes.
- **Alerts** for certificates and security events.
- **Bot Fight Mode**: evaluate carefully — it can block customers and link previews.
- **Block AI training crawlers** (managed robots) without affecting Google.

## 3. Server (origin)

- **Cloud firewall**: ports 80/443 accept **only Cloudflare IP ranges**. Finding the real IP is useless.
- **SSH** open only to the admin's IP range.
- **Next step:** Authenticated Origin Pulls, so nobody can point *their own* Cloudflare account at your IP.
- The app trusts the visitor IP header only when the connection comes from Cloudflare.

## 4. Application

- **Security headers:** strict CSP (`default-src 'self'`, `object-src 'none'`, `frame-ancestors 'none'`), `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy: no-referrer`, `Permissions-Policy`, COOP/CORP `same-origin`.
- **Cookies:** `HttpOnly; Secure; SameSite=Strict`, ideally with the `__Host-` prefix and a **neutral name** (cookie names must not reveal internals).
- **CSRF** on every write.
- **Redirects:** only internal targets starting with a single `/` (reject `//`, `\`, schemes, encoded variants, tabs/newlines) — validated on the server, not only in JS.
- **Admin area:** undisclosed path, absent from robots/JS/CSS; for non-owners it answers **exactly like a nonexistent page** — for every HTTP method, before CSRF and 405 handling, and without rate limiting revealing the difference.
- **Owner login with passkey only (WebAuthn):** user verification required, fixed origin and rpId, single-use challenge bound to the session, anti-cloning counter. A "passkey-only" account refuses password, recovery link and social login.
- **2FA:** turning it off requires fresh passkey proof; resetting a broken factor sends an email notice.
- **API docs** (`/docs`, `/openapi.json`) closed in production.
- **`security.txt`** published.

## 5. Data

- **Passwords:** Argon2.
- **Stored secrets** (e.g., users' API keys): symmetric encryption (Fernet) with the key outside the database.
- **Short codes/pins:** hashed (SHA-256).
- **Passkeys:** store only public key, counter and metadata.
- **"Never store" principle:** collect the minimum; purge what expires.

## 6. Operations

- Alerts that **don't repeat** (fingerprint per issue, at most one daily digest) — repeated alerts become noise and hide the real one.
- Every risky change ships with a written **rollback plan**.
