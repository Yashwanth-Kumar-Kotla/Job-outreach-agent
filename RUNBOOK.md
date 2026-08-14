# Runbook — what to do on each outreach run

You are the agent. There is no application to run. You do each step yourself with
your own tools: Gmail MCP, WebFetch, Bash (`curl`), Read, Write.

Work **one application at a time, start to finish**, then move to the next. Do not
batch stages across applications — a half-finished batch is hard to resume.

---

## Step 1 — Find new applications

Search Gmail for application confirmations the user hasn't processed yet.

```
mcp__claude_ai_Gmail__search_threads  →  label:job-applications
```

Read `state/processed.md`. **Skip any message ID already listed there.** If nothing
is new, say so and stop — don't invent work.

If the label doesn't exist yet, ask the user to create a Gmail filter for it, or
fall back to searching for confirmation-shaped subjects (`"application received"`,
`"thanks for applying"`, `"we received your application"`).

## Step 2 — Read the application

Open the message. Extract:

- **Company name**
- **Role title**
- **Job posting URL** (usually a Greenhouse / Lever / Ashby / Workday link)
- **Company domain** — from the posting URL or the sender's address

If you can't determine company and role, log the message ID in
`state/processed.md` with `SKIPPED — unparseable` and move on.

## Step 3 — Read the job posting

`WebFetch` the posting URL. Some ATS pages block bots; if the fetch fails, continue
with just company + role and note it.

Pull out:

- What the role actually involves — the real responsibilities, not the boilerplate
- The stack / domain
- Seniority and team size signals
- **Any named human** — recruiter, hiring manager, team lead. This is free contact
  data and it's the best kind. Look at the footer, the "questions?" line, and the
  posting author.

Estimate **company size** (startup <200 / mid 200–2000 / large 2000+) from the
posting, the careers page, or the domain. This drives who you contact and which
email variant you use.

## Step 4 — Find the contact

Follow the cascade in `docs/PROVIDERS.md` **in order**. Stop at the first result
with confidence ≥ 80.

Before spending any credit:

1. Did the posting name someone? → go straight to pattern-guess + verify.
2. Is there a team/about page? → `WebFetch` it and look for the right person.

Only then spend a Hunter search or an Apollo reveal.

**Log every credit you spend** in `state/quota.md`, immediately, before moving on.
If a provider returns 402/429, mark it exhausted there with today's date and fall
through to the next one.

If nothing clears the confidence bar, record the application in
`state/processed.md` as `NO CONTACT` and move to the next one. Do not lower the
bar to produce a result.

## Step 5 — Pick the right person

One person. Never two.

| Company size | Contact |
|---|---|
| Startup (<200) | Founder, eng lead, or hiring manager |
| Mid (200–2000) | Recruiter or hiring manager |
| Large (2000+) | Recruiter / talent partner |

Reject role aliases (`careers@`, `jobs@`, `hr@`, `info@`, `recruiting@`) — those
are shared inboxes, not people.

## Step 6 — Check you haven't contacted them

**Grep `state/contacted.md` for the email address.** If it's there, stop — that
person has already been emailed. Record the application as
`SKIPPED — already contacted <email>` and move on.

This check is not optional and it is not a judgment call.

## Step 7 — Write the email

Read `docs/EMAIL-TEMPLATE.md` and follow it exactly. Read `sender_profile.md` for
who the user is, and `products_used.md` for the Variant A gate.

Under 130 words. Under 15 lines. Every fact traceable to one of:

- the user → `sender_profile.md`
- the product → `products_used.md`
- the company/role → the posting you actually fetched
- the recipient → what the provider returned

**Nothing from your own knowledge about the company.** No funding rounds, no
launches, no headcount, no "I saw you recently…". If it isn't in the posting text
you fetched, it doesn't go in the email.

## Step 8 — Create the Gmail draft

```
mcp__claude_ai_Gmail__create_draft  →  to, subject, body
```

Append the provenance footer below the signature:

```
---
[agent] Hunter · confidence 82 · verified · variant B
        Senior Backend Engineer @ Acme · applied 2026-08-11
        DELETE THESE LINES BEFORE SENDING
```

**Never send.** You create drafts. The user reads, edits, and presses send.

## Step 9 — Record what you did

Append to `state/processed.md`:

```
2026-08-13 | <gmail-msg-id> | Acme | Senior Backend Engineer | DRAFTED priya@acme.com (hunter/82)
```

Do **not** write to `state/contacted.md` yet — a draft isn't a send.

## Step 10 — Reconcile earlier drafts

At the start or end of a run, check what happened to drafts from previous runs:

- Search Gmail `in:sent` for the recipients listed as `DRAFTED` in
  `state/processed.md`.
- **Found in sent** → append the address to `state/contacted.md`. That person is
  now permanently off-limits.
- **Draft gone, nothing in sent** → the user deleted it. Mark it `DISCARDED` in
  `state/processed.md` and **do not** add to `contacted.md` — they stay
  contactable.
- **Draft still sitting there** → leave it alone.

## Stop condition

If **15 or more drafts** are sitting unreviewed in Gmail, stop processing new
applications and tell the user. No point burning finder credits on drafts nobody
is reading.

---

## Reporting back

At the end of a run, give the user a short summary:

```
3 new applications.
  Acme (Senior Backend)  → drafted to priya@acme.com   [posting/95, verified]
  Beta Inc (ML Eng)      → no contact found
  Gamma (Platform)       → skipped, already contacted raj@gamma.io

Reconciled: 2 sent, 1 discarded.
Quota: Reoon 41/600 · Hunter 3/25 · Apollo 7/100
```

Not a wall of narration. The user wants to know what's in their drafts folder.

---

## When something's unclear

Ask. Do not guess at a company, invent a contact, or write an email you can't
source. A run that produces one good draft and two "no contact found" is a
successful run.
