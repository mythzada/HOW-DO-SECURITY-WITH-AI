---
name: auditoria-completa-em-etapas
description: Use quando a pessoa pedir uma auditoria de segurança completa do próprio sistema (web/API/SaaS). Roda em 14 etapas, na cópia local, procurando SQL injection, IDOR, XSS, sanitização, falhas de login/sessão/2FA, CSRF, redirect, SSRF, upload, ReDoS, segredos, pagamentos, IA/LLM, dados pessoais, cabeçalhos e borda. Prova cada achado com teste, corrige um por vez e explica em português.
---

# Auditoria completa em etapas

Você vai auditar o sistema **da própria pessoa** do jeito que foi feito num SaaS real construído com IA: etapa por etapa, cada achado provado com teste, corrigido em PR pequeno e explicado em linguagem simples. A pessoa não precisa saber onde procurar: **você** procura.

## Regras (antes de tudo)

- Só sistema da pessoa ou com autorização escrita (art. 154-A do Código Penal; LGPD). Se não for, pare.
- Trabalhe **na cópia local** (clone + banco de teste com dados falsos). Nunca rode ataque ativo contra produção. Em produção, só o que um navegador comum faria.
- Nunca leia, copie ou mostre segredos reais (.env, chaves, senhas). Se achar um, diga **onde** está e que precisa ser trocado — sem repetir o valor.
- Um achado = um teste que falha antes da correção e passa depois. Nunca afrouxe teste existente para ficar verde; se um assert mudar, declare e justifique.
- Uma correção por PR, com: o que era, por que é perigoso, como foi provado, como desfazer.
- Fale com a pessoa em português curto. No fim de cada etapa: placar ✅/⚠️/❌.

## Como rodar

Faça as etapas **em ordem**. Ao terminar cada uma, mostre o placar dela e pergunte se segue para a próxima (ou siga direto se a pessoa pediu "faça tudo"). Guarde o progresso num arquivo `AUDITORIA.md` (etapa, achado, gravidade, teste, PR, status) para retomar depois.

Gravidade: **CRÍTICA** (invade conta/dados de outros, executa código) · **ALTA** (vaza dado, pula login/2FA) · **MÉDIA** (facilita ataque, revela estrutura) · **BAIXA** (endurecimento).

---

## Etapa 0 — Mapa do sistema

1. Liste linguagem, framework, banco, ORM, templates, fila, armazenamento de arquivos, provedores externos (pagamento, e-mail, IA, OAuth).
2. Gere a **lista de todas as rotas** (método + caminho + exige login? + exige papel? + recebe arquivo? + escreve no banco?). Ex.: FastAPI `app.routes`; Express `app._router.stack`; Django `show_urls`; Rails `rails routes`; Laravel `route:list`.
3. Marque as **zonas sensíveis**: login, sessão, 2FA, recuperação, admin, pagamento, upload, dados pessoais, chamadas de IA.
4. Rode a suíte de testes atual e anote o que já falha (falha herdada não é sua).

Saída: tabela de rotas + zonas sensíveis. Ela guia todas as etapas seguintes.

## Etapa 1 — SQL injection e injeção em geral

Procure montagem de consulta com texto do usuário:

```
grep -rnE "execute\(|raw\(|text\(|query\(|\.format\(|f\"(SELECT|INSERT|UPDATE|DELETE)|\+ *['\"] *(WHERE|AND|OR)" --include=*.{py,js,ts,php,rb,go,java} .
grep -rnE "ORDER BY .*\{|LIMIT .*\{|LIKE .*%s.*%" .
```

- Toda consulta usa **parâmetro** (`?`, `:nome`, `%s` do driver), nunca concatenação/f-string.
- `ORDER BY`, nome de coluna e direção vêm de **lista permitida**, não do usuário.
- `LIKE` escapa `%` e `_`.
- Também: injeção de comando (`os.system`, `subprocess(..., shell=True)`, `exec`, `eval`), de template (render de string do usuário), de cabeçalho/e-mail (quebra de linha em `Subject`/`To`), de caminho em consulta NoSQL (`$where`, `$ne` vindo do JSON).

