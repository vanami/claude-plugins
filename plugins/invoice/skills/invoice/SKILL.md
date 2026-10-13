---
name: invoice
description: Write an e-invoice or credit note in the Peppol portal (invoice.vanami.com) from a description, check it against the Peppol rules, and send it over Peppol once the user says so. Use when the user wants to invoice a customer, bill for work or goods, or credit an invoice.
---

# Write and send an invoice

The user's request is in the arguments, such as "invoice Acme for 8 hours of consulting at
1 200 kr" or "credit invoice 10042".

1. Call `list_companies` (ask which company if there are several; pass its id as `company` to
   every other tool). If the tools are missing or unauthorised, tell the user to run `/mcp`,
   choose `peppol` and sign in.
2. Find the customer: `list_customers` with `q` for the name. If it is not there, ask for its
   organisation or VAT number and look it up with `lookup_participant_by_vat` (or
   `lookup_participant`) to get its Peppol ID; a customer not on Peppol cannot be sent to over
   it, so say so.
3. A credit note for a sent invoice: find it with `list_documents` (`q` the number,
   `direction` `outbound`) and call `create_credit_note`, then go to step 5.
4. Otherwise call `create_draft` with `document`: the customer as `buyer`, the lines (name,
   quantity, unit, unit price, VAT), and whatever else the user said (their reference, the
   order number, the period). Leave out what the company's defaults fill in — the seller, the
   number, the dates, the payment account and terms — unless the user said otherwise. Do not
   invent amounts, VAT rates or references: ask.
5. Call `check_document` for the draft and show the user, briefly: customer, lines, totals with
   VAT, due date, payment details, and any errors or warnings. Put errors right with
   `update_draft` and check again.
6. Ask the user whether to send it. Only on a clear yes, call `send_document`. If it answers
   `APPROVAL_REQUIRED`, the company wants it approved first: offer `request_approval` instead.
   Then tell the user the state (`queued`; `get_document` later tells whether it was delivered)
   and offer to mail the customer a copy with `mail_document` (`kind` `copy`).

What is sent cannot be taken back; a mistake in a sent invoice is put right with a credit note.
