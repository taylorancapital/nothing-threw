# Two catches took hours; the one that mattered took five days to notice

**June 2026.** Stripe's live key went in 2026-06-02 — this is the first month
with real money on the line. Part of a five-month retroactive sweep continuing
the numbering from `reports/NOTHING_THREW_2026-09-04.md` (01–18) and
`reports/FIELD_NOTES_2026-09.md` (19–22); see
`reports/FIELD_NOTES_2026-05.md` for why numbering doesn't run in calendar
order.

**The one-line finding:** a syntax error and a broken analytics tag both
announced themselves within hours of shipping, in the way loud bugs always do
— but a gap in the new-customer funnel sat live for five days before anyone
went looking for it, and there is no way to reconstruct after the fact how
many real guests fell into it.

| | |
|---|---:|
| Incidents | 3 |
| Caught by anything automated | 2 of 3 |
| Longest incident lived before being caught | 5 days |

---

## EVIDENCE — how they were caught

Two of June's three incidents (`THREW`, both) are genuine same-day catches —
a fatal `SyntaxError` and a broken `gtag()` initialization both fail loudly
and immediately by nature. **Do not read June's 67% automated rate as
representative of anything.** Neither bug is the kind a check has to be built
to find; both would be caught by anyone who so much as loaded the page. The
third incident (`HUMAN`) is the more typical case, and it is the one with an
unrecoverable cost.

## MECHANISM

### A. The agent was confidently wrong — 3 incidents

**24 — Two checkout gaps shipped with the original build, found five days
after go-live.** Fixed 2026-06-07 (`2b9a0f6d`, labeled P1 and P2):

- **P1 — 3-D Secure guests fell out of the funnel.** A guest paying with a
  3DS card gets `requiresAction` back from `purchase-ticket` *before* the
  inline `enrollGuestAsMember`/`recordLead` calls run. The ticket still
  confirmed correctly once the card cleared, but the guest got no welcome
  email, no 30-day Spark trial, and no nurture lead — the entire post-purchase
  funnel silently did not apply to anyone who hit a 3DS challenge.
- **P2 — a duplicate submit could hold a phantom seat.** Re-submitting a
  purchase reused the same Stripe idempotency key and got the same
  PaymentIntent back, but the seat counter had already been bumped a second
  time by the new request before the duplicate was recognised.

**Cost:** the 3DS-guest gap was live with real payments from 2026-06-02
(Stripe went live) to 2026-06-07 — five days. Any guest who hit a 3DS
challenge in that window has no record of ever receiving the trial-enrollment
funnel; there is no way to reconstruct after the fact how many that was. The
phantom-seat gap's worst case, per the fix commit, was "an over-counted seat,
never an oversell or double charge" — real but bounded.

**Anchor:** commit `2b9a0f6d` (2026-06-07, `Co-Authored-By: Claude Opus
4.8`). The gap traces to the original 3DS/trial-enrollment build around
2026-05-22–23 (`cbb8f5c6`, `a09ef885`).

**Detection:** `HUMAN`.

---

**25 — A placeholder Google Ads tag broke GA4 sitewide for about four hours.**
`a4c64bc8` (2026-06-09, 10:37 EDT) installed Facebook Pixel and Google Ads
conversion tracking together. The Google Ads tag shipped with the literal
placeholder `AW-GOOGLE_CONVERSION_ID` — not a real ID — which is not a valid
tag and broke `gtag()` initialization across all 7 public pages that carry it.
GA4 itself failed to load: `window.gtag = undefined`.

Fixed the same day, `921a39b3` (2026-06-09, 14:48 EDT) — a 4-hour-11-minute
gap. The fix removed the Google Ads tag from all 7 pages rather than
supplying a real ID, deferring Google Ads conversion tracking "per the plan
(payment UI is unstable)." Facebook Pixel, installed in the same commit, was
unaffected — it does not depend on the Google Ads tag.

**Cost:** GA4 analytics — the only telemetry the business had at the time,
eight days after go-live — was dark sitewide for roughly four hours during
active business hours. Whatever real traffic arrived in that window is
unmeasured and unrecoverable.

**Anchor:** commits `a4c64bc8` and `921a39b3`, both 2026-06-09,
`Co-Authored-By: Claude Opus 4.8`.

**Detection:** `THREW`.

---

**26 — A fatal syntax error shipped with the city pages, same day as the page
itself.** `f1fca4f8` (2026-06-22) added `public/city.html`. Two FAQ entries
used single-quoted JavaScript strings containing an unescaped apostrophe
("What's the age range…", "What's the vibe…") — a fatal `SyntaxError` that
broke the entire module script on both `/philadelphia` and `/lancaster`. No
events loaded, no meta tags, FAQ, or schema rendered; the page fell back to
static, hardcoded "Philadelphia" copy regardless of which city it was.

A second, independent defect in the same file: the schema.org markup tagged
every event with the *page's* city rather than the event's own, so a city
page in fallback mode (showing events from elsewhere because it has none of
its own) emitted structured data claiming those events were local — false
information handed directly to Google.

Fixed the same day, `b5542eee` (2026-06-22).

**Cost:** near zero. A fatal syntax error in a `<script type="module">` tag
fails immediately and visibly on load; same-day introduction and fix is
consistent with this being caught in ordinary testing rather than by a real
visitor. Recorded because "shipped same day as a new page, broke the whole
page" is a pattern worth having on file even when the catch was fast.

**Anchor:** commits `f1fca4f8` and `b5542eee`, both 2026-06-22,
`Co-Authored-By: Claude Opus 4.8`.

**Detection:** `THREW`.

## DECISION — what changed, what didn't

Nothing structural. Each of the three incidents was fixed in the same commit
range that found it, with no standing check, test, or process added against
the *class* of bug it represents (an incomplete cross-module refactor, a
placeholder value shipped as if it were real, a string literal with an
unescaped quote). Nothing here yet resembles a countermeasure — those don't
start appearing in this catalogue until incident 13 in September. June is
where the debt is visible, not where it was paid down.

## What I did not verify

- **The exact number of 3DS guests affected by incident 24's five-day gap.**
  There is no query that reconstructs "guests who hit a 3DS challenge before
  2026-06-07" after the fact; the fix commit's own tests are forward-looking,
  not a backfill.
- **Whether June's traffic during incident 25's four-hour GA4 outage was
  materially different from a typical morning.** No alternate telemetry
  exists for that window to check against.
- **Names, account identifiers and customer records are omitted throughout.**
