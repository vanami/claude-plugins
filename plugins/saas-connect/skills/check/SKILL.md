---
name: check
description: Check that a Vanami kit service is wired up end to end — its manifest loaded by the kit, Stripe in step with the catalog, the HQ24 business area, the kit's SDK, the domain, sign-in and the kit's webhooks. Use when the user asks whether a service is set up right, after a deploy, or when sign-in, plans or billing misbehave.
---

# Check a service's chain

Takes a product id (ask if none was given). Read-only: report findings and propose fixes,
but change nothing without a yes. Never print a secret.

Run the checks, then report each as **ok**, **wrong** (with what was seen) or **not checked**
(with why), in one table, followed by what to do about each wrong one.

1. **Manifest:** `saas-kit/products/<product>.yaml` exists on saas-kit's main branch, and
   the service's copy says the same (diff plans, prices, roles, domain, webhooks).
2. **Kit loaded it:** the kit's start-up log (`kubectl --context <kit context> -n accounts
   logs deploy/accounts`, context from `saas-kit/deploy/README.md`) shows `accounts listening`
   with the product count, and no catalog error, `missing: [...]` (Stripe key permissions) or
   drift.
3. **Stripe:** `/accounts sync --dry-run` in the kit's pod reports only `*.ok` for this
   product.
4. **HQ24:** with the HQ24 MCP, the manifest's `BUSINESS_AREA` value exists
   (`finance.list_dimension_values`). If a payment gateway is connected
   (`payments.list_gateways`), the service's Stripe products are linked
   (`payments.list_gateway_product_links`) and no document waits in review for it
   (`payments.list_gateway_documents` with `status: review`).
5. **SDK:** the service takes the kit's SDKs as dependencies (`@vanami/accounts` in
   `package.json`, `github.com/vanami/accounts-sdk/go` in a Go backend's `go.mod`), with no
   vendored copy left, and `.github/dependabot.yml` covers them. Compare the version in the
   lockfile with the latest release (`npm view @vanami/accounts version`); behind it, point to
   the open Dependabot PR or report an update to do.
6. **Domain:** `https://<domain>` answers over TLS; `/auth/kit/start` redirects to
   `https://accounts.vanami.com/authorize` with `client_id=<product>`.
7. **Webhooks:** a POST without a valid signature to the manifest's `webhooks` URL answers
   400 (the template's `bad signature`). 503 means `KIT_EVENTS_SECRET` is not set; 2xx or 404
   is wrong.
8. **Environment:** on Vercel (`vercel env ls`), production has `KIT_ISSUER`,
   `KIT_CLIENT_ID=<product>`, `KIT_EVENTS_SECRET`, `DATABASE_URL` and `APP_URL`. Check names
   only, never values.

Offer to fix what is wrong, one item at a time.