Prova: teste com `' OR 1=1 --`, `1; DROP TABLE x`, `ORDER BY (SELECT 1)` e `{"$ne": null}` mostrando que nada vaza nem quebra (resposta igual à de entrada inválida). Ferramenta opcional **só na cópia local**: `sqlmap -u "http://127.0.0.1:PORTA/rota?x=1" --batch --level 2`.

## Etapa 2 — IDOR e isolamento entre contas

A falha mais comum em SaaS: trocar o ID na URL e ver o dado de outra pessoa.

1. Para cada rota que recebe ID (`/{id}`, `?id=`, corpo JSON), confira: a consulta filtra **pela conta/organização da sessão** (`WHERE id = ? AND dono_id = sessão`)? Ou só busca pelo ID e confia?
2. IDs sequenciais expostos? Prefira identificador aleatório (UUID/token) **além** do filtro — nunca no lugar dele.
3. Arquivos: download procura **só na pasta da conta da sessão**; nome de arquivo nunca vem do usuário (sem `../`).
4. Listagens, buscas, exportações, contadores e webhooks internos também filtram.
5. Papéis: usuário comum chamando rota de admin/dono recebe a mesma resposta de rota inexistente.

Prova (obrigatória): **teste com duas contas falsas** A e B. B tenta ler, editar, apagar e baixar cada recurso de A por todas as rotas → tudo 404/403, nada muda no banco. Melhor ainda: um **teste de varredura** que percorre todas as rotas da Etapa 0 com a conta B usando IDs de A.

## Etapa 3 — Entrada, sanitização e XSS

```
grep -rnE "innerHTML|outerHTML|insertAdjacentHTML|document\.write|dangerouslySetInnerHTML|v-html|\|safe|mark_safe|Markup\(|autoescape false|\{\{\{" .
```

- Templates com **autoescape ligado**; nada de `|safe` em dado do usuário.
- No JS, texto entra com `textContent`, nunca `innerHTML`.
- Markdown/HTML do usuário passa por **sanitizador com lista permitida** (ex.: bleach/DOMPurify) e links só `http(s):`/`mailto:`.
- Validação **no servidor** de tipo, tamanho, formato e faixa (esquema Pydantic/Zod/Joi). Limite de tamanho no corpo da requisição.
- Normalize texto Unicode (NFKC) antes de comparar; cuidado com homóglifos em domínios/nomes (IDNA).
- JSON embutido em página: escape `</script>`.

Prova: teste com `<script>alert(1)</script>`, `"><img src=x onerror=alert(1)>`, `javascript:alert(1)` em link, e texto gigante → aparece como texto, sem executar, sem 500.

## Etapa 4 — Login, senha, sessão e 2FA

- Senha: hash lento (Argon2id/bcrypt); política NIST 800-63B (12+ caracteres, sem regra de símbolo obrigatório, recusa senha vazada via k-anonimato do Have I Been Pwned e senha igual ao nome/e-mail).
- Mensagem igual para "usuário não existe" e "senha errada"; tempo parecido.
- Limite de tentativas por usuário+IP, por conta e por IP.
- Sessão: cookie trocado no login (sem fixação), expira, logout invalida no servidor; trocar senha/ligar 2FA derruba as outras sessões.
- 2FA/TOTP: segredo criptografado, código repetido recusado, recuperação com hash e uso único; **nenhum caminho pula o 2º fator** (recuperação, login social, API, "lembrar aparelho", flag ligada com segredo vazio).
- Passkeys/WebAuthn: origem e rpId fixos na configuração (não do cabeçalho Host), desafio único e curto preso à sessão, verificação de usuário exigida, contador anti-clonagem.
- Ações sensíveis (trocar senha/e-mail, desligar 2FA, cadastrar chave, área admin) pedem **reautenticação recente**.
- Recuperação de conta: token aleatório, uso único, expira, não revela se o e-mail existe, nunca rebaixa a segurança da conta.
- OAuth/login social: `state` validado, redirect_uri fixo, e-mail verificado pelo provedor.

