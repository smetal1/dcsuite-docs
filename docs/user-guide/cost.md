# Cost & Billing

The **Cost** page turns metered usage into money, rolled up per organization,
so you can see what your GPU consumption costs.

## What's metered

DC Suite meters the two things that drive GPU cloud cost:

- **Node-hours** — how long nodes were held by clusters.
- **GPU-hours** — how long GPUs were held.

Metering starts when a cluster's nodes are allocated and stops when they're
released (on delete). Deleted clusters still appear in cost history for the
windows they ran.

## Reading the report

Choose a **month** (or a custom `from`/`to` range). The report shows:

- **Grand total** for the window.
- **Node-hours**, **GPU-hours**, and **unrated hours** as summary tiles.
- A per-organization table you can expand to per-cluster line items.

"**Unrated hours**" are usage for which no rate card exists yet — real
consumption that isn't priced. If you see a non-zero unrated figure, ask an
administrator to add a rate card for that profile.

## Currency

Costs are shown in each rate card's native currency by default. An
organization can set a **billing currency** so the report is converted into
one currency:

- Enter any ISO 4217 code (for example `USD`, `EUR`, `INR`).
- "**Use native**" clears the setting and reports each line item in its own
  currency.

When a conversion rate is missing for a currency pair, DC Suite shows the
native amount and flags that the conversion was skipped — it never silently
guesses a rate.

## Export

- **Download CSV** exports the current window for spreadsheets or finance
  tooling (`GET /v1/billing/costs.csv`).

## API

| Action | Endpoint |
| --- | --- |
| Cost report (JSON) | `GET /v1/billing/costs?from=…&to=…` |
| Cost report (CSV) | `GET /v1/billing/costs.csv?from=…&to=…` |
| Currency formatting table | `GET /v1/billing/currencies` |
| Set an org's billing currency | `PATCH /v1/orgs/{id}/billing-currency` |

!!! note "Who can set currency"
    Setting the currency a report is *presented* in is self-service for an org
    with the right permission. Setting the conversion *rates* themselves is an
    operator action.

---

**Next:** [API Reference](api.md).
