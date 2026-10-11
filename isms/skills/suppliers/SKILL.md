---
name: suppliers
description: Fill the supplier and asset registers in the ISMS (isms.vanami.com) from the code and infrastructure — cloud providers, SaaS, payment and e-mail services, databases, buckets, repositories and domains. Use when the user wants the registers filled in or checked against reality.
---

# Suppliers and assets from the code

1. Read the current registers with `list_suppliers` and `list_assets` from the `isms` server,
   and `list_people` for owners.
2. Find suppliers, read-only: dependency manifests (SDKs for Stripe, Sentry, SendGrid, AWS,
   Google, OpenAI/Anthropic …), IaC providers and regions, DNS and mail records, CI services,
   hosting (Vercel, Netlify, Fly, Heroku), environment variable names in `.env.example` and
   deploy configs (names only, never values).
3. Find assets: databases, buckets, queues, clusters, repositories, domains, and the
   information they hold. Classify them (`public`, `internal`, `confidential`, `restricted`);
   personal data and customer data are at least `confidential`.
4. Present one table of suppliers and one of assets: new, already registered, and
   registered but no longer seen in the code. For suppliers give the service, the data
   shared, a proposed criticality and the control (usually A.5.19, A.5.23 for cloud). Write
   everything in Swedish. Leave contract status as `none` unless the user knows better, and
   note that DPAs and certificates should be checked.
5. Add or update what the user approves with `save_supplier` and `save_asset`. Do not
   create duplicates; update the existing row by its id. Never delete anything; list rows
   that look obsolete for the user to remove in the app.
