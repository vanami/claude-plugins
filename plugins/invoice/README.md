# invoice — Claude Code plugin for invoice.vanami.com

Connects Claude Code to your Peppol portal at [invoice.vanami.com](https://invoice.vanami.com):
write and send e-invoices over Peppol, follow up what customers owe, and pay suppliers with a
payment file for your bank.

```sh
claude plugin marketplace add vanami/claude-plugins
claude plugin install invoice@vanami
```

Start `claude` and run `/invoice:status`. The first time, run `/mcp`, choose `peppol` and sign
in with your Vanami account: the sign-in page asks whether Claude Code may act for you.

| Skill | What it does |
| --- | --- |
| `/invoice:status` | What customers owe and what is overdue, what waits to be paid or approved, drafts |
| `/invoice:invoice <what>` | Writes an invoice or credit note from a description, checks it, and sends it when you say so |
| `/invoice:reminders` | Overdue customer invoices: marks paid what was paid, mails the reminders you approve |
| `/invoice:payments` | A payment run: a pain.001 file of the supplier invoices you choose, for your bank; then marks paid from the bank's camt.054/053 |

The assistant works with your role's permissions in the portal and asks before anything is
sent, paid, approved or mailed. What it does is in the portal's activity log, marked with its
name. You can see and remove it under Settings → AI assistant in the portal, which also shows
how to connect claude.ai, ChatGPT and others to the same server
(`https://invoice.vanami.com/mcp`).

To sign in to another company's organisation, sign out and in again:

```sh
claude mcp logout plugin:invoice:peppol
claude mcp login plugin:invoice:peppol
```
