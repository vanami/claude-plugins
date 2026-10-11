---
name: new
description: Create a new Vanami SaaS service on the Vanami kit from a name and its plans — the repo from the kit's template, the product manifest registered in saas-kit, Stripe products and prices, the HQ24 business area, the database, deployment on Vercel and the domain. Use when the user wants to start, scaffold or launch a new SaaS, app or service on the kit.
---

# A new service on the Vanami kit

The kit (`hallsten/saas-kit`, running at `accounts.vanami.com`) owns sign-in, organisations,
teams, plans, Stripe and the billing page. A service keeps only its own data, keyed by the
kit's `account_id`. This skill takes the user from an idea to a deployed service that signs in,
limits by plan, sells through Stripe and books its revenue in HQ24.

The source of truth is the user's local clone of saas-kit, not this skill. Read these before
step 2, and follow them where they are more specific or newer than this text:

- `saas-kit/template/README.md`: the template and the steps for a new service
- `saas-kit/products/example.yaml.sample`: every manifest field
- `saas-kit/deploy/README.md`: the kit's cluster context and operations
- `saas-kit/CLAUDE.md`: the kit's rules

## Rules

- **Ask before every step that changes something outside the new repo:** creating a GitHub
  repo, opening or merging a PR in saas-kit, `accounts sync` without `--dry-run`, creating
  a Vercel project or env var, DNS, and HQ24. Show what will happen first.
- **Secrets never pass through the conversation.** Pipe them from where they are made to
  where they go (step 6). Never print them, never commit them.
- **Plans and prices are the user's decision.** Propose, then wait for a clear yes before
  `accounts sync`, which creates real Stripe products and prices.
- Identifiers are English. Swedish lives only in `sv` texts. Talk to the user in their language.
- Work in thin steps and report after each: what was done, what was checked, what is next.

## 0. Prerequisites

Check, and stop with a clear list of what is missing:

- `saas-kit` cloned next to where the service will live (`../saas-kit`), or ask for its path.
  It is private: the user needs access.
- `gh` signed in with access to `hallsten`, `node` 22+, `npm`, `docker` (for the end-to-end test).
- `kubectl` with the kit's cluster context from `saas-kit/deploy/README.md`. Check with
  `kubectl -n accounts get deploy accounts`.
- `vercel` CLI signed in (or the Vercel MCP), when deploying to Vercel.
- An HQ24 MCP server (`finance.*` tools) for step 5. Without it, the user does step 5 in HQ24.

## 1. The brief

Ask in one message, with a proposal for each, and wait for the answers:

1. **Product id** (`product`): short, English, lowercase, e.g. `booking`. It is the OIDC
   client id, `KIT_CLIENT_ID`, the Stripe metadata and the repo name. It cannot be changed later.
2. **Name** in English and Swedish, and one sentence on what it does.
3. **Domain**: default `<product>.vanami.com`.
4. **Plans**: a free plan (or none: then the service is paid only and data is deleted
   `retention_days` after the plan ends), and paid plans with monthly and yearly prices
   (minor units, currency, VAT added by Stripe Tax), entitlements (limits as numbers,
   features as booleans), optionally `trial_days`, seats and usage meters.
   See `references/manifest.md` for the choices.
5. **Roles and permissions** in the service: default `admin` (everything), `member`, `viewer`.
6. **Business area** in HQ24 (`BUSINESS_AREA`): a short code, e.g. `BOOKING`.
7. **Brand colour** and, optionally, a logo URL.
8. **Database**: where Postgres runs (e.g. Neon through Vercel's marketplace, or an existing
   server). Ask; do not pick one silently.

## 2. The repo

1. From the folder that holds `saas-kit`, copy the template without build output, and start a
   repo:
   ```sh
   cp -R saas-kit/template <product> && cd <product>
   rm -rf node_modules .next tsconfig.tsbuildinfo
   git init -b main && npm install
   ```
   The kit's SDKs come from npm (`@vanami/accounts`, `@vanami/accounts-react`), and the
   template's `.github/dependabot.yml` proposes their updates; keep both.
2. Fill in `kit/product.yaml` from the brief (`references/manifest.md`), and mirror the plans
   and roles in `lib/product.ts`. Set the title in `app/layout.tsx` and the texts.
