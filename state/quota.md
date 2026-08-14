# Provider quota ledger

Append a line **as you spend a credit**, not at the end of a run. If the run dies
halfway, the log should still be accurate.

```
<YYYY-MM-DD> | <provider> | <credits> | <what for>
```

On a quota error from a provider:

```
<YYYY-MM-DD> | <provider> | EXHAUSTED until <YYYY-MM-DD>
```

Skip that provider until the date passes. Still exhausted? Back off further:
3 → 7 → 14 → 31 days.

**Your count is a hint. The provider's error response is the truth.** If Apollo
402s while this file says 40 credits remain, believe Apollo — the drift is
normal (a failed request can still consume a credit, and web-UI use isn't logged
here).

## Monthly allowances

| Provider | Free/month | Resets |
|---|---|---|
| Reoon | 600 verifications | monthly |
| Apollo | 100 credits | monthly |
| Hunter | 25 search + 50 verify | monthly |

---
