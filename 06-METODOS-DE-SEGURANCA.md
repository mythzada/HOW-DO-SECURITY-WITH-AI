# 06 — Catálogo de métodos de segurança (aplicados num sistema real)

Tudo abaixo foi aplicado num SaaS em produção construído com agentes de IA. Use como lista de verificação para o seu sistema. ✅ = aplicado no projeto de referência.

## Identidade e acesso
- ✅ **Senhas com Argon2** (hash lento com sal). Nunca guarde senha em texto ou com hash rápido (MD5/SHA1).
- ✅ **Passkeys (WebAuthn)**: verificação de usuário obrigatória, origem e rpId fixos na configuração (nunca tirados do cabeçalho Host), desafio de uso único preso à sessão e com validade curta, contador anti-clonagem, só a chave pública é guardada.
- ✅ **Conta do dono só com passkey**: senha, link de recuperação e login social recusados para essa conta.
- ✅ **2º fator (TOTP)** com segredo criptografado, código repetido recusado, **códigos de recuperação com hash e de uso único**.
- ✅ **Prova recente** (re-autenticação de poucos minutos) para ações sensíveis: trocar senha, desligar 2FA, cadastrar passkey, área administrativa.
- ✅ **Desligar o 2FA exige passkey**; qualquer reset de fator gera **e-mail de aviso**.
- ✅ **Sessão**: cookie trocado no login; ligar 2FA derruba as outras sessões.
- ✅ **Limites de tentativa** por par usuário+IP, por conta e por IP, com janelas de tempo.
- ✅ **Recuperação de conta** nunca rebaixa a segurança de uma conta forte.

## Área administrativa
- ✅ Caminho não divulgado, ausente de robots, JS, CSS, mapas do site e mensagens de erro.
- ✅ Para quem não é dono, **resposta idêntica a uma página inexistente** em todos os métodos HTTP (GET, HEAD, OPTIONS, POST), antes de CSRF e do tratamento de método — e o limitador de tentativas não pode revelar a diferença.
- ✅ Toda ação administrativa grava **evento de auditoria** (quem, quando, o quê — sem dados sensíveis).
- 🔜 Camada extra na borda (Zero Trust / Access) na frente do caminho.

## Aplicação web
- ✅ **CSRF** em toda requisição que escreve.
- ✅ **CSP restrita**: só scripts do próprio site, sem `unsafe-eval`, sem scripts inline, `object-src 'none'`, `frame-ancestors 'none'`.
- ✅ **Sem `innerHTML` com dado do usuário**; texto entra como texto.
- ✅ **Redirecionamentos seguros**: destino só interno, começando com uma única `/`; recusa `//`, `\`, esquemas (`http:`, `javascript:`, `data:`), variantes codificadas (`%2F`, `%5C`, `%09`), tabs e quebras de linha, validando **depois de normalizar** (NFKC + decodificação limitada) e **no servidor**, não só no JS.
- ✅ **Comparação de domínio por lista positiva**: depois do domínio só podem vir fim, `/`, `?`, `#` ou porta numérica; qualquer outro caractere (incluindo pontos Unicode como `。`) invalida.
- ✅ **Sem regex com retrocesso catastrófico (ReDoS)**: troque por varredura linear e teste com entradas grandes (ex.: 20 mil caracteres) com teto de tempo.
- ✅ **Cookies** `HttpOnly; Secure; SameSite=Strict`, prefixo `__Host-`, **nome neutro** (nome de cookie não pode revelar nada interno).
- ✅ **Cabeçalhos**: HSTS com preload, `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy: no-referrer`, `Permissions-Policy`, COOP/CORP `same-origin`.
- ✅ **Documentação da API fechada** em produção.
- ✅ **Endpoints JSON** com `Cache-Control: no-store` e 401 sem detalhes para quem não está logado.
- ✅ **Dados por dono**: toda consulta filtra pela conta da sessão; testes com duas contas falsas provando que uma não lê a outra (IDOR).

## Dados
- ✅ **Segredos dos usuários** (ex.: chaves de API) criptografados (Fernet) com chave fora do banco.
- ✅ **Códigos curtos** só com hash.
- ✅ **"Nunca guardar"**: coletar o mínimo; migrações de purga apagam o que expira.
- ✅ **Logs** sem senha, token, código ou dado pessoal; em falha, registrar só o identificador e o tipo do erro.
- ✅ **LGPD**: finalidade clara, mínimo necessário, direito de exclusão.

## Pagamentos
- ✅ Webhooks do provedor de pagamento **com assinatura verificada**.
- ✅ **Conciliação diária** entre o banco e o provedor, com alerta.
- ✅ **Alertas sem repetição**: impressão digital por divergência, no máximo um resumo por dia, o dono marca "explicada" com motivo e auditoria — nada é escondido automaticamente.
- ✅ Testes reais de cobrança com valor baixo e estorno, documentando cada falha encontrada.

## Infraestrutura
- ✅ **CDN na frente** (Cloudflare) com IP real escondido (IPv4, IPv6, www, subdomínios).
- ✅ **Firewall da origem**: portas web aceitam só os IPs da CDN; SSH só da faixa do administrador.
- ✅ A aplicação só confia no IP do visitante informado pela CDN quando a conexão vem da CDN.
- ✅ **TLS Full (strict), mínimo 1.2, HSTS, Always HTTPS**.
- ✅ **WAF gerenciado** + regra bloqueando sondagens (`.env`, `.git`, WordPress, PHP) + rate limit em rotas sensíveis.
- ✅ **DNSSEC, CAA, SPF `-all`, DMARC `p=reject`, Null MX**, envio de e-mail por subdomínio dedicado com DKIM.
- ✅ **whois** com e-mail de contato dedicado.
- ✅ Configurações antigas de hospedagem que não são usadas são **removidas do repositório**.
- 🔜 **Authenticated Origin Pulls** (certificado de cliente entre CDN e origem).

## Código e processo
- ✅ **Verificador de segredos** antes de todo commit (com baseline revisada).
- ✅ **E-mail privado no git** (endereço noreply) e bloqueio de push que exponha o e-mail.
- ✅ **Migrações de banco lineares**: uma base, uma cabeça; guarda automática que recusa duas cabeças ou revisão publicada alterada.
- ✅ **Testes de "impressão digital" de telas sensíveis** (hash do HTML renderizado): qualquer mudança em tela de login/2FA exige revisão consciente.
- ✅ **Arquivos sensíveis com dono único** (autenticação, sessão, cobrança, cálculo): só um agente designado altera.
- ✅ **Todo PR**: teste local colado, revisão de segurança no commit exato, veredito em linguagem simples, plano de volta; assert alterado declarado.
- ✅ **Recurso novo atrás de flag desligada** até estar verificado.
- ✅ **Auditorias com IA** periódicas (skill de auditoria da Cloudflare) + correção achado por achado.
- ✅ **Reconhecimento passivo** periódico do próprio domínio ([02](02-RECONHECIMENTO-PASSIVO.md)).
- ✅ **`security.txt`** para receber relatos de falhas.

## Contas e pessoas
- ✅ Login de todos os painéis (registrador, CDN, nuvem, Git) com **2FA/passkey**.
- ✅ Credenciais nunca passam por IA; cliques finais de criar/apagar/pagar/publicar são humanos.
- ✅ Contas abandonadas em serviços que não usa são **excluídas**.
- ✅ Repositórios que não precisam ser públicos ficam **privados**.
