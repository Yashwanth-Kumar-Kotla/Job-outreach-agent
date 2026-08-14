# Email Template Specification

The drafter's contract. Read with `SAFETY.md` §3.

## 1. Target shape

Under **130 words**, under **15 lines**, plain text. Every line earns its place
or gets cut.

```
Hey {first_name},

I'm {name}, {age} {year} {major} @ {school}. I built {project} — {metric}.
Previously {prior_work} @ {prior_company}.

{EVIDENCE BLOCK — variant A or B, see §3}

{Why this company, tied to something real — 1-2 lines, §4}

{Ask — one line, §5}

Best,
{name}
{site} · {linkedin}

{opt-out line}
```

## 2. Hard rules

1. **No self-adjectives.** Not "passionate," "driven," "hard-working,"
   "exceptional." State what you built and what it did. The reader draws their
   own conclusion.
2. **No flattery.** Not "amazing product," "huge fan," "love what you're
   building." It reads as filler and costs credibility.
3. **Numbers over claims.** "3k users in four months" not "significant
   traction." If there's no number, describe the thing concretely instead.
4. **One ask.** Never two.
5. **Nothing the posting or the profile doesn't support.** No invented facts
   about the company, ever. See §6.
6. **No em-dashes in the body.** Reads as machine-written.
7. **Contractions on.** "I've," "I'd," "you're." Absence of contractions is the
   clearest tell of a generated email.

## 3. The evidence block — the part that does the work

This is the section that makes the email land. It has two variants and **the
agent does not get to choose freely** — the choice is forced by what evidence
exists.

### Variant A — real product feedback

**Gate: only if the company appears in `products_used.md` with dated notes from
the user.** No exceptions. If the company is not in that file, Variant A is
unavailable.

```
I've been using {product} for {duration}. A few things I hit:
1. {observation}
2. {observation}
3. {observation}
```

Rules:
- Every item comes **verbatim in substance** from `products_used.md`. The agent
  rephrases for brevity, never invents, never extrapolates, never adds a fourth.
- Two items is fine. One good one beats three padded ones.
- Frame as observations, not criticism. "Search doesn't handle typos" not
  "your search is broken."
- Never speculate about their codebase or why something is the way it is.

**If `products_used.md` has fewer than two entries for this company, fall
through to Variant B.** Do not top up with guesses.

### Variant B — the problem they're hiring to solve

The default, and it is not a weaker email. It's grounded in the posting, which
is real evidence.

```
Looks like you're building out {thing the JD implies}. I ran into
{the same problem} building {project} — {what you did about it, one line}.
```

Rules:
- The problem statement comes from the **fetched posting text only** — the
  responsibilities, the stack, the team description.
- The connection must be to something in `sender_profile.md` that actually
  exists.
- If the posting is too thin to infer a real problem, say something honest and
  short instead of manufacturing insight. A plain email beats a wrong one.

### Choosing

```
if company in products_used AND entries >= 2:  → Variant A
else:                                          → Variant B
```

Log which variant was used, and store it on the draft. It's the main thing to
measure later.

## 4. Why this company

One or two lines, tied to something in `sender_profile.md` — a project, a
background detail, a problem the user has actually hit. Not the company's
marketing copy.

Bad: *"I really believe in your mission to democratize finance."*
Good: *"I built a UPI reconciliation tool for my dad's shop, so the
small-merchant problem you're working on isn't abstract to me."*

If there is nothing genuine to say, **cut this section entirely.** A 10-line
email with nothing false is stronger than a 14-line one with a manufactured
connection.

## 5. The ask

One line, small, easy to say yes to.

- *"Would love to be considered for the {role} — happy to send more if useful."*
- *"If there's an intern-shaped gap on the team, I'd love to talk."*

Not: a call request plus a portfolio review plus a referral ask.

## 6. Grounding — non-negotiable

Every factual claim traces to exactly one of:

| Claim about | Source |
|---|---|
| The user | `sender_profile.md` |
| The product | `products_used.md` |
| The company / role | the posting text you actually fetched this run |
| The recipient | `contacts` row |

**Nothing comes from model memory about the company.** No funding rounds, no
product launches, no headcount, no news. If it isn't in the posting, it doesn't
go in the email. This is the failure mode that most damages the user, because
hallucinated company facts are fluent and confident.

The drafter prompt must state that the posting text is the only permitted
source of company facts, and a post-generation check should flag drafts
containing named entities absent from all four sources.

## 7. Company-size awareness

The template above is tuned for startups. It does not transfer.

| Company size | Contact | Evidence block |
|---|---|---|
| **Startup (<200)** | Founder, eng lead, hiring manager | Variant A works well; product feedback is welcome |
| **Mid (200–2000)** | Recruiter or hiring manager | Variant B. Product feedback usually misplaced |
| **Large (2000+)** | Recruiter | Variant B, shortest form. Skip §4. Recruiters skim |

Unsolicited product critique to a recruiter at a 5,000-person company reads as
odd. The drafter must branch on size, inferred from the posting and company
domain.

## 8. The opt-out line

Required by `SAFETY.md` §3.3. Keep it to one short line after the signature:

> *If I've got the wrong person, no worries — I won't follow up.*

It costs one line, it's honest, and the low-pressure framing helps rather than
hurts.

## 9. Worked example — Variant B

```
Hey Priya,

I'm Yash, a CS undergrad. I built Mana Raithu, a Flutter app that does
crop-input cost math for farmers in Telugu and English.

Looks like the mobile team is taking on offline-first sync. I hit that
building Mana Raithu — rural users lose signal constantly, so I moved to
a local-first store with reconciliation on reconnect.

I'd love to be considered for the mobile intern role.

Best,
Yash
yash.dev · linkedin.com/in/yash

If I've got the wrong person, no worries — I won't follow up.
```

68 words. No flattery, no adjectives about himself, one ask, every fact
traceable. This is the bar.

## 10. Self-check before writing the draft

The drafter validates its own output and regenerates once on failure:

- [ ] Under 130 words and under 15 lines
- [ ] Zero self-describing adjectives
- [ ] Zero compliments about the company
- [ ] Exactly one ask
- [ ] Every company fact appears in the posting you fetched
- [ ] Variant A used only if gated by `products_used.md`
- [ ] Recipient's first name correct, and it's a person not an alias
- [ ] Opt-out line present
- [ ] No em-dashes; contractions used
