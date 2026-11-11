# 06 — Security methods catalog (applied in a real system)

Everything below was applied in a production SaaS built with AI agents. Use it as a checklist for your own system. ✅ = applied in the reference project.

## Identity and access
- ✅ **Argon2 password hashing** (slow, salted). Never plaintext or fast hashes (MD5/SHA1).
- ✅ **Passkeys (WebAuthn)**: user verification required, origin and rpId fixed in config (never taken from the Host header), single-use, short-lived challenge bound to the session, anti-cloning counter, only the public key stored.
- ✅ **Owner account is passkey-only**: password, recovery link and social login refused for it.
- ✅ **TOTP 2FA** with encrypted secret, replayed codes refused, **hashed single-use recovery codes**.
- ✅ **Recent proof** (re-authentication within a few minutes) for sensitive actions: password change, disabling 2FA, adding passkeys, admin area.
- ✅ **Disabling 2FA requires a passkey**; any factor reset triggers an **email notice**.
- ✅ **Sessions**: cookie rotated at login; enabling 2FA revokes other sessions.
- ✅ **Rate limits** per user+IP pair, per account and per IP, with time windows.
- ✅ **Account recovery** never downgrades a strong account.

## Admin area
- ✅ Undisclosed path, absent from robots, JS, CSS, sitemaps and error messages.
- ✅ For non-owners, **identical response to a nonexistent page** for every HTTP method (GET, HEAD, OPTIONS, POST), before CSRF and method handling — and rate limiting must not reveal the difference.
- ✅ Every admin action writes an **audit event** (who, when, what — no sensitive data).
- 🔜 Extra edge layer (Zero Trust / Access) in front of the path.

## Web application
- ✅ **CSRF** on every write.
- ✅ **Strict CSP**: own scripts only, no `unsafe-eval`, no inline scripts, `object-src 'none'`, `frame-ancestors 'none'`.
- ✅ **No `innerHTML` with user data**.
- ✅ **Safe redirects**: internal targets only, starting with a single `/`; reject `//`, `\`, schemes (`http:`, `javascript:`, `data:`), encoded variants (`%2F`, `%5C`, `%09`), tabs and newlines; validate **after normalizing** (NFKC + bounded decoding) and **on the server**, not only in JS.
- ✅ **Domain matching with an allow-list**: after the domain only end, `/`, `?`, `#` or a numeric port may follow; anything else (including Unicode dots like `。`) invalidates.
- ✅ **No catastrophic-backtracking regex (ReDoS)**: use linear scanners and test with large inputs (e.g., 20k chars) under a time ceiling.
- ✅ **Cookies** `HttpOnly; Secure; SameSite=Strict`, `__Host-` prefix, **neutral names**.
- ✅ **Headers**: HSTS with preload, `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy: no-referrer`, `Permissions-Policy`, COOP/CORP `same-origin`.
- ✅ **API docs closed** in production.
- ✅ **JSON endpoints** with `Cache-Control: no-store` and a detail-free 401 when logged out.
- ✅ **Per-owner data**: every query filters by the session's account; tests with two fake accounts prove one can't read the other (IDOR).

## Data
- ✅ **User secrets** (e.g., API keys) encrypted (Fernet) with the key outside the database.
- ✅ **Short codes** stored only as hashes.
- ✅ **"Never store"**: collect the minimum; purge migrations delete what expires.
- ✅ **Logs** without passwords, tokens, codes or personal data; on failure log only the id and error type.
- ✅ **Privacy law** (LGPD/GDPR): clear purpose, minimum data, right to deletion.

## Payments
- ✅ Payment provider webhooks with **verified signatures**.
- ✅ **Daily reconciliation** between database and provider, with alerts.
- ✅ **Non-repeating alerts**: fingerprint per discrepancy, at most one daily digest, owner marks "explained" with a reason and audit — nothing auto-hidden.
- ✅ Real low-value charge tests with refunds, documenting every failure found.

## Infrastructure
- ✅ **CDN in front** (Cloudflare), real IP hidden (IPv4, IPv6, www, subdomains).
- ✅ **Origin firewall**: web ports accept only CDN IPs; SSH only from the admin's range.
- ✅ The app trusts the CDN's client-IP header only when the connection comes from the CDN.
- ✅ **TLS Full (strict), min 1.2, HSTS, Always HTTPS**.
- ✅ **Managed WAF** + rule blocking probes (`.env`, `.git`, WordPress, PHP) + rate limiting on sensitive routes.
- ✅ **DNSSEC, CAA, SPF `-all`, DMARC `p=reject`, Null MX**, email sent from a dedicated subdomain with DKIM.
- ✅ **whois** with a dedicated contact email.
- ✅ Unused old hosting configs **removed from the repo**.
- 🔜 **Authenticated Origin Pulls** (client certificate between CDN and origin).

## Code and process
- ✅ **Secret scanner** before every commit (reviewed baseline).
- ✅ **Private email in git** (noreply) and blocking pushes that expose it.
- ✅ **Linear DB migrations**: one base, one head; an automatic guard rejects two heads or edited published revisions.
- ✅ **Fingerprint tests of sensitive screens** (hash of rendered HTML): any change to login/2FA screens needs a conscious review.
- ✅ **Single-owner sensitive files** (auth, sessions, billing, calculations): only one designated agent changes them.
- ✅ **Every PR**: pasted local tests, security review at the exact commit, plain-language verdict, rollback plan; changed asserts declared.
- ✅ **New features behind a disabled flag** until verified.
- ✅ **Periodic AI audits** (Cloudflare's audit skill) + fixing finding by finding.
- ✅ **Periodic passive recon** of your own domain ([EN-02](EN-02-PASSIVE-RECON.md)).
- ✅ **`security.txt`** to receive vulnerability reports.

## Accounts and people
- ✅ Every dashboard login (registrar, CDN, cloud, Git) with **2FA/passkey**.
- ✅ Credentials never go through AI; final create/delete/pay/publish clicks are human.
- ✅ Abandoned accounts on unused services are **deleted**.
- ✅ Repos that don't need to be public stay **private**.