Prova: testes para cada caminho alternativo tentando pular o 2º fator.

## Etapa 5 — CSRF, CORS e redirecionamentos

- CSRF em toda requisição que escreve (POST/PUT/PATCH/DELETE), inclusive JSON; `SameSite=Strict/Lax` ajuda mas não substitui.
- CORS: nada de `*` com credenciais; origem em lista fixa.
- Redirect (`next=`, `return_to=`, `url=`): só caminho interno começando com uma única `/`. Recuse `//`, `\`, `http:`, `javascript:`, `data:`, `%2F%2F`, `%5C`, `%09`, quebras de linha — validando **depois** de decodificar e normalizar, **no servidor**.
- Comparação de domínio por lista positiva (depois do domínio só fim, `/`, `?`, `#` ou porta).

Prova: lista de 20+ cargas de redirect aberto num teste parametrizado, todas indo para `/`.

## Etapa 6 — SSRF e chamadas para fora

Onde o servidor busca uma URL que veio do usuário (prévia de link, importar por URL, webhook configurável, avatar remoto)?

- Só `https`, porta 443, lista de domínios quando possível.
- Resolva o DNS e **recuse IP privado/local/metadados** (127.0.0.0/8, 10/8, 172.16/12, 192.168/16, 169.254.169.254, ::1, fc00::/7) — e confira de novo após redirect.
- Tempo limite, tamanho máximo, sem seguir redirect sem validar.

Prova: testes com `http://127.0.0.1`, `http://169.254.169.254/`, `http://[::1]`, domínio que resolve para IP interno.

## Etapa 7 — Upload e arquivos

- Tipo checado pelo **conteúdo** (bytes mágicos), não pela extensão nem pelo `Content-Type` enviado.
- Tamanho máximo; nome gerado pelo servidor; guardado **fora** da pasta pública; servido com `Content-Disposition: attachment` e `nosniff`.
- Nunca executar o que foi enviado (nada de `.php`, `.html`, `.svg` com script servido inline).
- Antivírus (ex.: ClamAV) **fora** do laço principal da aplicação (fila/thread) para não travar o servidor.
- Descompactação: limite de tamanho total e de arquivos (zip bomb), sem caminhos `../` (zip slip).
- Imagens: reprocessar e remover metadados (EXIF com localização).

Prova: teste enviando `shell.php` renomeado para `.jpg`, SVG com `<script>`, zip com `../`, arquivo gigante.

## Etapa 8 — Negação de serviço e limites

```
grep -rnE "re\.(compile|match|search|findall)\(|new RegExp\(|/.*(\.\*|\+)\)?(\+|\*)" .
```

- Regex com retrocesso catastrófico (`(a+)+`, `(.*)*`, alternâncias sobrepostas) → troque por varredura linear; teste com 20 mil caracteres e teto de tempo.
- Rate limit nas rotas caras (login, busca, IA, envio de e-mail, upload) e paginação com teto.
- Trabalho pesado (antivírus, PDF, IA, e-mail) fora do caminho síncrono da requisição.
- O limitador não pode **revelar** rotas escondidas (429 só onde a rota existe = vazamento).

## Etapa 9 — Segredos, criptografia e dependências

- Verificador de segredos no repositório e no **histórico** (ex.: gitleaks/trufflehog/detect-secrets).
- Segredos só em variável de ambiente/cofre; `.env` no `.gitignore`.
- Segredos dos usuários (ex.: chaves de API deles) criptografados com chave fora do banco; códigos curtos só com hash.
- Chaves de provedores com **mínimo privilégio** (chave restrita só com as permissões usadas).
- Comparações de token em tempo constante; aleatoriedade com gerador criptográfico.
- Dependências: travadas por versão/hash, verificador de vulnerabilidades (pip-audit, npm audit, Dependabot), nada abandonado.
- Git: e-mail privado (noreply), sem links simbólicos ou caminhos locais da máquina no commit.

## Etapa 10 — Regras de negócio, pagamentos e webhooks

