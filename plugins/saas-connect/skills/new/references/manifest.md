# The product manifest: choices to make with the user

`saas-kit/products/example.yaml.sample` lists every field. This page is about the decisions.

## Plans

- **A free plan** (`key: free`, no `prices`) is what every account has without a subscription.
  With one, the service is "free to start". Without one, a signed-in account without a
  subscription has nothing, and its data is deleted `retention_days` (default 90) after a paid
  plan ends (`product.expired`).
- **Paid plans** have `prices`: one per interval, `month` and/or `year`, in minor units
  (`9900` = 99.00) and a lowercase currency (`sek`, `eur`). Stripe Tax adds VAT unless
  `tax_behavior: inclusive`. A common shape: the yearly price is ten months.
- **Entitlements** are what the service checks: numbers are limits (`items: 1000`), booleans
  are features (`exports: true`). Every plan should name the same keys, so a check never
  meets a missing one.
- **`trial_days`**: the first purchase starts a trial without a card.
- **Seats** (`seats:`): a cap on members (`pricing: flat, max: 10`) or a price per member beyond
  those included (`pricing: per_seat, included: 3`, with `seat_amount` on each price). The
  kit's billing page cannot change a `per_seat` plan yet. Prefer a cap unless the user wants
  per-member pricing.
- **Usage** (`meters:` and `usage:` per plan): the service reports units. Each plan includes some
  per calendar month and may price the overage; no overage price = a hard cap.
- **`over_limit`**: after a downgrade, `block_new` (default: existing data stays, new is
  refused) or `read_only`.

## Roles and permissions

`permissions` are the service's own verbs (`items.read`, `items.write`). `roles` map a role to
them (`"*"` = all). `default_roles` gives the kit's roles (owner, admin, member) a role in the
service until someone assigns another. All three or none.

## Names

Each plan and meter has a `name`, and `labels:` names the roles, permissions and
entitlements: a map per language (`{ en: Editor, sv: Redigerare }`), `en` required, only for
what the manifest declares. The service shows them from the kit's catalogue, so give them in
every language the service is written in. Without one, the key is shown.

## The rest

- `domain` and the `client` redirect URIs: the service's public origin. Checkout and the
  billing page return only to these.
- `dimensions: { BUSINESS_AREA: <CODE> }`: HQ24 books the revenue on it. The value must exist
  in HQ24 first.
- `branding`: `color` for buttons on the kit's pages, `logo_url` optional.
- `webhooks`: `https://<domain>/api/kit/events` for the template.
- `client.login_domain`: sign-in pages on the service's own host. It needs a certificate and
  routing set up first (`saas-kit/deploy/README.md`); leave it out at the start.
