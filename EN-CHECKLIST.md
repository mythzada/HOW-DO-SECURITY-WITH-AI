# Quick checklist

## DNS & email
- [ ] Records proxied; IPv4, IPv6 and www without the real IP
- [ ] DNSSEC active (`ad` flag) and DS matching at the registrar
- [ ] CAA with allowed authorities
- [ ] Null MX if you receive no email
- [ ] SPF `-all` on the main domain; DMARC `p=reject`; DKIM PASS
- [ ] Dedicated whois contact email

## Edge
- [ ] TLS Full (strict), min 1.2, Always HTTPS, HSTS preload
- [ ] Managed WAF + anti-probe rule (`.env`, `.git`, WP/PHP)
- [ ] Rate limiting on sensitive routes; alerts on

## Server
- [ ] Firewall: 80/443 from CDN only; SSH from your range only
- [ ] Authenticated Origin Pulls
- [ ] API docs closed

## Application
- [ ] CSP, X-Frame-Options, nosniff, Referrer-Policy, Permissions-Policy, COOP/CORP
- [ ] Cookies `__Host-`, HttpOnly, Secure, SameSite=Strict, neutral name
- [ ] CSRF on every write
- [ ] Internal-only redirects (server-validated)
- [ ] Invisible admin (same 404 as nonexistent, every method)
- [ ] Owner passkey-only; 2FA can't be disabled without passkey
- [ ] security.txt; robots without secret paths

## Data
- [ ] Argon2 passwords; encrypted secrets; hashed codes
- [ ] Collect the minimum; purge what expires

## Process
- [ ] Every PR with pasted local tests + VERIFY + verdict
- [ ] Changed asserts declared
- [ ] Rollback written
- [ ] Lessons recorded
