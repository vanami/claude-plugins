---
name: payments
description: Pay suppliers through the bank from the Peppol portal (invoice.vanami.com) — list received invoices waiting to be paid, write an ISO 20022 payment file (pain.001) of those the user chooses to upload to the bank, and mark paid from the bank's statement (camt.054/053). Use when the user wants a payment run, to pay supplier invoices, or to reconcile what the bank paid.
---

# Payment run

1. Call `list_companies` (ask which company if there are several; pass its id as `company`).
   If the tools are missing or unauthorised, tell the user to run `/mcp`, choose `peppol` and
   sign in. Payment files need a plan that includes payments; on `PLAN_REQUIRED`, say so and
   stop.
2. Call `list_payables`. Show what waits to be paid, those due soonest first: supplier,
   number, amount, due date, where it is paid (`to`: bankgiro, plusgiro or IBAN) and the
   reference quoted, and whether it is already in a file (`ordered`, `file`). List those with
   a `problem` apart, with why they cannot be paid this way.
3. Ask the user which to pay, and on what day (each on its due date unless the user says
   otherwise). Point out anything unusual before they decide: an account the supplier has not
   used before, a large amount, a duplicate number.
4. Only on the user's clear choice, call `create_payment_file` with `documents` (their ids)
   and `date` if one was given. If the company has no paying account, it answers so: the
   account is set in the portal under Payments. Save the XML it answers to a file in the
   current directory named like `pain001-<date>.xml`, and tell the user to upload it to their
   bank, where they sign the payments. The invoices are marked as put in the file.
5. Afterwards, when the user has the bank's camt.054 notification or camt.053 statement, read
   it with `read_bank_statement` (`xml`, the file's text): it marks paid what the bank paid,
   and incoming payments of the company's own invoices too. Report what was marked paid and
   what was not matched, with why.

The bank, not this plugin, moves the money: nothing is paid until the user signs the file at
their bank. Never put an invoice in a file the user has not chosen.
