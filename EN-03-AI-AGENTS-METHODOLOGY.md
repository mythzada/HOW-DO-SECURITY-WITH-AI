# 03 — Methodology: a team of AI agents

## Roles

| Role | Does | Doesn't |
|---|---|---|
| **Coordinator** (AI) | Assigns work, decides trade-offs, checks evidence, merges, records lessons | Lets nothing in without proof |
| **Engineer** | Backend, migrations, fixes | Touch "fenced" files |
| **Front-end** | Screens, UX, JS, accessibility | Create server routes alone |
| **Security** | Audits; VERIFY of each PR at the exact head | Probe production |
| **Explainer** | Plain-language verdict (green/yellow/red) and summary for the owner | Approve alone |
| **"Fence" code agent** | The only AI allowed to touch auth, sessions, billing, critical math | — |
| **Owner (human)** | Product, money and data decisions; final create/delete/pay clicks; logins | Never hands passwords to a bot |

## PR flow
```
CLAIM → code + local tests → PR → Security VERIFIED (exact head)
      → Explainer GREEN → Coordinator checks head + results → merge
```
- New commit? The previous VERIFIED is void.
- Test results **pasted**, with the command. No paste, no merge.

## The fence
Auth, security, account recovery, billing and calculation files are changed only by the designated agent. Others report `file:line` and hand off.

## Lessons (real examples)
- Accepting a stale base is the coordinator's recorded decision.
- No pasted local test, no merge.
- Server and UI that depend on each other ship in **one** PR.
- Never weaken a test in a redesign; any changed assert is declared in the PR.
- Run the pinned/snapshot suite before asking to merge (a merge once turned main red because of this).
- Local proof ≠ production proof. Always state what was **not** verified.
- Semantic palette is a contract: red is red; states come from the server.

## Economy
- Agent-to-agent messages short and technical; with the owner, clear and simple.
- Manual CI: agents test locally, the coordinator merges on evidence.
- Track plan usage and set a ceiling.

## Rounds
The coordinator reviews every agent at fixed intervals: reads **all** messages since the last check, merges what's ready, unblocks decisions and updates a living to-do list.
