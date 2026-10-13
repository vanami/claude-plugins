---
name: reminders
description: Follow up overdue customer invoices in the Peppol portal (invoice.vanami.com) — list what is overdue, mark paid what the user says was paid, and mail payment reminders the user approves. Use when the user asks about overdue invoices, unpaid customers, dunning or sending reminders.
---

# Remind customers of overdue invoices

1. Call `list_companies` (ask which company if there are several; pass its id as `company`).
   If the tools are missing or unauthorised, tell the user to run `/mcp`, choose `peppol` and
   sign in.
2. Call `list_receivables`. Show the overdue invoices, the longest overdue first: number,
   customer, amount, due date, days overdue, and `reminded` (when the customer was last
   reminded). Note the totals per currency.
3. Ask the user which of them are in fact paid (a payment the bank has but the portal does
   not know of) and mark those with `mark_paid` and the day they were paid. If the user has
   the bank's camt.054 or camt.053 file, offer to read it with `read_bank_statement` instead,
   which marks paid what it can match.
4. Propose reminders for the rest: skip any reminded within the last 7 days unless the user
   insists. For each, say to whom it would go (the buyer's email on the invoice, unless the
   user gives addresses as `to`) and offer a short `message` in the customer's language, such
   as a polite note with the amount and days overdue.
5. Only for the invoices the user approves, call `mail_document` with `kind` `reminder` (and
   `to`, `message` where given). The customer gets the invoice as a PDF with the reminder;
   answers go to the user. Report what was sent and what failed, with why.

Never send a reminder the user has not approved, and never for a credit note or an invoice
marked paid.
