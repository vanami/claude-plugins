---
name: status
description: Show where a company stands in Vanami Procure (invoice.vanami.com) — what customers owe and what is overdue, supplier invoices waiting to be paid, documents waiting to be approved, and drafts. Use when the user asks how things stand, what needs doing, or for an overview of invoicing.
---

# Vanami Procure status

1. Call the `peppol` MCP server's `list_companies`. If the tools are missing or unauthorised, tell
   the user to run `/mcp`, choose `peppol` and sign in with their Vanami account. With more than
   one company, ask which one, unless the user named it; every other tool takes its id as
   `company`.
2. For that company, call `list_receivables`, `list_payables`, `list_approvals` and
   `list_documents` with `state` `draft`. A tool that answers `PLAN_REQUIRED` is not in the
   company's plan: say so in one line and go on.
3. Report briefly, in the user's language:
   - what customers owe per currency, how much of it is overdue, and the overdue invoices
     (number, customer, amount, days overdue, when last reminded), at most ten
   - supplier invoices waiting to be paid, those due within a week first, and any with a
     `problem` (such as no account to pay to)
   - documents waiting to be approved, and for whom
   - drafts, and any with validation errors
4. Offer what this plugin can do next: `/invoice:reminders` for the overdue invoices,
   `/invoice:payments` for a payment run, `/invoice:invoice` to write and send a new one.
   Approving is the user's own call: never approve anything unasked.
