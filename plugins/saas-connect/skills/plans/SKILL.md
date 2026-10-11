---
name: plans
description: Change an existing Vanami kit service's plans, prices, limits, roles or HQ24 business area — the manifest in saas-kit and the service's copy, Stripe through `accounts sync`, and the HQ24 products. Use when the user wants to change what a service costs, what a plan includes, or which business area its revenue is booked on.
---

# Change a service's plans

A service's manifest lives in `saas-kit/products/<product>.yaml`; the service keeps a copy
(`kit/product.yaml`, or wherever its README says) and mirrors plans and roles in its UI
(`lib/product.ts` in the template). All three change together.

Ask before every change outside the local repos, and never print a secret.

## 1. What changes

Read the current manifest and show it as a short table: plans, prices, entitlements, roles,
business area. Ask what should change, and propose. Then say what the change means:

- **A new price amount** creates a new Stripe price that takes over the lookup key, and
  deactivates the old one (Stripe prices cannot change). Existing subscribers stay on the old
  price; only new purchases and plan changes get the new one. Moving existing subscribers is a
  separate decision: say so and do not do it here.
- **Never remove a plan that has subscribers.** A plan the catalog no longer knows grants the
  free plan (saas-kit CLAUDE.md rule 7), while Stripe goes on charging for it. `accounts sync`
  leaves its Stripe product and price as they are. Move the subscribers first (Stripe, or the
  kit's billing page), or keep the plan. Removing one interval of a plan is safe: its
  subscribers keep the plan.
- **A changed entitlement** takes effect for everyone on that plan within a token's
  lifetime (15 minutes). A lowered limit follows the manifest's `over_limit`.
- **A changed business area** (`dimensions`) updates the Stripe products' `hq24_dim_*`
  metadata, but HQ24 reads it **only when it creates a product**. Existing HQ24 products keep
  their old value: change them in HQ24 too (step 4). Invoices already booked keep the old value.
- **Plan keys** (`key:`) are stored on subscriptions: never rename one, add a new one.

## 2. The manifest

1. On a branch in saas-kit, change `products/<product>.yaml`. Run
   `go test ./internal/catalog/...` (`TestShippedManifestsLoad` loads it as the server does).
2. Make the same change in the service's copy and in its UI's plan list, on a branch there.
3. With a yes, open both PRs. Merge saas-kit first, then roll the kit out
   (`saas-kit/deploy/README.md`: release, then `rollout restart deploy/accounts`) and check the
   start-up log for `accounts listening` and no catalog error.

## 3. Stripe

```sh
kubectl --context <kit context> -n accounts exec deploy/accounts -- /accounts sync --dry-run
```

Show the dry run. With a yes, run it without `--dry-run`, then the dry run again: no changes.
Then merge the service's PR.

## 4. HQ24 (only when the business area changed)

1. The new value must exist: `finance.list_dimension_values` for `BUSINESS_AREA` with the
   HQ24 MCP; create it with `finance.create_dimension_value` on a yes.
2. Find the HQ24 products made from this service's Stripe products
   (`payments.list_gateway_product_links`) and, on a yes, set the new value on each
   (`product.update_product`). Without the HQ24 MCP, list what the user must change in HQ24.

## 5. Report

What changed where (PR links, Stripe prices created or archived, HQ24 products updated), and
what was deliberately left alone (existing subscribers on old prices, booked invoices).
