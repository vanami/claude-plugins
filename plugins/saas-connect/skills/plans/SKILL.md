---
name: plans
description: Change an existing Vanami kit service's plans, prices, limits, roles or HQ24 business area — the manifest in the kit's database and the service's copy, Stripe through `accounts sync`, and the HQ24 products. Use when the user wants to change what a service costs, what a plan includes, or which business area its revenue is booked on.
---

# Change a service's plans

The kit keeps each service's manifest in its database, versioned (saas-kit doc 07 G198). The
service keeps its copy (`kit/product.yaml`, or wherever its README says) and mirrors plans
and roles in its UI (`lib/product.ts` in the template). All three change together. The kit
context is in `saas-kit/deploy/README.md`; `P` below stands for

```sh
P="kubectl --context <kit context> -n accounts exec -i deploy/accounts -- /accounts products"
```

Ask before every change outside the local repos, and never print a secret.

## 1. What changes

Read the kit's current version: `$P show <product>` (its first line names the version; `$P
history <product>` lists the earlier ones). If the service's copy differs from it, say so
first: the kit's is what runs. Show the manifest as a short table: plans, prices,
entitlements, roles, business area. Ask what should change, and propose. Then say what the
change means:

- **A new price amount** creates a new Stripe price that takes over the lookup key, and
  deactivates the old one (Stripe prices cannot change). Existing subscribers stay on the old
  price; only new purchases and plan changes get the new one. Moving existing subscribers is a
  separate decision: say so and do not do it here.
- **A plan an account holds cannot be removed**: the kit refuses (`in use`) while a live
  subscription or a plan granted without Stripe (`accounts grant`) is on it. Move the
  subscribers first (Stripe, or the kit's billing page) and revoke the grants, or keep the
  plan. Removing one interval of a plan is safe: its subscribers keep the plan.
- **A changed entitlement** takes effect for everyone on that plan within a token's
  lifetime (15 minutes). A lowered limit follows the manifest's `over_limit`.
- **A changed business area** (`dimensions`) updates the Stripe products' `hq24_dim_*`
  metadata, but HQ24 reads it **only when it creates a product**. Existing HQ24 products keep
  their old value: change them in HQ24 too (step 4). Invoices already booked keep the old value.
- **Plan keys** (`key:`) are stored on subscriptions: never rename one, add a new one.

## 2. The manifest

1. On a branch in the service, change `kit/product.yaml` and the UI's plan list.
2. Check it with every check the kit makes, saving nothing (`N` is the version `show` named):
   ```sh
   $P save <product> --expect N --check < kit/product.yaml
   ```
   Show the diff against `$P show <product>` and what the check said.
3. With a yes, save it (the person's name in `--by`, for the kit's audit log). Every replica
   serves it at once; no release:
   ```sh
   $P save <product> --expect N --by <who> --note "<what changed>" < kit/product.yaml
   ```
   `version conflict` means someone saved in between: show their version, and start over
   from step 1.
4. To undo a save, save the earlier version again: `$P show <product> --version N-1` into a
   file, then `save` it with the new `--expect`.

## 3. Stripe

```sh
kubectl --context <kit context> -n accounts exec deploy/accounts -- /accounts sync --dry-run
```

Show the dry run. With a yes, run it without `--dry-run`, then the dry run again: no changes.
Then, with a yes, open and merge the service's PR (its copy and its UI).

## 4. HQ24 (only when the business area changed)

1. The new value must exist: `finance.list_dimension_values` for `BUSINESS_AREA` with the
   HQ24 MCP; create it with `finance.create_dimension_value` on a yes.
2. Find the HQ24 products made from this service's Stripe products
   (`payments.list_gateway_product_links`) and, on a yes, set the new value on each
   (`product.update_product`). Without the HQ24 MCP, list what the user must change in HQ24.

## 5. Report

What changed where (the manifest's new version in the kit, PR links, Stripe prices created or archived, HQ24 products updated), and
what was deliberately left alone (existing subscribers on old prices, booked invoices).
