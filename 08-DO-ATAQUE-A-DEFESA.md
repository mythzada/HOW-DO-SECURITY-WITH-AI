# 08 — Do ataque à defesa (trilha em construção)

Cada tema de ataque que estudamos vira um guia de **como se defender**. Os temas seguem a ordem de um curso introdutório de hacking e pentest (estudamos pelo curso *Introdução ao Hacking e Pentest 3.0*, da Solyd Offensive Security — recomendado). Os laboratórios ofensivos ficam no curso; aqui fica a defesa aplicada.

| # | Tema de ataque | Como defendemos | Status |
|---|---|---|---|
| 1 | Carreira e ética | Escopo, autorização por escrito, lei (art. 154-A, LGPD), `security.txt` | ✅ README |
| 2 | Linux e terminal | Permissões mínimas (nada de `777`), usuários sem privilégio, atualizações | 🔜 guia |
| 3 | Reconhecimento (recon) | Reduzir o que aparece: IP escondido, DNS limpo, whois dedicado, robots sem segredos, cabeçalhos sem versão | ✅ [02](02-RECONHECIMENTO-PASSIVO.md) |
| 4 | Varredura e serviços (Nmap, Netcat) | Firewall só com portas necessárias, origem só aceita a CDN, SSH restrito, serviços sem banner | 🔜 guia + laboratório local |
| 5 | Exploits de serviços | Dependências atualizadas, imagens mínimas, nada exposto além do necessário | 🔜 guia |
| 6 | SQL Injection | Consultas parametrizadas/ORM, nunca concatenar SQL, menor privilégio no banco | 🔜 guia |
| 7 | XSS e roubo de cookies | CSP sem inline/eval, sem `innerHTML`, cookies HttpOnly/Secure/SameSite | ✅ parcial em [06](06-METODOS-DE-SEGURANCA.md) · 🔜 guia |
| 8 | LFI / leitura de arquivos | Nunca usar caminho vindo do usuário; lista positiva de arquivos; `.env`/`.git` bloqueados na borda | 🔜 guia |
| 9 | Upload de shell | Validar tipo pelo conteúdo, renomear, guardar fora da raiz web, antivírus, sem execução | 🔜 guia |
| 10 | Escalada de privilégio e pós-exploração | Processo sem root, segredos fora do disco do app, logs e alertas | 🔜 guia |
| 11 | Pivoting na rede | Rede privada, firewall entre serviços, banco sem acesso público | 🔜 guia |
| 12 | IDOR (bug bounty) | Toda consulta filtrada pela conta da sessão; teste com duas contas | ✅ parcial em [06](06-METODOS-DE-SEGURANCA.md) · 🔜 guia |
| 13 | SSRF | Lista positiva de destinos, bloquear IPs internos/metadados, sem seguir redirect cego | 🔜 guia |
| 14 | Tomada de contas | Passkey, 2FA, limites de tentativa, recuperação segura, avisos por e-mail | ✅ [06](06-METODOS-DE-SEGURANCA.md) |

**Regra do laboratório:** toda prática ofensiva roda numa **cópia local** do seu sistema ou nos laboratórios do curso — nunca em produção.
