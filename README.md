# Job Outreach Agent

Instructions for Claude Code. **Not an application** — there's no code to build,
no server, no LLM API key. Claude Code reads these files and does the work with
its own tools: Gmail MCP, WebFetch, Bash (`curl`), Read, Write.

## What it does

You apply to a job. The confirmation lands in Gmail. You open Claude Code in this
folder and say **"run outreach"**. It then, per application:

1. Reads the confirmation → company, role, posting URL
2. Fetches and reads the actual job posting
3. Finds the right person — cheapest source first, spending a paid credit only
   as a last resort
4. Writes a short, specific email grounded only in what it actually read
5. **Creates a Gmail draft** — never sends
6. Later, notices which drafts you sent and permanently retires those addresses

You review in Gmail, on your phone or desktop, and press send.

## Setup

**1. API keys** — three free signups, no card:

| Provider | Free/month | Sign up |
|---|---|---|
| Reoon | 600 verifications | reoon.com/email-verifier |
| Hunter | 25 search + 50 verify | hunter.io |
| Apollo | 100 credits | apollo.io |

Put them in `.env`:

```bash
REOON_API_KEY=
HUNTER_API_KEY=
APOLLO_API_KEY=
```

**2. Gmail label** — create a filter that labels application confirmations
`job-applications`. Or label a few by hand to start.

**3. Fill in `sender_profile.md`** — who you are, what you've built. The drafts
are only as good as what's in here.

**4. Optional: `products_used.md`** — products you genuinely use, with dated
notes. This unlocks the stronger email variant ("I've been using X and hit
these things") for those companies. Without entries, that variant is disabled —
by design.

## Running it

Open Claude Code in this folder:

```
run outreach
```

That's it. `CLAUDE.md` loads automatically and points at `RUNBOOK.md`.

To automate: `/loop 6h run outreach`, or the `/schedule` skill for a cron-style
cloud run. Note that interactively-authenticated MCP servers (including Gmail)
may not be available in unattended runs — test before relying on it.

## The files

| File | What it is |
|---|---|
| `CLAUDE.md` | Entry point, auto-loaded. The rules. |
| `RUNBOOK.md` | The step-by-step procedure |
| `docs/PROVIDERS.md` | Contact cascade + curl commands + free-tier research |
| `docs/EMAIL-TEMPLATE.md` | Exact email shape, grounding rules |
| `docs/SAFETY.md` | The rules that don't bend |
| `sender_profile.md` | **You fill this in** |
| `products_used.md` | **You fill this in** (optional but valuable) |
| `state/contacted.md` | Everyone already emailed — the dedupe list |
| `state/processed.md` | Applications already handled |
| `state/quota.md` | Provider credits spent |

## Two hard rules

1. **It never sends.** It creates drafts. You press send.
2. **It never claims anything it didn't read.** Company facts come only from the
   posting it fetched; product feedback only from `products_used.md`.

Full list in [`docs/SAFETY.md`](docs/SAFETY.md).

## Cost

$0/month. Three free tiers, ~125 contact lookups available per month, which
comfortably covers 100+ applications because most get resolved by free steps
(reading the posting, guessing a pattern and verifying it).
