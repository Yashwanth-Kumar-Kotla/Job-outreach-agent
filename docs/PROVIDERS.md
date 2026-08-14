# Contact Cascade — how to find the person

Step 4 of `RUNBOOK.md`. This is the only step that spends a limited resource, so
work down the list in order and stop as soon as you have a contact at
confidence ≥ 80.

API keys live in `.env` at the project root. Read it with the Read tool and use
the values in your `curl` calls. **Never echo a key into your output.**

---

## The cascade

| # | Source | Cost | Limit/month | Get from it |
|---|---|---|---|---|
| 1 | The job posting | free | ∞ | A name, sometimes an email |
| 2 | Company team/about page | free | ∞ | A name |
| 3 | Pattern-guess + Reoon verify | 1 verify | 600 | Email, from a name |
| 4 | Hunter domain search | 1 search | 25 | Name + email |
| 5 | Apollo reveal | 1 credit | 100 | Email, last resort |

**The insight:** verification is nearly free (600/month), so once you have a
**name** you can get the email for almost nothing. Steps 1–3 handle most cases.
Steps 4–5 exist for companies where no name is obtainable.

---

## Step 1 — The posting

You already fetched it in Step 3 of the runbook. Re-read it for a human: a
recruiter named in the footer, a hiring manager quoted, a "reach out to ___"
line, an author byline. Free, and the highest-confidence source there is.

Found a name **and** an email → confidence **95**. Done.
Found just a name → go to step 3.

## Step 2 — Team / about page

`WebFetch` `https://<domain>/team`, `/about`, `/people`, or `/company`. Look for
someone whose title matches who you should be contacting (see the company-size
table in the runbook).

Found a name → step 3. Nothing → step 4.

## Step 3 — Pattern-guess + Reoon verify

Given a name and a domain, generate candidates in likelihood order:

```
first.last@   flast@   first@   firstl@   first_last@   lastf@
```

Verify each with Reoon, stop at the first deliverable one:

```bash
curl -s "https://emailverifier.reoon.com/api/v1/verify?email=priya.sharma@acme.com&key=$REOON_API_KEY&mode=power"
```

Map the returned `status`:

| Reoon status | Action |
|---|---|
| `safe` | confidence **80**, verified. Use it. |
| `invalid` | discard, try the next pattern |
| `disposable` / `spamtrap` | discard entirely, don't try more patterns |
| `catch_all` / `unknown` / `risky` | confidence 30 — **below the bar**. See below. |

**Budget 4 verifications maximum per person.** Past that the domain doesn't use a
standard pattern and you're burning credits on noise.

**Catch-all domains accept everything**, so a `catch_all` result tells you
nothing. Don't treat it as a hit. If the domain is catch-all, skip to step 4 or 5
where you get a provider-attested address instead of a guess.

## Step 4 — Hunter domain search

```bash
curl -s "https://api.hunter.io/v2/domain-search?domain=acme.com&api_key=$HUNTER_API_KEY&limit=10"
```

Returns people with emails, titles, and Hunter's own confidence score. Filter
titles for recruiting signals (`recruiter`, `talent`, `people`, `hiring`, `HR`)
or for the hiring manager's likely function.

Normalize Hunter's score to ours:

- Hunter confidence ≥ 90 → our confidence **80**
- Hunter confidence 50–89 → our **60** (below bar — verify with Reoon to lift it
  to 80, that's cheap)
- Below 50 → discard

## Step 5 — Apollo (last resort)

**People Search is free — only revealing an email costs a credit.** So search
broadly, pick exactly one person, then spend one credit.

Search:

```bash
curl -s -X POST "https://api.apollo.io/api/v1/mixed_people/search" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $APOLLO_API_KEY" \
  -d '{"q_organization_domains_list":["acme.com"],"person_titles":["recruiter","talent acquisition"],"page":1,"per_page":10}'
```

Reveal the single best match:

```bash
curl -s -X POST "https://api.apollo.io/api/v1/people/match" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $APOLLO_API_KEY" \
  -d '{"first_name":"Priya","last_name":"Sharma","domain":"acme.com"}'
```

**Never reveal more than one person per application.** That's the whole budget
discipline.

Free-tier caveat: Apollo **masks last names** in search results. You can still
reveal the email, but you can't pattern-guess from an Apollo search hit.

> These endpoint shapes were correct as of August 2026 but provider APIs change.
> If a call returns an unexpected shape, `WebFetch` the provider's API docs and
> adapt rather than retrying the same broken call.

---

## Confidence scale

One 0–100 scale across all sources. The bar is **80**.

| Situation | Score |
|---|---|
| Name from the posting + email verified deliverable | 95 |
| Provider-returned email, verified deliverable | 90 |
| Provider email, their confidence ≥ 90, unverified | 80 |
| Pattern-guessed, verified deliverable | 80 |
| Provider email, their confidence 50–89 | 60 |
| Pattern-guessed, catch-all or unknown | 30 |
| Anything else | 0 — discard |

Below 80 → the application gets `NO CONTACT`. **Do not lower the bar to produce a
result.** A wrong email under the user's name is worse than no email.

---

## Quota tracking

Append to `state/quota.md` **as you spend**, not at the end of the run:

```
2026-08-13 | reoon  | 3 | verify priya.sharma@acme.com (+2 patterns)
2026-08-13 | hunter | 1 | domain-search beta.io
2026-08-13 | apollo | 1 | reveal raj@gamma.io
```

On a 402 / 429 / quota error, add:

```
2026-08-13 | apollo | EXHAUSTED until 2026-08-16
```

and skip that provider until the date passes. If it's still exhausted then, back
off further: 3 → 7 → 14 → 31 days. Don't try to model each provider's billing
cycle — you can't know it, and backing off converges on the real one anyway.

**Your count is a hint; the provider's error response is the truth.** If a
provider 402s while your log says you have credits left, believe the provider.

---

## Free tiers (verified August 2026)

**Use these three.** All recurring monthly, all with API access on free:

| Provider | Free/month | Notes |
|---|---|---|
| **Reoon** | 600 verifications | Best in class. No card. Plus 100 non-expiring credits |
| **Apollo** | 100 credits | Search free, reveal costs. Last names masked |
| **Hunter** | 25 search + 50 verify | Permanent, no card |

> **Apollo signup warning:** the free allowance depends on your signup domain. A
> personal Gmail gets **100 credits/month**; a verified corporate domain gets
> 10,000. Assume 100.

**Don't bother with these:**

- **Snov.io** — 50 credits/mo, but API access is explicitly excluded from the
  free tier. Web UI only. Useless here.
- **Lusha, ContactOut, RocketReach** — extension-driven, negligible volume.
- **Emailable, MillionVerifier, Kickbox, NeverBounce, Bouncer** — one-time signup
  credits that never refresh. Given Reoon's 600/month, they'd be adapters for
  capacity you'll never need.

**Untested, might be worth 15 minutes each:** Skrapp (~100/mo), Prospeo (75/mo),
Tomba (25/mo). All three claim monthly refills but list REST API under paid
plans. Sign up and curl before relying on any of them.

**Free tiers get cut.** Apollo went 10,000 → 100 credits/month in under a year.
If a provider's behavior stops matching this document, trust the API and tell the
user.

## Expect

- A usable contact for roughly **40–60%** of applications.
- Coverage is good at mid/large companies and poor at small startups — which is
  the annoying inversion, because outreach works best at startups.
- ~125 finder lookups/month is plenty for 100+ applications, because steps 1–3
  absorb most of the volume.
