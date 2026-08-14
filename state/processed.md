# Processed applications

Every application confirmation already handled, so runs don't redo work. Check
the Gmail message IDs here before processing anything in Step 1.

Format:

```
<YYYY-MM-DD> | <gmail-msg-id> | <company> | <role> | <outcome>
```

Outcomes:

| Outcome | Meaning |
|---|---|
| `DRAFTED <email> (<source>/<confidence>)` | Draft sitting in Gmail, awaiting the user |
| `SENT <email>` | Confirmed sent — also added to `contacted.md` |
| `DISCARDED <email>` | User deleted the draft — **not** added to `contacted.md` |
| `NO CONTACT` | Cascade found nobody above the confidence bar. Retryable later |
| `SKIPPED — already contacted <email>` | Hit in `contacted.md` |
| `SKIPPED — unparseable` | Couldn't extract company + role from the email |

`NO CONTACT` rows are worth retrying on a later run — quota refills monthly and
a company's data coverage can improve.

---
