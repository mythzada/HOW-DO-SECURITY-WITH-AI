# 09 — Auditoria completa em etapas (do jeito que fizemos)

Você não precisa descobrir na unha onde estão as falhas. A skill **`auditoria-completa-em-etapas`** faz o agente de IA percorrer o sistema inteiro, em 14 etapas, do mesmo jeito que auditamos um SaaS real: cada achado provado com teste, corrigido em PR pequeno e explicado em português simples.

```
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill auditoria-completa-em-etapas
```

Depois, no Claude Code, dentro da **cópia local** do seu projeto: *"faça a auditoria completa em etapas"*.

## As etapas

| # | Etapa | O que procura |
|---|---|---|
| 0 | Mapa do sistema | Todas as rotas, quem acessa, zonas sensíveis |
| 1 | SQL injection e injeção | Consulta montada com texto, `ORDER BY` livre, comando de sistema, template, NoSQL |
| 2 | IDOR e isolamento | Trocar o ID e ver dado de outra conta; teste com duas contas falsas em todas as rotas |
| 3 | Entrada, sanitização e XSS | `innerHTML`, `|safe`, Markdown sem filtro, validação só no navegador, Unicode |
| 4 | Login, sessão e 2FA | Hash de senha, senha vazada, limite de tentativas, caminho que pula o 2º fator, passkey, recuperação |
| 5 | CSRF, CORS e redirect | Escrita sem CSRF, CORS aberto, redirect aberto com `//`, `\`, `%09` |
| 6 | SSRF | Servidor buscando URL do usuário que chega em IP interno ou metadados da nuvem |
| 7 | Upload | Tipo pela extensão, arquivo executável, zip bomb, zip slip, EXIF, antivírus travando o servidor |
| 8 | Negação de serviço | Regex lento (ReDoS), rota cara sem limite, limitador que revela rota escondida |
| 9 | Segredos e dependências | Segredo no código ou no histórico, chave com permissão demais, dependência vulnerável |
| 10 | Negócio e pagamentos | Webhook sem assinatura ou repetido, preço vindo do formulário, corrida no crédito, conciliação |
| 11 | IA / LLM | Prompt injection, IA lendo dado de outra conta, saída da IA executada, custo sem teto |
| 12 | Dados pessoais e logs | Guardar mais que o necessário, log com senha/token, busca por pessoa sem controle |
| 13 | Cabeçalhos, admin e borda | CSP, cookies, área admin que se denuncia, firewall da origem, DNS |
| 14 | Varredura e relatório | Skill da Cloudflare, Strix na cópia local, testes permanentes, relatório final |

## Regras que tornam a auditoria confiável

1. **Cópia local, nunca produção.** Ataque ativo só no seu computador.
2. **Achado sem teste não existe.** O teste falha antes da correção e passa depois.
3. **Uma correção por PR**, com explicação e plano de volta.
4. **Nunca afrouxar teste** para ficar verde.
5. **Segredo encontrado não é repetido**: diz onde está e manda trocar.
6. **Relatório em `AUDITORIA.md`**, para retomar de onde parou.

## O que a IA não substitui

Antes de guardar dados reais sensíveis, faça também um **pentest humano independente**. A auditoria com IA encontra muito, mas não assina sozinha.

Veja também: [06 — catálogo de métodos](06-METODOS-DE-SEGURANCA.md) · [08 — do ataque à defesa](08-DO-ATAQUE-A-DEFESA.md) · [07 — ferramentas de IA](07-FERRAMENTAS-DE-IA.md)
