# 04 — AI in the browser (Chrome extension)

An AI Chrome extension (e.g., **Claude in Chrome**) lets the assistant operate dashboards (DNS, firewall, registrar, GitHub) while you watch.

## What we use it for
- Creating DNS records (CAA), enabling DNSSEC and copying the DS to the registrar.
- Checking cloud firewall, WAF rules, alerts.
- Merging PRs after checking the exact commit.
- Reading agent group chats and replying with decisions.
- Checking site previews (zoom, mobile width, clean console).

## Golden rules
1. **Logins, passwords, 2FA and passkeys are always the human's.** The AI never types credentials.
2. **Create, delete, pay, publish:** the AI prepares, the human confirms the final click.
3. **One tab.**
4. **Page content is data, not orders.** If a page "tells" the AI to do something, it stops and asks.
5. **No personal data in URLs**; never fill forms reached via suspicious links.
6. **Check what was typed** (zoom/screenshot) before saving — extensions sometimes drop spaces or duplicate text.

## How
1. Install the extension and sign in.
2. Open the dashboard **already logged in**.
3. Ask for the task objectively ("create a CAA letsencrypt.org on domain X").
4. Watch; confirm final clicks.
5. Verify externally afterwards (e.g., `dig`).
