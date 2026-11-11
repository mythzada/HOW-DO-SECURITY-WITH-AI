# 07 — Ferramentas de segurança com IA

Três ferramentas que se completam. **Use só no que é seu ou com autorização por escrito.**

| Ferramenta | O que faz | Onde roda |
|---|---|---|
| **Skill de auditoria da Cloudflare** (`security-audit`) | Lê o **código** e caça falhas em várias fases, com verificação independente de cada achado | No seu repositório, pelo agente de código |
| **Strix** | Agentes de IA que **executam** o sistema e tentam explorar falhas, com prova de conceito | **Só contra cópia local** ou ambiente de teste seu |
| **Nossas skills** (`passive-domain-audit` / `secure-ai-methodology`) | Reconhecimento passivo do domínio e o método de trabalho seguro | Domínio em produção (passivo) e dia a dia do código |

## 1. Skill de auditoria da Cloudflare

Licença MIT. Repositório: `github.com/cloudflare/security-audit-skill`.

```
npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
```

Depois, dentro do projeto, peça ao Claude Code: `security audit this codebase`.

- Faz reconhecimento da arquitetura, caça por classe de ataque (web, autenticação, IA/LLM, cadeia de suprimentos, nuvem, esgotamento de recursos, isolamento de dados…), tenta **refutar** cada candidato e gera `findings.json` e `REPORT.md`.
- Rodadas repetidas no mesmo repositório são cumulativas.
- Precisa de Node.js e, para executar código do alvo, de sandbox; sem sandbox, os achados ficam como "precisa validar".

**Como usamos:** cada achado vira uma tarefa, corrigida **um por vez**, com teste que prova a correção e revisão de segurança. Achado sem prova fica marcado como "não confirmado".

## 2. Strix

Licença Apache-2.0. Repositório: `github.com/UseStrix/strix`.

```
curl -sSL https://strix.ai/install | bash
export STRIX_LLM="provedor/modelo"     # veja a documentação do Strix
export LLM_API_KEY="sua-chave"           # nunca coloque no git
strix --target ./sua-aplicacao           # código local
```

- Precisa de **Docker** e de uma chave de API de modelo.
- **Ele ataca de verdade** o alvo. Aponte **só** para a sua cópia local (ex.: `http://localhost:8000` ou a pasta do projeto). **Nunca** contra produção, e nunca contra sistema de terceiros.
- Resultados em `strix_runs/`. Em automação: `strix -n --target ...` (sai com erro se achar falha).

**Como usamos:** suba uma cópia local com dados falsos, rode o Strix, e trate cada achado como os da auditoria: corrigir um por vez, com teste.

## 3. Ordem recomendada

1. `secure-ai-methodology` ligada desde o primeiro commit.
2. `passive-domain-audit` assim que o domínio estiver no ar (e depois todo mês).
3. Auditoria da Cloudflare no código a cada grande entrega.
4. Strix contra a cópia local antes de lançar e a cada mudança grande em login, pagamento ou dados.
5. Repita. Segurança é processo, não evento.
