# saas-connect — a new Vanami SaaS on the kit

Takes you from a name and its plans to a running service on the Vanami kit
(`accounts.vanami.com`): sign-in, organisations and teams, plans and Stripe, the billing page,
and revenue booked in HQ24 on the service's business area.

```sh
claude plugin marketplace add vanami/claude-plugins
claude plugin install saas-connect@vanami
```

| Skill | What it does |
| --- | --- |
| `/saas-connect:new` | A new service: the repo from the kit's template, its manifest registered in the kit, Stripe products and prices, the HQ24 business area, the database, Vercel and the domain, and a check of the whole chain |
| `/saas-connect:plans` | Change a service's plans, prices, limits, roles or business area, in the kit, Stripe and HQ24 |
| `/saas-connect:check booking` | Check a service's chain: manifest, kit, Stripe, HQ24, SDK, domain, webhooks, environment |

**For Vanami's own services.** The skills need a clone of the private `hallsten/saas-kit`
next to your services, `gh`, `kubectl` with access to the kit's cluster, and the `vercel` CLI.
The skills read the kit's own docs (`template/README.md`, `deploy/README.md`) for the details,
so they follow the kit as it changes.

The assistant asks before every change outside the new repo (GitHub, the kit, Stripe, Vercel,
DNS, HQ24), never decides prices for you, and never shows a secret: the webhook secret is piped
straight from the kit into Vercel.
