# costtoserve

> Activity-based cost-to-serve that reports which customers change profitability sign depending on the allocation driver.

`FLAGSHIP` · **Business Analyst** · Advanced · ~4 weeks · Logistics - regional parcel carrier

**Primary language:** SQL
**Tags:** `activity-based-costing`, `dbt`, `postgis`, `sql`, `sensitivity-analysis`, `profitability`

---

> **Implementation note.** The catalogue specifies **SQL** for this
> project and that remains the target. This repository ships a runnable
> **Python** reference implementation of the core differentiator so the
> behaviour is executable and tested today; port it to SQL as step one
> of your own build.

## The problem

A carrier prices by weight and zone and believes every customer above a revenue threshold is profitable. In reality a customer with many small rural deliveries, frequent redelivery attempts and a high claims rate can be substantially loss-making while appearing as a top-ten account on the revenue report - and the commercial team is incentivised to win more of them.

## ⭐ The differentiator

Activity-based costing with **explicit allocation-driver sensitivity analysis**. Every ABC model rests on contestable allocation choices - per-stop versus per-parcel versus per-kilometre - so this one reports how customer profitability rankings *change* under each defensible driver set, and flags customers whose profitability **sign** is driver-dependent. A generic ABC model picks one driver set, presents a single ranked list as fact, and is destroyed in the first stakeholder meeting by someone who disagrees with the allocation.

This is the sentence to lead with when someone asks you to walk through the
project. Everything else in this repo exists to make it true and to prove it.

## Data

A documented synthetic generator producing shipment-level events (pickup, sort, linehaul, last-mile, redelivery, claim) with a **known true cost structure**, plus public sources - US Census County Business Patterns and OpenStreetMap road networks - for geographic and density realism.

> No paid API key is required to run or demo this project. Where a paid
> service would add value it is wired as an optional enhancement behind an
> interface with an offline mock as the default implementation.

## Stack

- PostgreSQL + PostGIS - the SQL and the data model are the artefact
- dbt with tests
- Python for the sensitivity harness
- Metabase or Streamlit for the output
- dbt tests + pytest

## Core capabilities

- Activity model - cost pools, drivers and rates - all versioned and expressed as data, not hard-coded
- Shipment-level cost build-up with full traceability from total cost back to each pool
- Customer and lane profitability with a contribution-margin waterfall
- Driver-sensitivity harness reporting rank stability and sign-flip customers
- Rural/urban density segmentation derived from real road-network geography

## Repository layout

```
models/staging/
models/activities/
models/marts/
sensitivity/
generator/
dbt_tests/
app/
```

## Build plan

1. Generator with a known true cost structure. You cannot validate an allocation without one.
2. Activity model as data. If drivers are hard-coded in SQL, the sensitivity analysis is impossible - which is why most ABC models never do one.
3. Cost build-up, then reconciliation. Reconcile before you rank.
4. Sensitivity harness last; it is the differentiator and it depends on everything above.

## Testing strategy

Assert allocated costs **reconcile exactly to total cost with zero leakage under every driver set** - unallocated cost silently vanishing is the most common and most damaging ABC bug. Assert recovered per-unit costs match the generator's true values within tolerance.

Tests assert **correctness**, not merely that the code runs. A green suite on
this repo is a claim about behaviour under adversarial conditions; treat any
test that would pass against a deliberately broken implementation as a bug in
the test.

## Quality & safety layer

Every profitability figure carries the driver set that produced it. The tool will not emit a customer ranking without also emitting its sensitivity.

## Measurable outcome

> Eleven of the top-50 accounts by revenue are contribution-negative, and four of those flip sign depending on allocation driver - so the report names which four need a commercial decision rather than a modelling decision.

State it in these terms — business units, not technical ones — in your CV
bullet and in the first thirty seconds of describing the project.

## Interview questions this project answers

- **What allocation driver did you choose and why?**
- **How do you defend an ABC model against a stakeholder who disagrees with the allocation?**
- **What is a sign-flip customer and what do you do about one?**

## What this deliberately is *not*

- Not a general ledger. It models cost to serve and reconciles to a total you supply.
- Not a pricing tool - it is the input to one.


## Run it now

```bash
python -m unittest discover -s tests -v   # the suite
python -m src.demo                        # the 60-second artefact
```

Requires Python 3.11+. The runnable core uses **only the standard
library** (including `sqlite3`), so there is nothing to install.

## Getting started

```bash
git clone <your-fork-url> costtoserve
cd costtoserve
docker compose up -d          # Postgres + PostGIS
python -m generator --shipments 2000000
dbt build
python -m sensitivity run --drivers config/drivers.yaml
dbt test && pytest
```

Docker is supported but optional — every path above works on a plain
Windows/macOS/Linux laptop without a cloud account.

## Definition of done

- [ ] The differentiator above is implemented, and a test proves it
- [ ] The measurable outcome is produced by a command anyone can run
- [ ] `README` explains the one decision a generic version gets wrong
- [ ] CI runs the full suite on every push and is green on `main`
- [ ] A recruiter can see the headline artefact in under 60 seconds

## Licence

MIT — see [LICENSE](LICENSE).
