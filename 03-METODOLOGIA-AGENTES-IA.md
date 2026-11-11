# 03 — Metodologia: uma equipe de agentes de IA

## Papéis

| Papel | Faz | Não faz |
|---|---|---|
| **Coordenador** (IA) | Distribui tarefas, decide trade-offs, confere evidências, faz merge, registra lições | Não deixa nada entrar sem prova |
| **Engenheiro** | Backend, migrações, correções | Não toca em arquivos da "cerca" |
| **Front** | Telas, UX, JS, acessibilidade | Não cria rota de servidor sozinho |
| **Segurança** | Auditoria, VERIFY de cada PR no head exato | Não sonda produção |
| **Explica Tudo** | Veredito em linguagem simples (verde/amarelo/vermelho) e resumo para o dono | Não aprova sozinho |
| **Agente de código "da cerca"** | Única IA autorizada a mexer em autenticação, sessão, cobrança, cálculos críticos | — |
| **Dono (humano)** | Decisões de produto, dinheiro, dados; cliques finais de criar/apagar/pagar; logins | Nunca passa senha para bot |

## Fluxo de um PR

```
CLAIM → código + teste local → PR → Segurança VERIFIED (no head exato)
      → Explica VERDE → Coordenador confere head + resultados → merge
```
- Mudou um commit? O VERIFIED anterior cai.
- Resultado de teste **colado**, com o comando. Sem colar, não entra.

## A "cerca"

Arquivos de autenticação, segurança, recuperação de conta, cobrança e cálculo só são alterados pelo agente designado. Os outros listam `arquivo:linha` e repassam.

## Lições (exemplos reais)

- **Base desatualizada:** aceitar ou não é decisão do coordenador, registrada.
- **Sem teste local colado, sem merge.**
- **Servidor e tela que dependem um do outro entram no mesmo PR** (para nada ir ao ar quebrado).
- **Nunca afrouxar um teste num redesign.** Assert alterado precisa ser declarado no PR.
- **Rode a suíte de testes "fixados" antes de pedir merge** (um merge deixou a main vermelha porque isso não foi feito).
- **Prova de produção ≠ prova local.** Diga sempre o que *não* foi verificado.
- **Paleta semântica é contrato:** vermelho é vermelho (não rosa); estados vêm do servidor.

## Economia

- Mensagens entre IAs curtas e em inglês técnico; com o dono, português claro.
- Esteira de CI manual: os agentes testam localmente e o coordenador faz merge pela evidência.
- Acompanhe o uso do plano e defina um teto (ex.: 95%).

## Rondas

O coordenador revisa todos os agentes em intervalos fixos (ex.: 45 min): lê **todas** as mensagens desde a última leitura, faz merges prontos, destrava decisões e atualiza uma lista viva de pendências.