3. Leave the example feature (`item`) for now. It proves the plan and permission checks
   until the service's own feature replaces it in step 9.
4. Check: `npm run typecheck && npm test`.
5. With a yes, create the private repo and push:
   `gh repo create hallsten/<product> --private --source . --push`.

## 3. Register it in the kit

1. In saas-kit, on a branch, copy the manifest to `products/<product>.yaml`. Keep the
   header comment pointing at the service's repo.
2. Validate it before the PR: `go test ./internal/catalog/...` in saas-kit.
   `TestShippedManifestsLoad` loads every file in `products/`, as the server does at start;
   a manifest that fails there would stop the kit from starting.
3. With a yes, open the PR (title `products: <Name> (<product>)`), wait for green CI, and merge
   when the user says so.
4. Roll the kit out so it loads the manifest (the release workflow pushes the image on
   merge; wait for it):
   ```sh
   kubectl --context <kit context> -n accounts rollout restart deploy/accounts
   kubectl --context <kit context> -n accounts rollout status deploy/accounts
   ```
   The log line `accounts listening` shows `products: <n>`, one more than before. An error
   naming the manifest means it did not load: fix it in a new PR.

## 4. Stripe

```sh
kubectl --context <kit context> -n accounts exec deploy/accounts -- /accounts sync --dry-run
```

Show the dry run: products, prices, metadata `hq24_dim_BUSINESS_AREA`. With a yes, run it
without `--dry-run`. Run the dry run again: it must show no changes.

## 5. HQ24

The revenue is booked on the business area. Its value must exist in HQ24 **before** the
first invoice, or the invoice waits in HQ24's review queue.

- With the HQ24 MCP: `finance.list_dimension_values` for `BUSINESS_AREA`; if the code is
  missing, and the user says yes, `finance.create_dimension_value`.
- Otherwise, tell the user to add it in HQ24 under the ledger's dimensions.

HQ24 creates its own product from each Stripe product automatically when Stripe is connected
as the payment gateway in HQ24.

## 6. Database and deployment

1. **Database:** an empty Postgres database and a role that owns it. The app creates its
   schema at start (`db/migrations.ts`).
2. **Vercel** (default; the template also runs as a container, see its README): with a yes,
   create the project from the repo, then set the variables for production:
   - `KIT_ISSUER=https://accounts.vanami.com`
   - `KIT_CLIENT_ID=<product>`
   - `APP_URL=https://<domain>`
   - `DATABASE_URL`: from the database provider or the Vercel integration, never typed into the chat.
   - `KIT_EVENTS_SECRET`: piped, never shown:
     ```sh
     kubectl --context <kit context> -n accounts exec deploy/accounts -- /accounts webhook-secret <product> \
       | vercel env add KIT_EVENTS_SECRET production
     ```
3. **Domain:** add `<domain>` to the Vercel project. Vercel shows a project-unique CNAME
   target. `vanami.com` is in Route 53, managed in Terraform the same way as
   `isms.vanami.com`: find that record in the infra repo and add one beside it, as a PR, with
   a yes. Vercel issues the certificate once DNS resolves.
4. Deploy (`git push`, or `vercel --prod`), and wait for the build.

## 7. Check the chain

Run `/saas-connect:check <product>`, or the same checks by hand:

- `https://<domain>` answers, and *Sign in* goes to the kit's sign-in page with the service's
  name and colour.
- After signing in, the app page shows the free plan; the account page shows plan, team and
  the upgrade buttons.
- An upgrade opens Stripe Checkout with the right price. In Stripe test mode use
  `4242 4242 4242 4242`. Back in the service, the plan changes within seconds, and
  *Manage billing* opens the kit's billing page for this service.
- The kit's webhook reached the service: the service's log shows `account.created` applied.

Report each check as passed or failed, with what was seen.

## 8. Update the README of the service

Replace the template's README with the service's own: what it does, the plans, how to run it
locally, the environment variables, and that the kit's SDKs are npm dependencies that
Dependabot updates.

## 9. Next: the service's own feature

Offer to replace the example feature (`item`) with the first real one, keeping the two checks
on every write: the plan (`require` → 402) and the permission (`requirePermission` → 403).
