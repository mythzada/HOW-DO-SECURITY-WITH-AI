# HOW DO SECURITY WITH AI

🇺🇸 [English](README.md) · 🇧🇷 **Português**

> Como construir, operar e proteger um sistema real usando IA como equipe — do código à borda da rede.

Este repositório é o caderno de campo de um projeto real: um SaaS jurídico brasileiro construído e protegido com ajuda de agentes de IA, Cloudflare e muito estudo de segurança ofensiva e defensiva. Tudo aqui foi feito na prática, testado e verificado.

**Status:** código aberto, em construção contínua. Cada aula estudada e cada defesa aplicada viram um guia novo aqui.

---

## ⚖️ Antes de tudo: ética e lei

- Teste **só** o que é seu ou o que você tem **autorização por escrito** para testar.
- No Brasil, invadir dispositivo alheio é crime (**art. 154-A do Código Penal**, Lei 12.737/2012). Dados pessoais seguem a **LGPD** (Lei 13.709/2018).
- Ferramentas ativas (Nmap, Nikto, sqlmap, gobuster, hydra, agentes de pentest) rodam **apenas em cópia local** ou laboratório. Em produção, só reconhecimento passivo.
- Achou falha em sistema de terceiros? Procure o `security.txt` do site e reporte de forma responsável.

---

## 🗺️ Mapa do repositório

| Arquivo | O que tem |
|---|---|
| [01-CAMADAS-DE-DEFESA.md](01-CAMADAS-DE-DEFESA.md) | Defesa em profundidade: DNS, borda (Cloudflare), servidor, aplicação, dados |
| [02-RECONHECIMENTO-PASSIVO.md](02-RECONHECIMENTO-PASSIVO.md) | Comandos para auditar o próprio domínio como um atacante veria — e o que cada resultado significa |
| [03-METODOLOGIA-AGENTES-IA.md](03-METODOLOGIA-AGENTES-IA.md) | Como coordenar vários agentes de IA (código, segurança, front, explicação) sem bagunça |
| [04-NAVEGADOR-COM-IA.md](04-NAVEGADOR-COM-IA.md) | Usar uma extensão de IA no Chrome para tarefas manuais em painéis, com regras de segurança |
| [05-TRILHA-DE-ESTUDO.md](05-TRILHA-DE-ESTUDO.md) | O que estudar, em que ordem, e onde |
| [CHECKLIST.md](CHECKLIST.md) | Checklist resumido para aplicar em qualquer projeto |

---

## 🧠 A mentalidade

1. **Olhe seu sistema com os olhos de quem ataca.** Rode as mesmas ferramentas que um atacante rodaria, no seu próprio domínio.
2. **Defesa em camadas.** Nenhuma proteção sozinha basta. Se uma falha, a próxima segura.
3. **IA acelera, humano decide.** Agentes escrevem, testam e auditam; decisões de risco, dinheiro e dados ficam com o dono.
4. **Verifique tudo.** Nada vai para produção sem teste local colado, revisão de segurança e veredito claro.
5. **Cada erro vira lição.** Erros de agentes (e humanos) viram regras numeradas que nunca mais se repetem.
6. **Custo baixo é decisão de projeto.** Testes locais, esteiras manuais e plano gratuito bem configurado vão longe.

---

## 🧰 Ferramentas usadas

| Frente | Ferramentas |
|---|---|
| Código e revisão | Claude Code, agentes Grok especializados, GitHub (PRs, revisão) |
| Coordenação | Um agente coordenador que distribui, verifica e faz merge |
| Borda / rede | Cloudflare (DNS, WAF, regras, rate limit, HSTS, TLS, DNSSEC, CAA, DMARC) |
| Servidor | DigitalOcean (firewall de nuvem) |
| Pentest / estudo | Kali Linux em VirtualBox, `dig`, `whois`, `curl`, `openssl`, Google dorks |
| Testes com IA | Strix (agentes de IA open source para testes de segurança) — **só em ambiente local autorizado** |
| Auditoria | Skill de auditoria de segurança da Cloudflare rodando em agente de IA |
| Navegador | Extensão de IA no Chrome para operar painéis (com o humano no controle) |

---

## 🚀 Instale as skills no seu Claude Code

Cole no terminal do seu projeto:

```
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill metodologia-seguranca-com-ia
npx skills add https://github.com/mythzada/HOW-DO-SECURITY-WITH-AI --skill auditoria-passiva-dominio
```

| Skill | O que faz |
|---|---|
| **metodologia-seguranca-com-ia** | Faz o Claude Code trabalhar com você do jeito deste guia: ensina enquanto faz, defesa em camadas, PR com teste e revisão de segurança, nada de "vibe code" sem verificação |
| **auditoria-passiva-dominio** | Audita o seu domínio como um atacante faria no reconhecimento (DNS, e-mail, TLS, cabeçalhos, subdomínios, arquivos expostos), só com comandos passivos, e explica cada achado em português |

---

## ⭐ Recomendações

- **Claude Code** — agente de código no terminal; use com as skills acima.
- **Claude in Chrome** (extensão oficial) — para a IA operar painéis (Cloudflare, registrador, GitHub) enquanto você acompanha. Veja as regras em [04-NAVEGADOR-COM-IA.md](04-NAVEGADOR-COM-IA.md).
- **Cloudflare (plano gratuito)** — DNS, WAF, HSTS, DNSSEC, CAA, DMARC.
- **Skill de auditoria de segurança da Cloudflare** — `npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit`
- **Kali Linux em VirtualBox** — laboratório para estudar.
- **Curso introdutório de hacking e pentest da Solyd** — boa porta de entrada em português.

---

## 🛣️ Próximos guias

- [ ] Authenticated Origin Pulls (mTLS entre CDN e servidor)
- [ ] Cloudflare Access / Zero Trust para áreas administrativas
- [ ] Laboratório local: Nmap, Netcat e agentes de pentest contra uma cópia do seu sistema
- [ ] Monitoramento e resposta a incidentes
- [ ] Mais skills

Contribuições são bem-vindas: abra uma issue ou PR.

---

Licença MIT. Feito com estudo, café e IA no Brasil. 🇧🇷
