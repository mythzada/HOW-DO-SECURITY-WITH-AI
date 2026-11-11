# 01 — Camadas de defesa

Defesa em profundidade: se uma camada falhar, a próxima segura. Abaixo, cada camada com o que aplicamos e por quê.

## 1. DNS

| Controle | Para que serve | Como verificar |
|---|---|---|
| **DNS na Cloudflare, registros "proxied"** | O IP real do servidor nunca aparece no DNS público | `dig dominio A` / `AAAA` → só IPs da Cloudflare |
| **DNSSEC** | Assinatura digital do DNS; impede respostas falsificadas | `dig +dnssec dominio @1.1.1.1` → flag `ad` |
| **CAA** | Só autoridades listadas podem emitir certificado para o domínio | `dig dominio CAA` |
| **Null MX** (`MX 0 .`) | Declara que o domínio não recebe e-mail — nada para atacar | `dig dominio MX` |
| **SPF `-all`** | Nenhum servidor pode enviar e-mail pelo domínio principal | `dig dominio TXT` |
| **DMARC `p=reject`** | E-mail falso em nome do domínio é recusado | `dig _dmarc.dominio TXT` |
| **Envio por subdomínio dedicado** | O sistema envia e-mails por um subdomínio próprio com DKIM, sem abrir o domínio principal | "Mostrar original" no Gmail → DKIM/DMARC PASS |

> Dica: no Registro.br (.br), o titular é público. Use um e-mail de contato dedicado (ex.: endereço privado/relay) e, quando houver empresa, registre o domínio no CNPJ.

## 2. Borda (Cloudflare)

- **SSL/TLS Full (strict)** + **TLS mínimo 1.2** + **Always Use HTTPS**.
- **HSTS** com `includeSubDomains; preload`.
- **WAF gerenciado** ligado.
- **Regra customizada** bloqueando sondagens comuns (`.env`, `.git`, WordPress, PHP em site que não usa PHP).
- **Rate limit** em rotas sensíveis.
- **Alertas** de certificado e eventos de segurança para o e-mail do dono.
- **Bot Fight Mode**: avaliar com cuidado — pode bloquear clientes e prévias de link.
- **Bloqueio de robôs de treino de IA** (robots gerenciado) sem afetar o Google.

## 3. Servidor (origem)

- **Firewall de nuvem**: portas 80/443 aceitam **apenas os blocos de IP da Cloudflare**. Quem descobrir o IP real não conecta.
- **SSH** liberado só para a faixa de IP de quem administra.
- **Próximo passo:** Authenticated Origin Pulls (certificado de cliente da Cloudflare), para impedir que alguém aponte a *própria* conta Cloudflare para o seu IP.
- A aplicação só confia no IP do visitante informado pela Cloudflare quando a conexão vem da Cloudflare.

## 4. Aplicação

- **Cabeçalhos de segurança:** CSP restrita (`default-src 'self'`, `object-src 'none'`, `frame-ancestors 'none'`), `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer`, `Permissions-Policy`, COOP/CORP `same-origin`.
- **Cookies:** `HttpOnly; Secure; SameSite=Strict`, preferencialmente com prefixo `__Host-` e **nome neutro** (o nome do cookie não pode revelar nada interno).
- **CSRF** em toda escrita.
- **Redirecionamentos:** só destinos internos que começam com uma única `/` (recusar `//`, `\`, esquemas, variantes codificadas, tabs e quebras de linha) — validado no servidor, não só no JS.
- **Área administrativa:** caminho não divulgado, ausente do robots/JS/CSS, e para quem não é dono responde **exatamente igual a uma página inexistente** — em todos os métodos HTTP, antes de CSRF e antes do 405, e sem que o rate limit revele a diferença.
- **Login do dono só com passkey (WebAuthn):** verificação de usuário obrigatória, origem e rpId fixos na configuração, desafio de uso único preso à sessão, contador anti-clonagem. Conta "só passkey" recusa senha, link de recuperação e login social.
- **2º fator:** desligar exige prova recente com passkey; reset de fator quebrado gera aviso por e-mail.
- **Documentação da API** (`/docs`, `/openapi.json`) fechada em produção.
- **`security.txt`** publicado com contato, política e validade.

## 5. Dados

- **Senhas:** Argon2 (hash lento e com sal; nunca reversível).
- **Segredos guardados** (ex.: chaves de API dos usuários): criptografia simétrica (Fernet) com chave fora do banco.
- **Pinos e códigos curtos:** hash (SHA-256) — o valor original não é guardado.
- **Passkeys:** guarda só chave pública, contador e metadados. Nunca a chave privada.
- **Princípio do "nunca guardar":** o que não é necessário não é coletado; o que expira é apagado (migrações de purga).

## 6. Operação

- Alertas que **não repetem** o mesmo aviso (impressão digital por divergência, no máximo um resumo por dia) — alerta repetido vira ruído e esconde o alerta real.
- Toda mudança de risco tem **plano de rollback** escrito no PR.
