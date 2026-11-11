# Checklist rápido

## DNS e e-mail
- [ ] Registros na CDN (proxied); IPv4, IPv6 e www sem IP real
- [ ] DNSSEC ativo (flag `ad`) e DS igual no registrador
- [ ] CAA com as autoridades permitidas
- [ ] Null MX se não recebe e-mail
- [ ] SPF `-all` no domínio principal; DMARC `p=reject`; DKIM PASS
- [ ] Contato do whois com e-mail dedicado

## Borda
- [ ] TLS Full (strict), mínimo 1.2, Always HTTPS, HSTS preload
- [ ] WAF gerenciado + regra contra sondagem (`.env`, `.git`, WP/PHP)
- [ ] Rate limit em rotas sensíveis; alertas ligados

## Servidor
- [ ] Firewall: 80/443 só da CDN; SSH só da sua faixa
- [ ] Authenticated Origin Pulls
- [ ] Docs da API fechadas

## Aplicação
- [ ] CSP, X-Frame-Options, nosniff, Referrer-Policy, Permissions-Policy, COOP/CORP
- [ ] Cookies `__Host-`, HttpOnly, Secure, SameSite=Strict, nome neutro
- [ ] CSRF em toda escrita
- [ ] Redirects só para caminhos internos (validado no servidor)
- [ ] Admin invisível (404 igual ao inexistente, em todos os métodos)
- [ ] Dono só com passkey; 2FA não desliga sem passkey
- [ ] security.txt publicado; robots sem caminhos secretos

## Dados
- [ ] Senhas Argon2; segredos criptografados; códigos com hash
- [ ] Coletar o mínimo; purgar o que expira

## Processo
- [ ] Todo PR com teste local colado + VERIFY + veredito
- [ ] Assert alterado declarado no PR
- [ ] Rollback escrito
- [ ] Lições registradas
