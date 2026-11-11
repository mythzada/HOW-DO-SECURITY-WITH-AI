# 02 — Passive recon (audit your domain like an attacker)

Everything here is **passive or equivalent to a normal browser visit**: safe to run on your own production domain. Replace `YOURDOMAIN`.

## Google dorks
```
site:YOURDOMAIN inurl:admin
site:YOURDOMAIN intitle:"index of"
site:YOURDOMAIN ext:env OR ext:sql OR ext:log OR ext:bak
site:YOURDOMAIN inurl:docs OR inurl:openapi
site:YOURDOMAIN "Traceback" OR "Internal Server Error"
site:*.YOURDOMAIN -www
```
**Expected:** zero results.

## Ownership
```
whois YOURDOMAIN
```
See what is public (name, email, ID). Use a dedicated contact email.

## DNS
```
dig YOURDOMAIN            # IPv4 — CDN IPs only
dig YOURDOMAIN AAAA       # IPv6 — often forgotten; CDN too
dig www.YOURDOMAIN
dig YOURDOMAIN NS
dig YOURDOMAIN MX         # "0 ." = Null MX
dig YOURDOMAIN TXT        # SPF
dig _dmarc.YOURDOMAIN TXT # DMARC (p=reject is ideal)
dig YOURDOMAIN CAA
dig YOURDOMAIN DS +short  # DNSSEC published at the registrar
dig +dnssec YOURDOMAIN @1.1.1.1 | grep flags   # "ad" = validated
```
Visual tool: **dnsviz.net**.

## Subdomains (Certificate Transparency)
```
curl -s "https://api.certspotter.com/v1/issuances?domain=YOURDOMAIN&include_subdomains=true&expand=dns_names" \
  | grep -o '"[a-z0-9.*-]*YOURDOMAIN"' | sort -u
```
(crt.sh works too but is often overloaded.) `dig` each one: **none** may point to your real server.

## HTTP / TLS
```
curl -sI https://YOURDOMAIN
curl -sI http://YOURDOMAIN | head -3     # must be 301 to https
echo | openssl s_client -connect YOURDOMAIN:443 -servername YOURDOMAIN 2>/dev/null \
  | openssl x509 -noout -issuer -subject -dates
openssl s_client -connect YOURDOMAIN:443 -tls1_1 </dev/null 2>&1 | grep -iE "alert|Cipher is"   # must fail
```
Look for HSTS, CSP, frame protection, nosniff, referrer and permissions policies, cookies with HttpOnly/Secure/SameSite, and **no** server version or internal names leaking (including **cookie names**).

## Public files
```
curl -s https://YOURDOMAIN/robots.txt
curl -s https://YOURDOMAIN/.well-known/security.txt
curl -s -o /dev/null -w "%{http_code}\n" https://YOURDOMAIN/.env
curl -s -o /dev/null -w "%{http_code}\n" https://YOURDOMAIN/.git/config
```
- robots must **not** list secret paths (Disallow announces them).
- security.txt: Contact, Expires, Policy.
- `.env` / `.git`: `403` or `404`.

## Online passive scanners
Mozilla Observatory, SSL Labs, Security Headers, DNSViz. Treat findings as **leads** and confirm in code.

## Terminal tips
- Pasted text starts with `^[[200~`? Paste garbage — delete it before Enter.
- Kali terminal paste: `Ctrl+Shift+V`.
