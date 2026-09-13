# One incident, and it landed three days before real money could reach it

**May 2026.** SparkDate's first commit landed 2026-05-02; this is the month the
product was still being built, not yet run. Part of a five-month retroactive
sweep of git history, `reports/`, and memory files, written 2026-09-07 to give
every month since launch its own field-notes edition — continuing the numbering
from `reports/NOTHING_THREW_2026-09-04.md` (incidents 01–18) and
`reports/FIELD_NOTES_2026-09.md` (19–22). One May incident left an anchor
still findable four months later; it is numbered 23 because it was catalogued
last, not because it happened last — see "What I did not verify" below.

**The one-line finding:** a pricing refactor left the payment path silently out
of sync with the admin form for 44 hours, in a shape that would have charged a
real customer $2.50 for a ticket — and it was fixed the day before there was a
live card on file to charge.

| | |
|---|---:|
| Incidents | 1 |
| Caught by anything automated | 0 |
| Days between the bug shipping and Stripe going live | 3 |

---

## EVIDENCE — how it was caught

One incident, one detection mode: `HUMAN`. Nobody outside the project asked a
question and nothing threw an error — it was found in ordinary pre-launch
testing of the new single-price event flow, the same way you'd expect a bug in
an unreleased feature to be found. There is no automated, gate-level, or
operator-reported catch to report this month; the sample is simply too small
to say anything about detection mix beyond "a human looked before it mattered."

## MECHANISM — an incomplete two-sided contract

### 23 — A pricing refactor that touched half of a two-sided contract

A commit removing gendered ticket pricing (`38785355`, 2026-05-30,
`Co-Authored-By: Claude Opus 4.8`) updated the admin event-creation form and
the on-page price display to a single spots/price model. It did **not** touch
the payment path: `api/purchase-ticket.js` still read
`spotsWomen`/`spotsMen` and `priceWomen`/`priceMen` — fields that are
`undefined` on any event created through the new admin.

The effect on a new-model event: the capacity check read `undefined → 0` and
rejected **every** purchase attempt as `Event full` (409). Had a purchase
somehow passed the capacity check, price would have resolved to `$0`, so it
would have charged only the `$2.50` service fee — a real ticket for a dollar
fifty short of nothing.

Fixed the next day, 2026-06-01 (`69cdb99d`), which added
`lib/seat-model.js` as a single source of truth for both the admin and payment
sides, plus 9 new unit tests.

**Cost:** none realised. Stripe's live publishable key was not switched in
until 2026-06-02 (`e7a2007e`) — three days after the mismatch was introduced
and a full day after it was fixed. No real customer, and no live payment
method, could have reached this code path during the window it existed. The
near-miss is real regardless: the shape of the bug (silently charge $2.50
instead of full price) is exactly the kind of thing that is expensive to
notice once real money is involved, and this was fixed the week before real
money was involved.

**Anchor:** commits `38785355` (introduces the mismatch) and `69cdb99d`
(fixes it); Stripe live-key switch at `e7a2007e`.

## DECISION — nothing further was needed

The fix shipped with its own regression coverage
(`lib/seat-model.js` plus `tests/seat-model.test.js`) the day it was found, a
day before it could have mattered. No follow-up action, standing rule, or
countermeasure was opened against this incident — the ordinary pre-launch
testing that caught it is not a process that needs strengthening on the
strength of one data point.

## What I did not verify

- **Whether May had other incidents that simply left no anchor.** May has 343
  commits and one catalogued incident. That is not a claim that May had one
  bug — it is a claim that one May incident left an anchor (a named commit
  fixing a named prior commit, with an explicit before/after) still findable
  four months later. Anything not written down plainly enough to survive that
  gap is invisible to this method, by construction, and selection bias here is
  sharper than in any later month.
- **Whether the admin/payment mismatch affected anything else in the same
  window.** The fix commit describes exactly the capacity and price fields it
  corrected; I did not audit the rest of the May 30 – June 1 diff for related
  gaps beyond what that commit's own message documents.
- **Numbering convention.** Incident numbers are assigned in the order they
  were catalogued, not the order they happened — this is why a May incident is
  numbered 23, higher than August's incidents in the 03–18 range. Numbers are
  never reassigned once published.
- **Names, account identifiers and customer records are omitted throughout.**
