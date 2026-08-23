# What open source actually earns — a census of two ecosystems

*Measured 2026-08-23. All numbers below come from public APIs; endpoints and dates are given
so the measurement can be repeated.*

I wanted to know whether building small open source tools can pay for itself. Not "can it in
principle" — what the distribution actually looks like. So I counted, in two ecosystems that
have public data: Home Assistant custom integrations, and projects hosted by Open Source
Collective.

Short version: **the median is zero, and it stays zero for a long time.**

---

## Part 1 — Home Assistant custom integrations (n = 4,219)

Home Assistant publishes anonymous usage analytics. Sources, all fetched 2026-08-23:

- `analytics.home-assistant.io/data.json` — 669,593 active installations reporting
- `analytics.home-assistant.io/custom_integrations.json` — install counts for 4,219 custom
  integration domains
- `data-v2.hacs.xyz/integration/data.json` — the 3,159 integrations listed in HACS's default
  store
- GitHub REST/GraphQL for funding files, READMEs and sponsor counts

**Caveat that matters for every absolute number here:** analytics is opt-in and covers 56.7 %
of active installations. Install counts are therefore lower bounds. Ratios and medians are
unaffected as long as opt-in isn't correlated with which integrations people use — which I
cannot verify.

### How many installs a custom integration gets

| Threshold | Integrations | Share |
|---|---:|---:|
| ≥ 10 installs | 2,544 | 60.3 % |
| ≥ 100 | 1,125 | 26.7 % |
| ≥ 1,000 | 259 | 6.1 % |
| ≥ 10,000 | 27 | **0.6 %** |

Median across all 4,219: **20 installs.** Mean: 452. The gap between the two is the whole story.

### What being listed in HACS is worth

| | n | median | p25 | p75 | p90 |
|---|---:|---:|---:|---:|---:|
| Listed in `hacs/default` | 2,048 | **62** | 15 | 277 | 1,148 |
| Not listed | 2,171 | **6** | 2 | 31 | 146 |

A factor of ten — but this is confounded. Projects that get listed are more mature to begin
with, so factor 10 is an **upper bound** on the causal effect of listing, not an estimate of it.

The number I did not expect: the median *listed* integration has **62 installations**.

### Does anyone charge for one?

I took the 100 most-installed custom integrations (2,762 to 43,851 installs) and checked every
one for funding files, README donation links, and paywalls.

| | of 100 |
|---|---:|
| `.github/FUNDING.yml` with at least one real entry | 38 |
| README links a donation channel | 51 |
| at least one of the two | 64 |
| **charges money for the integration itself** | **0** |

A regex flagged three candidates as having a paywall. All three were false positives, and I
only found that out by reading them. One of them says verbatim *"the integration is free and
does not require any paid subscriptions"* — my pattern had matched the words and inverted the
meaning. The other two referred to subscriptions to *third-party services*, not to the
integration. If you run a similar scan, read your hits.

### What the donations amount to

Public sponsor counts for the 37 maintainers behind the top 40 integrations:
**median 0.** 22 of the 37 have no GitHub Sponsors profile at all — including the maintainers
of integrations with 43,851, 32,281 and 29,306 installations.

The two highest counts (81 and 72) both have explanations that don't generalise: one maintainer
sells a separate commercial product alongside the free integration, the other is a Home
Assistant core developer sponsored for his work on the project as a whole.

Only *public* sponsors are visible, and Ko-fi or Buy-Me-a-Coffee amounts are not visible at all.
These are lower bounds. What they establish is an order of magnitude, not an income.

---

## Part 2 — Open Collective, full census of one fiscal host (n = 2,647)

Open Source Collective is a 501(c)(6) nonprofit that acts as fiscal host for open source
projects, so a project can receive money without being a legal entity itself. Every collective's
finances are public.

Queried via `api.opencollective.com/graphql/v2` on 2026-08-23: all 2,647 collectives hosted by
`opensource`, with `totalAmountReceived(periodInMonths: 12, net: false)`. This is a full census
of that host, not a sample. All amounts in USD, gross, before fees.

| | n | median | p25 | p75 | p90 | p99 | max |
|---|---:|---:|---:|---:|---:|---:|---:|
| All collectives | 2,647 | **$10** | $0 | $555 | $4,615 | $100,499 | $1,225,838 |
| Created < 24 months ago | 459 | **$0** | $0 | $145 | $1,884 | — | $214,680 |

- **47.5 %** of all collectives received **nothing** in twelve months. Among those younger than
  two years: **56.6 %**.
- 13.7 % of the young ones cleared $1,000/year. 0.4 % cleared $100,000.
- The top 10 collectives account for **39.4 %** of the $12.5M that moved through the host in
  twelve months. The top 1 % (27 collectives) account for **56.6 %**.

Most of the largest recipients are infrastructure other developers build on — a storage
protocol, a language foundation, a documentation project, a mapping library, a web framework, a
compiler. Not all: the top ten also contains a note-taking application and a museum-collections
consortium, so "infrastructure" is the tendency, not the rule. **Nine of the top ten are more
than four years old** (4.4 to 9.4 years); the tenth is 1.2 years old.

**Selection bias, stated plainly:** collectives on Open Source Collective applied and were
accepted. This is a population of projects that *tried* to raise money. For an arbitrary new
project the distribution is worse than this, not better. These numbers are an upper bound.

---

## Measurement notes, including the two ways I got it wrong first

Both of these produced confident, plausible, wrong numbers before I caught them.

1. **An HTTP error body read as data.** `gh api` writes 404 response bodies to *stdout*, not
   stderr. My first funding-file scan checked only stdout and reported that 100 of 100 repos
   had a `FUNDING.yml` — because every repo received `{"message":"Not Found"}` and that is a
   non-empty string. A 100 % hit rate is a warning sign, not a result. I broke the check
   deliberately against a repository I knew had no funding file; it still reported "present",
   which is how I found it. The corrected run returned 40, of which 38 had real entries — the
   other two were template comments (`patreon: # Replace with…`) counted as live entries.

2. **A GraphQL error returned HTTP 200.** Querying a fiscal host slug that does not exist
   returns status 200 with `data: null` and the error in the body. A script checking only the
   status code would have silently reported "0 collectives". During the real run this
   protection fired twice — an HTTP 429 whose body used the key `error` rather than `errors`,
   and an HTTP 503 that returned an HTML error page after 1,000 records. Either would have
   truncated the census silently, and because results were sorted newest-first, what survived
   would have been almost entirely the young collectives — the ones that are mostly zeros.

Both checks were validated by breaking them on purpose first. A check that measures nothing
also reports green.

Two collectives were verified against independently summed transaction data: one matched to the
cent across 430 transactions, the other to within 0.35 %. To confirm the 12-month window really
filters rather than defaulting to zero, I checked a collective showing $0 for the period that
has $190 lifetime — it does.

---

## What I take from this

Donations are not a business model for a new open source project. That is now measured twice, in
two ecosystems with no overlap: an application ecosystem where **none** of the hundred most-used
add-ons charges anything and the median maintainer has zero public sponsors, and a funding
platform where the median project created in the last two years received **zero dollars in a
year**.

What does earn money on that second list is mostly infrastructure other projects depend on, and
in nine of ten cases it has been maintained for more than four years. I did not measure *who*
pays — that would need a separate look at the contributor side, and I have not done it. What the
distribution does say is that "useful small tool" and "project that receives money" are not the
same population, and that age is the most visible thing separating them.

None of this says open source is not worth doing. It says that if the plan involves being paid,
the payment has to come from somewhere other than the goodwill of the people who install it.

