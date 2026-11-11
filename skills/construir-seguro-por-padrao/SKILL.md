---
name: construir-seguro-por-padrao
description: Use sempre que for criar ou alterar qualquer app (inclusive "vibe code") — login, banco, API, upload, pagamento, IA, telas. Faz a IA montar tudo já seguro por padrão, com o mesmo nível de um SaaS real auditado, e não aceita entregar recurso sem os controles e o teste de segurança.
---

# Construir seguro por padrão

Você está construindo um app para a pessoa. Regra única: **todo recurso já nasce seguro**. Segurança não é etapa do fim; é o jeito de escrever cada linha. Seja persistente: não pule nenhum item abaixo por pressa, nem porque a pessoa não pediu. Se algo não puder ser feito, diga o que ficou faltando e o risco.

## 1. Primeira vez no projeto

1. Crie (ou complete) o `CLAUDE.md` do projeto com a seção **"Regras de segurança"** abaixo resumida, para que toda sessão futura siga as mesmas regras.
2. Configure desde o início: verificador de segredos antes do commit, `.env` no `.gitignore`, `.env.example` sem valores, dependências travadas, testes rodando com um comando só, e-mail privado no git.
3. Se o projeto já existe, rode a skill `auditoria-completa-em-etapas` antes de adicionar recursos.

## 2. Receita obrigatória por tipo de recurso

| Ao criar… | Já entra pronto com… | Teste que acompanha |
|---|---|---|
| **Qualquer consulta ao banco** | Parâmetros do driver/ORM (nunca concatenar texto); `ORDER BY`/colunas por lista permitida; filtro pela conta/organização da sessão | Carga `' OR 1=1 --` não vaza nada |
| **Rota com ID** | Busca com `id` **e** dono da sessão; ID aleatório (UUID) | Conta B não lê/edita/apaga/baixa nada da conta A (IDOR) |
| **Cadastro e login** | Argon2id/bcrypt; senha 12+ com recusa de senha vazada (k-anonimato HIBP); mensagem igual para usuário inexistente e senha errada; limite de tentativas por usuário+IP, conta e IP; cookie de sessão trocado no login | Força bruta bloqueada; mesma mensagem nos dois erros |
| **Sessão/cookies** | `__Host-`, `HttpOnly`, `Secure`, `SameSite=Lax/Strict`, nome neutro; expiração; logout invalida no servidor | Cookie antigo não funciona depois do logout |
| **2FA / passkey** | TOTP com segredo criptografado e código não reutilizável; códigos de recuperação com hash; passkey com origem/rpId fixos e desafio único; nenhum caminho pula o 2º fator | Recuperação/login social/API não entram sem 2º fator |
| **Ação sensível** (trocar senha/e-mail, apagar conta, desligar 2FA) | Reautenticação recente + e-mail de aviso | Sem reautenticação → recusado |
| **Formulário / rota que escreve** | CSRF; validação no servidor por esquema (tipo, tamanho, formato); limite do corpo | POST sem token CSRF → 403 |
| **Tela que mostra texto do usuário** | Autoescape do template; `textContent` no JS; Markdown/HTML por sanitizador com lista permitida | `<script>` aparece como texto |
| **Redirect (`next=`)** | Só caminho interno com uma única `/`, validado no servidor depois de normalizar | `//site.com`, `\`, `javascript:` → `/` |
| **Buscar URL do usuário** | Só https; recusa IP privado/local/metadados, inclusive após redirect; tempo e tamanho limitados | `http://169.254.169.254` recusado |
| **Upload** | Tipo pelo conteúdo; tamanho máximo; nome gerado; fora da pasta pública; download como anexo; antivírus fora do laço principal; EXIF removido; limites para zip | `.php` disfarçado de `.jpg` recusado |
| **Regex** | Sem retrocesso catastrófico; prefira varredura linear | 20 mil caracteres em menos de 1 s |
| **Rota cara** (busca, IA, e-mail, upload) | Rate limit; paginação com teto; trabalho pesado em fila | Excesso → 429 |
| **Pagamento** | Preço vem do provedor/servidor; webhook com assinatura e idempotência; estados pendentes não liberam acesso; trava contra uso duplo do crédito | Webhook repetido não libera duas vezes |
| **Recurso com IA** | Texto externo é dado, não instrução; ferramentas da IA só veem dados da conta da sessão e pedem confirmação para ações de risco; saída da IA escapada; teto de custo por conta | Documento com "ignore as instruções" não muda o comportamento |
| **Segredo** (chave de API etc.) | Só variável de ambiente/cofre; segredo do usuário criptografado com chave fora do banco; chave do provedor com mínimo privilégio | Verificador de segredos passa |
| **Dado pessoal** | Coletar o mínimo; nunca logar senha/token/documento; purga do que expira; exportar e apagar conta funcionam; `Cache-Control: no-store` | Log não contém o dado |
| **Área admin** | Para quem não é dono, resposta idêntica a página inexistente em todos os métodos; fora de robots/JS/sitemap; auditoria de cada ação | Usuário comum recebe o mesmo 404 |
| **Erro** | Página amigável sem stack trace; docs da API e debug fechados em produção | Erro 500 não mostra código |
| **Resposta HTTP** | CSP sem `unsafe-eval`/inline, `frame-ancestors 'none'`, HSTS, `nosniff`, `Referrer-Policy`, `Permissions-Policy`, COOP/CORP | Teste de cabeçalhos em todas as rotas |

## 3. Portão antes de dizer "pronto"

Nenhum recurso está pronto sem:

1. Os itens da receita aplicados (diga quais).
2. Os testes de segurança da receita escritos e passando — cole o comando e o resultado.
3. Os testes antigos passando; nenhum assert afrouxado (se mudou, declare por quê).
4. Revisão de segurança do diff: o que pode dar errado, para quem, e por que não dá.
5. Plano de volta.
6. Uma explicação curta em linguagem simples do que foi protegido.

Se a pessoa pedir para pular, explique o risco em uma frase e ofereça a versão segura mais simples. Não entregue a versão insegura em silêncio.

## 4. Antes de publicar

- Rode a skill `auditoria-completa-em-etapas` (cópia local) e a `auditoria-passiva-dominio` (domínio).
- CDN na frente, HTTPS forçado, firewall da origem aceitando só a CDN, SSH restrito.
- Recurso novo arriscado atrás de flag desligada até ser verificado.
- Dados reais sensíveis só depois de pentest humano independente.

## Estilo

Português curto. Ensine enquanto faz: em cada recurso, uma linha "isso protege contra…". Credenciais, pagamentos e cliques finais de criar/apagar/publicar ficam com a pessoa.
