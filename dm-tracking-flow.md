# Post-Send Tracking Flow — AI-Work Receipts DM Run

Purpose: every reply gets logged in `dm-targets.csv` within minutes, so the Oct 2 kill-rule is a data read, not a memory.

---

## Pipeline stages (use exactly)

The cycle per Target in `dm-targets.csv`:

```
pending → open (sent) → replied → interested → won / no
```

Fill the `stage` and `note` columns. Keep the CSV as the single source of truth; write values by hand after each event.

---

## Reply mapping — what to record

| What happened | Stage → | Note to log |
|---|---|---|
| DM sent | `open` | date sent |
| No reply after 48h | stays `open` | apply one follow-up (2-touch rule), log "followup1 <date>" |
| Reply: "interested but busy" | `replied`→`interested` | key phrase, they asked for sample/template? |
| Reply: "how much" | `interested` | swap to price variant, log price question |
| Reply: "why show AI" | `interested`? | they're resistant — use invoice-protection follow-up |
| Clicked/praised sample, no buy | `interested` | strong-want signal counts for kill-rule even if unpaid |
| Paid $29 | `won` | log: payment link used, date |
| Explicit no / ghost after 2 touches | `no` | log reason, do NOT re-contact |

---

## Kill-rule math (Oct 2)

| Metric | Read |
|---|---|
| DMs sent (cumulative) | count stages ≥ `open` |
| Strong signal | stages `interested` + `won` |
| Kill-rule fail | strong-signal count = 0 by Oct 2 |
| Kill-rule pass | strong-signal count ≥ 1 |

Compute on demand: `(Import-Csv dm-targets.csv | Where stage -in 'interested','won').Count`

If 0 by Oct 2: re-target (new 10) or kill. Do NOT build first.

---

## Operating cadence

- Log within 5 min of each event (stage + note).
- One follow-up per target per 48h max; two touches total then stop.
- Nightly read: `Get-Content dm-targets.csv` → update `openers` variant if reply-rate <15% after 5 sent.

## Failure-mode triggers

1. Reply rate <15% after 5 sent → rewrite the opening around invoice-protection (C2PA), not time-saving. Variant B in outreach skill.
2. All "busy" replies → lead with sample first, then price.
3. Priyanka (pain-signal) converts but Dominica doesn't → pain-signal wins; bias all openers to dispute/invoice angles.