---
name: passive-domain-audit
description: Use to audit the external security of a domain you own (DNS, email, TLS, headers, subdomains, exposed files) with passive reconnaissance only, explaining every finding in plain language.
---

# Passive domain audit

Audits a domain **owned by the user** the way an attacker would during recon, without attacking anything. Reply briefly, in the user's language, with ✅/⚠️/❌ per item.

## Rules

- Confirm the domain belongs to the user (or there is written authorization). If not, stop.
- Passive commands or normal-browser-equivalent requests only. **Never** run Nmap, Nikto, sqlmap, gobuster, hydra, brute force or pentest agents against production — those only against a local copy.
- Don't publish or repeat personal data that shows up (whois, emails, IDs, origin IPs).
- Respect computer-crime and data-protection laws (e.g., Brazil art. 154-A / LGPD, CFAA, GDPR).

## Steps (replace D with the domain)

1. **Dorks** (user runs on Google): `site:D inurl:admin`, `site:D intitle:"index of"`, `site:D ext:env OR ext:sql OR ext:log OR ext:bak`, `site:D inurl:docs OR inurl:openapi`, `site:D "Traceback" OR "Internal Server Error"`, `site:*.D -www`. Expected: zero.
2. **whois D**: what is public (name, email, ID). Suggest a dedicated contact email; company registration when possible.
3. **DNS**: `dig D`, `dig D AAAA`, `dig www.D` (CDN IPs only, never the real IP); `dig D NS`; `dig D MX` (`0 .` = Null MX); `dig D TXT` (SPF, `-all` if the main domain sends nothing); `dig _dmarc.D TXT` (`p=reject` ideal); `dig D CAA`; `dig D DS +short` and `dig +dnssec D @1.1.1.1 | grep flags` (`ad` flag).
4. **Subdomains**: `curl -s "https://api.certspotter.com/v1/issuances?domain=D&include_subdomains=true&expand=dns_names" | grep -o '"[a-z0-9.*-]*D"' | sort -u` (crt.sh as fallback). `dig` each one: none may point to the origin.
5. **HTTP/TLS**: `curl -sI https://D` (HSTS with preload, CSP without unsafe-eval, X-Frame-Options/frame-ancestors, nosniff, Referrer-Policy, Permissions-Policy, COOP/CORP, cookies HttpOnly+Secure+SameSite, no server version, **cookie names without internal hints**); `curl -sI http://D | head -3` (301 to https); `openssl s_client` for issuer and dates; a `-tls1_1` test must fail.
6. **Public files**: `robots.txt` (must not list secret paths), `/.well-known/security.txt` (Contact, Expires, Policy), `/.env` and `/.git/config` (403/404).
7. **Visual**: dnsviz.net (DNSSEC), Mozilla Observatory, SSL Labs.
8. **Real email**: Gmail "Show original" of a message from the domain → SPF, DKIM and DMARC PASS.

## Output

- A scoreboard with ✅/⚠️/❌ per item and one plain-language line each.
- Findings ordered by risk, with the proposed fix.
- What was **not** verified.
- If the user pastes screenshots, analyze them one by one before the scoreboard.

## Terminal tips

- `^[[200~` at the start = paste garbage; delete before Enter.
- Kali terminal: paste with Ctrl+Shift+V.
