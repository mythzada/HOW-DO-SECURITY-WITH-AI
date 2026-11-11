# 02 — Reconhecimento passivo (audite seu domínio como um atacante)

Tudo aqui é **passivo ou equivalente a uma visita normal de navegador**: seguro para rodar no seu próprio domínio em produção. Troque `SEUDOMINIO` pelo seu.

## Google dorks

```
site:SEUDOMINIO inurl:admin
site:SEUDOMINIO intitle:"index of"
site:SEUDOMINIO ext:env OR ext:sql OR ext:log OR ext:bak
site:SEUDOMINIO inurl:docs OR inurl:openapi
site:SEUDOMINIO "Traceback" OR "Internal Server Error"
site:*.SEUDOMINIO -www
```
**Esperado:** zero resultados. Qualquer resultado é um vazamento indexado.

## Titularidade

```
whois SEUDOMINIO
```
Veja o que fica público (nome, e-mail, documento). No `.br` o titular é público: use e-mail de contato dedicado e, se possível, CNPJ.

## DNS

```
dig SEUDOMINIO            # IPv4 — devem ser IPs da CDN
dig SEUDOMINIO AAAA       # IPv6 — muita gente esquece; também CDN
dig www.SEUDOMINIO
dig SEUDOMINIO NS
dig SEUDOMINIO MX         # "0 ." = Null MX (não recebe e-mail)
dig SEUDOMINIO TXT        # SPF
dig _dmarc.SEUDOMINIO TXT # DMARC (p=reject é o ideal)
dig SEUDOMINIO CAA        # quem pode emitir certificado
dig SEUDOMINIO DS +short  # DNSSEC publicado no registrador
dig +dnssec SEUDOMINIO @1.1.1.1 | grep flags   # flag "ad" = DNSSEC validado
```

Ferramenta visual: **dnsviz.net** mostra a cadeia de confiança do DNSSEC.

## Subdomínios (Certificate Transparency)

```
curl -s "https://api.certspotter.com/v1/issuances?domain=SEUDOMINIO&include_subdomains=true&expand=dns_names" \
  | grep -o '"[a-z0-9.*-]*SEUDOMINIO"' | sort -u
```
(O crt.sh também serve, mas vive sobrecarregado.) Rode `dig` em cada subdomínio: **nenhum** pode apontar para o IP real do servidor.

## HTTP / TLS

```
curl -sI https://SEUDOMINIO                    # cabeçalhos de segurança
curl -sI http://SEUDOMINIO | head -3           # tem que ser 301 para https
echo | openssl s_client -connect SEUDOMINIO:443 -servername SEUDOMINIO 2>/dev/null \
  | openssl x509 -noout -issuer -subject -dates
openssl s_client -connect SEUDOMINIO:443 -tls1_1 </dev/null 2>&1 | grep -iE "alert|Cipher is"   # tem que falhar
```

O que procurar nos cabeçalhos: `strict-transport-security`, `content-security-policy`, `x-frame-options`, `x-content-type-options`, `referrer-policy`, `permissions-policy`, cookies com `HttpOnly; Secure; SameSite`, e **nenhum** nome/versão de servidor ou nome interno vazando (inclusive no **nome dos cookies**).

## Arquivos públicos

```
curl -s https://SEUDOMINIO/robots.txt
curl -s https://SEUDOMINIO/.well-known/security.txt
curl -s -o /dev/null -w "%{http_code}\n" https://SEUDOMINIO/.env
curl -s -o /dev/null -w "%{http_code}\n" https://SEUDOMINIO/.git/config
```
- robots: **não pode listar caminhos secretos** (Disallow anuncia o caminho).
- security.txt: Contact, Expires, Policy.
- `.env` / `.git`: `403` ou `404`.

## Scanners online (passivos)

Mozilla Observatory, SSL Labs, Security Headers, DNSViz, scanners de "am I hackable". Use os achados como **pistas** e confirme no código.

## Dicas de terminal

- Colou e apareceu `^[[200~`? É lixo de colagem — apague antes de dar Enter.
- No terminal do Kali, colar é `Ctrl+Shift+V`.
- Teclado errado na VM: `setxkbmap br`.
