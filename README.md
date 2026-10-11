# isms — Claude Code plugin for isms.vanami.com

Connects Claude Code to your ISMS at [isms.vanami.com](https://isms.vanami.com) and answers
the ISO/IEC 27001:2022 auditor's questions from your code, CI and infrastructure, with
evidence the auditor can verify.

```sh
claude plugin marketplace add vanami/claude-plugins
claude plugin install isms@vanami
```

Start `claude` in your repository and run `/isms:audit`. The first time, your browser opens:
sign in with Vanami ID and pick the company.

| Skill | What it does |
| --- | --- |
| `/isms:audit` | Goes through the open controls, collects evidence, proposes answers and writes the approved ones |
| `/isms:control A.8.13` | One control or clause, or an auditor's question |
| `/isms:status` | Audit dates, progress and what is left |
| `/isms:suppliers` | Supplier and asset registers from the code |

The assistant works with your role's permissions, asks before every write, never approves
documents and never deletes anything. Changes to controls are logged as made by the AI
assistant. It only reads your infrastructure, and asks before its first cloud CLI call.

To switch company, run `/mcp`, choose `isms`, clear the authentication and sign in again.