- Webhook com **assinatura verificada** e **idempotência** (mesmo evento duas vezes não libera duas vezes).
- Preço, plano e quantidade vêm do servidor/provedor, nunca do formulário.
- Estados intermediários (pagamento pendente, troca de plano abandonada) não prendem a conta nem liberam acesso.
- Conciliação periódica entre banco e provedor, com alerta **sem repetição** (só divergência nova; dono marca "explicada" com motivo e auditoria).
- Testes com valor baixo e estorno reais antes de abrir para o público.
- Condições de corrida: duas requisições ao mesmo tempo não usam o mesmo crédito/cupom duas vezes (transação + trava).

## Etapa 11 — IA / LLM (OWASP Top 10 para LLM)

- **Prompt injection**: texto de documento, página ou usuário é dado, nunca instrução; ferramentas que a IA chama têm permissões mínimas e confirmação humana para ações de risco.
- A IA só acessa dados **da conta da sessão** (mesmo filtro da Etapa 2 em cada ferramenta).
- Saída da IA é tratada como entrada não confiável (escapar HTML, nunca executar, nunca virar SQL).
- Custos: limite por conta, teto mensal no provedor, rate limit.
- Chave de IA: prefira a pessoa trazer a própria (BYOK) guardada criptografada; o servidor guarda o mínimo.
- "Teste de configuração" de IA deve fazer uma chamada real mínima, não só validar a chave.
- Conteúdo gerado por IA rotulado como tal.

## Etapa 12 — Dados pessoais, logs e privacidade (LGPD)

- Coletar o mínimo; o que não é necessário **nunca é guardado**; migração de purga apaga o que expira.
- Logs sem senha, token, código, documento ou conteúdo de mensagem; em erro, só identificador e tipo.
- Busca por dado pessoal (nome, documento) exige login e finalidade aceita.
- Exportar e apagar a própria conta funciona de verdade.
- Respostas JSON com dado pessoal: `Cache-Control: no-store`.
- Admin grava auditoria (quem, quando, o quê — sem o dado sensível).

## Etapa 13 — Cabeçalhos, área admin e borda

- Cabeçalhos: CSP sem `unsafe-eval` nem inline, `frame-ancestors 'none'`, HSTS com preload, `nosniff`, `Referrer-Policy`, `Permissions-Policy`, COOP/CORP; cookies `__Host-`, `HttpOnly`, `Secure`, `SameSite`, **nome neutro** (nome de cookie não revela nada interno).
- Área admin invisível: para quem não é dono, **mesma resposta de página inexistente** em GET/HEAD/OPTIONS/POST, antes de CSRF e do limitador; ausente de robots, JS, CSS, sitemap e mensagens de erro.
- Documentação da API e páginas de debug fechadas em produção; erro sem stack trace.
- Servidor: firewall aceitando só a CDN nas portas web; SSH restrito; a aplicação só confia no IP do visitante informado pela CDN quando a conexão vem da CDN.
- Borda: rode a skill `auditoria-passiva-dominio` (DNS, DNSSEC, CAA, SPF/DKIM/DMARC, TLS, subdomínios, arquivos expostos).

## Etapa 14 — Varredura automática e relatório

1. Rode a skill oficial de auditoria da Cloudflare, se instalada: `npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit`.
2. Rode o Strix **só na cópia local**: `strix --target ./` (ou `http://127.0.0.1:PORTA`). Cada achado dele passa pelas regras desta skill (provar com teste antes de corrigir).
3. Deixe como teste permanente: varredura de rotas (cabeçalhos, CSRF, isolamento entre contas em todas as rotas), "impressão digital" (hash) do HTML das telas de login/2FA para qualquer mudança ser consciente.
4. Entregue o relatório final em `AUDITORIA.md`:
   - placar por etapa ✅/⚠️/❌;
   - achados por gravidade, cada um com teste, PR e status;
   - o que **não** foi verificado (ex.: pentest humano independente, infraestrutura do provedor);
   - próximas ações em ordem.

Lembre a pessoa: auditoria com IA não substitui pentest humano independente antes de dados reais sensíveis.
