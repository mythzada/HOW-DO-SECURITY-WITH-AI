---
name: auditoria-passiva-dominio
description: Use ao auditar a segurança externa de um domínio próprio (DNS, e-mail, TLS, cabeçalhos, subdomínios, arquivos expostos) só com reconhecimento passivo, explicando cada achado em português.
---

# Auditoria passiva de domínio

Audita um domínio **do próprio usuário** do jeito que um atacante faria no reconhecimento, sem atacar nada. Responde em português, curto, com ✅/⚠️/❌ por item.

## Regras

- Confirme que o domínio é do usuário (ou que há autorização por escrito). Se não for, pare.
- Só comandos passivos ou equivalentes a uma visita de navegador. **Nunca** Nmap, Nikto, sqlmap, gobuster, hydra, força bruta ou agentes de pentest contra produção — esses só em cópia local.
- Não publique nem repita dados pessoais que aparecerem (whois, e-mails, documentos, IPs de origem).
- Lei: art. 154-A do Código Penal e LGPD.

## Passos (troque D pelo domínio)

1. **Dorks** (o usuário roda no Google): `site:D inurl:admin`, `site:D intitle:"index of"`, `site:D ext:env OR ext:sql OR ext:log OR ext:bak`, `site:D inurl:docs OR inurl:openapi`, `site:D "Traceback" OR "Internal Server Error"`, `site:*.D -www`. Esperado: zero.
2. **whois D**: o que fica público (nome, e-mail, documento). Sugira e-mail de contato dedicado; no .br, CNPJ quando possível.
3. **DNS**: `dig D`, `dig D AAAA`, `dig www.D` (só IPs da CDN, nunca o IP real); `dig D NS`; `dig D MX` (`0 .` = Null MX); `dig D TXT` (SPF, ideal `-all` se o domínio principal não envia); `dig _dmarc.D TXT` (`p=reject` ideal); `dig D CAA`; `dig D DS +short` e `dig +dnssec D @1.1.1.1 | grep flags` (flag `ad`).
4. **Subdomínios**: `curl -s "https://api.certspotter.com/v1/issuances?domain=D&include_subdomains=true&expand=dns_names" | grep -o '"[a-z0-9.*-]*D"' | sort -u` (crt.sh como alternativa). Rode `dig` em cada um: nenhum pode apontar para a origem.
5. **HTTP/TLS**: `curl -sI https://D` (HSTS com preload, CSP sem unsafe-eval, X-Frame-Options/frame-ancestors, nosniff, Referrer-Policy, Permissions-Policy, COOP/CORP, cookies HttpOnly+Secure+SameSite, sem versão de servidor, **nome de cookie sem pistas internas**); `curl -sI http://D | head -3` (301 para https); `openssl s_client` para emissor e validade; teste `-tls1_1` tem que falhar.
6. **Arquivos públicos**: `robots.txt` (não pode listar caminhos secretos), `/.well-known/security.txt` (Contact, Expires, Policy), `/.env` e `/.git/config` (403/404).
7. **Visual**: dnsviz.net (DNSSEC), Mozilla Observatory, SSL Labs.
8. **E-mail real**: no Gmail, "Mostrar original" de um e-mail do domínio → SPF, DKIM e DMARC PASS.

## Saída

- Um placar com ✅/⚠️/❌ por item e uma linha explicando cada um em linguagem simples.
- Achados ordenados por risco, com a correção proposta.
- O que **não** foi verificado.
- Se o usuário colar prints, analise um por um com calma antes do placar.

## Dicas de terminal

- `^[[200~` no início = lixo de colagem; apagar antes do Enter.
- Terminal do Kali: colar com Ctrl+Shift+V; teclado BR com `setxkbmap br`.
