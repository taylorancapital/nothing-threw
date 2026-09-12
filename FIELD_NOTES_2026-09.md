# Twenty-three of thirty-one incidents were found by someone digging into something else

**September 2026, through 09-12.** This file has now been written in four
passes. Edition 2 (2026-09-04, PR #438) added incidents 19–22 on top of
edition 1's May–September retrospective (`reports/NOTHING_THREW_2026-09-04.md`)
and said explicitly: *"if `/field-notes` runs again later in September it
should extend this file, not replace it."* The third pass (2026-09-07) folded
in the seven September-dated incidents edition 1 had already catalogued
(01, 02, 06, 07, 10, 13, 14 — see `reports/FIELD_NOTES_2026-05.md` for why a
May incident can be numbered higher than these) and added incidents 29–34.
This pass, 2026-09-12, extends the window from 2026-09-07 through today,
adding incidents 35–48.

**The one-line finding:** fourteen new incidents in five days, one of which is
this document reporting its own countermeasure as unbuilt three days after it
was built — and the month's worst is a nightly that told its operator the
checkout was dead and revenue was leaking, on a week when two tickets had
actually sold.

| | |
|---|---:|
| Incidents, full month | 31 |
| Caught by anything automated | 2 of 31 |
| `LATER` — found by an unrelated dig | 23 of 31 |
| New in this pass (35–48) | 14 |
| …of those, `LATER` | 12 of 14 |

---

## EVIDENCE — how they were caught

| Detection | This pass (35–48) | September (full, 31) | Cumulative, all 5 months (48) |
|---|---:|---:|---:|
| `GATE` — a deterministic check refused it | 0 | 1 | 1 |
| `THREW` — an actual error surfaced | 1 | 1 | 5 |
| `HUMAN` — someone distrusted a number | 1 | 6 | 12 |
| `OPERATOR` — reported as "data went missing" | 0 | 0 | 1 |
| `LATER` — found by an unrelated dig | 12 | 23 | 29 |
| **Total** | **14** | **31** | **48** |

**Twenty-three of September's 31 were `LATER`.** That is not a selection
artifact of the catch-up passes: the incidents keep being found by the
project's own habit of digging into something adjacent — root-cause
investigations, audit closeouts, a workflow's own follow-up edit, a narrower
question about an email quota. That is a real pattern in how this project
catches its own mistakes now, not just how this document finds them.

**The automated share (`GATE`+`THREW`) is not improving.** Per month across
the full five-month catalogue: May 0% (0/1) → June 67% (2/3, small-sample —
two same-day syntax/init errors) → July 50% (1/2) → August 9% (1/11) →
September 6% (2/31). Cumulative at each pass: **11% (2/18) → 9% (2/22) →
15% (5/34) → 13% (6/48)**.

> **Correction, 2026-09-12.** The first version of this paragraph said "every
> edition so far has reported a lower number than the one before it." That is
> false: 9% → 15% is a rise. The claim was published here, in the artifact,
> and in the public repo before being caught by re-reading edition 1's own
> "Two of eighteen were caught by the system itself."

The rise at the third pass was an artifact of the sweep, not of anything
working: backfilling May–July added June's two same-day syntax and init
errors, which are precisely the loud, self-announcing kind, and lifted the
numerator from 2 to 5 in one pass. The honest reading is narrower and harder
to argue with than a trend line: across five months, a 2.7× growth in
catalogued incidents, and every countermeasure built so far, **the share
anything automated catches has stayed inside a 9–15% band and has never
durably improved.** "It is falling" invites the obvious objection that the
denominator grew. "It has not moved" does not.

**Two things are worth separating, because this pass found both.** First,
countermeasures *are* being built — the listing-redirect `--check` became a
real CI step on 2026-09-09 (incident 41), the price bug got a regression test
(39), the Resend rejections got a counter (44). Second, none of them caught
anything in this window, because they are all built *after* the incident they
are named for, one at a time, and the failures that dominate this month are
wrong conclusions drawn from correct data rather than wrong code. A test can
assert that `framesForRow` prices by the row's date. Nothing asserts that a
report's headline is true.

**And the ratio itself has now been measured wrong once.** Incident 40 is
this document's previous pass reporting a countermeasure as unbuilt three
days after it had been built. That error ran in the pessimistic direction —
it under-counted a fix — which is the direction least likely to be noticed by
anyone reading a failure catalogue. The numbers above are the best reading of
what is on the record; they are not audited, and the one time anybody checked
one against the live system, it was wrong.

## MECHANISM

### A. The agent was confidently wrong — 15 incidents

**01 — Two ID spaces that never match.** 2026-09-02. An analysis compared an
ad creative's `video_id` against the `object_id`s in a video-engagement
audience. The first is the source upload; the second is the delivered
rendition — the platform's auto-generated crops. They can never match, even
for the same video.

**Cost:** a false headline that the retargeting audience contained none of
the running videos, an entire "the funnel was never wired up" conclusion
built on top of it, and a pull request. The audience was correctly scoped the
whole time. **Detection:** `HUMAN`.

---

**02 — A share of 105%.** Found reviewing PR #400, 2026-09-02. Meta's
insights API returns different conversion counts for the same window with
and without a gender breakdown. A report divided a breakdown numerator by an
un-broken denominator, printing "women's share of landing-page views:
105.00%."

**Cost:** published and nobody noticed. The impossible figure was the
visible tip; every other gender-split cost in that report sat on the same
mismatch and looked fine. `scripts/meta-ads-review.js` now flags any
breakdown-vs-total disagreement over 2% by name. **Detection:** `LATER`.

---

**19 — 245 GA4 fields declared empty without probing them.** 2026-09-04. An
audit of the analytics property's coverage concluded that 245 fields were
"structurally empty because every ad we run is on Meta," written from
category names rather than from probing the fields. Wrong three separate
ways: the `sessionGoogleAds*` family is real and populated (a Google Ads
account is in fact linked); `sessionSa360Medium`/`sessionDv360Medium` return a
literal `"cpc"`/`"cpm"` on every row regardless of real source — constant
placeholders, more dangerous than empty because they look like data; and 195
of the remaining fields return `(not set)`, a different fact with a different
remedy.

**Cost:** $37.91 of live Google Ads spend stayed unread for months (incident
20 carries that cost; not counted twice here). The durable cost is the
method — triaging an API's fields by category name produces confident,
checkable, wrong claims. **Anchor:** memory `ga4-fields-that-are-switched-off`,
rewritten 2026-09-04T14:54Z with an explicit correction block. **Detection:**
`LATER`.

---

**20 — A capability probe that tested metrics in isolation.** 2026-09-04.
GA4's `advertiserAd*` family and `returnOnAdSpend` error when queried alone
("Please add sessionCampaignName to make the request compatible"). A probe
that tests metrics one at a time concludes they are unavailable; they are
not — they work paired with a campaign dimension.

**Cost:** $37.91 of Google Ads spend invisible to every report written
before 2026-09-04, $35.35 of which bought 110 clicks and zero attributed
sessions — unknown, because Meta spend auto-syncs and `recurring_costs` is
hand-entered, so Google spend landed in neither path. Every CAC figure in
every prior report is understated by that amount. **Detection:** `LATER`.

---

**21 — Correlation presented as causation.** 2026-09-04. A live read of
every campaign found `OUTCOME_SALES` holding all 5 purchases on $373.49 and
`OUTCOME_TRAFFIC` holding 0 on $254.14 and 1,156 landing-page views,
presented as establishing that the traffic objective does not sell. Tested
per dollar, p = 0.224 — with six lifetime conversions the account cannot
support a causal claim in either direction.

**Cost:** none realised. Taylor asked whether it was causal; the finding was
corrected the same day. **Detection:** `HUMAN`.

---

**29 — A wrong headline reached `main` and had to be retracted.** 2026-09-04,
21:45 EDT, PR #446. The 2026-09-04 delivery diagnosis
(`reports/META_ADS_DELIVERY_DIAGNOSIS_2026-09-04.md`) read Meta's attributed
conversions as *sales* and concluded "the sales stopped nine days ago." They
had not. In the exact window the report called "$193.25 spent for zero," 14
paid tickets worth $381.12 actually sold. Marion Court — the event the report
called dead — had 11 paid tickets, $282.18, with its best week the most
recent one.

The category mistake: Meta sees roughly 11% of real ticket sales (6 of 53
for the window in question). A complete spend-to-revenue join already
existed in Firestore (`ad_spend.byEvent` crossed with `tickets.eventId`) and
nobody had read it. Gross ROAS for the period was 1.79, not the 0.23 the
retracted report implied.

**Cost:** a wrong "the business is dying" headline reached the main branch
of the repository that runs this business, before being retracted the same
evening. No ad account change was made on the strength of the wrong number
before the retraction, per the retraction commit itself — the near-miss is
real regardless. **Anchor:** commit `c5d1cfd4`, 2026-09-04 21:45:12 EDT, "Two
attribution bugs, and a retraction of yesterday's headline (#446)"; memory
`meta-attribution-is-not-sales`. **Detection:** `LATER`.

---

**30 — Two more attribution bugs found in the same pass.** Same commit, same
evening, 2026-09-04. Two separate, previously-undetected defects in the
attribution pipeline itself, distinct from incident 29's category mistake:

- **A string literal, not a regex literal.** `'(^|;)\s*' + name +
  '\s*=\s*([^;]+)'` is a plain JS string; `\s` is not a recognised string
  escape, so it silently collapses to the letter `s`. The resulting pattern
  cannot match `document.cookie`'s `"; "`-separated entries unless the target
  cookie happens to come first. Proven by execution: with `_fbp` first it
  read correctly; otherwise both `_fbp` and `_fbc` returned `null`. The
  earlier claim that "only 1 of 131 tickets carries an `fbc`" was this reader
  bug, not a real signal about ad-driven buying — and every CAPI Purchase
  sent to Meta in the meantime went with degraded match quality. Fixed on all
  five public pages.
- **First-touch attribution had no expiry.** Three pages guarded on
  `!localStorage.getItem('sparkdate_attr')` with no TTL, so a visitor whose
  first-ever touch was an Eventbrite listing click in June stayed tagged
  `eventbrite / listing` for the life of the browser, through any number of
  later ad clicks — which is why 6 of 9 own-site buyers read
  `direct`/stale source instead of their real, recent one. Now ages out at 30
  days.

**Cost:** an unknown span of CAPI sends with silently degraded match
quality (bug one), and an unknown number of real ad-driven purchases credited
to a stale first touch instead (bug two) — both live since whenever the
original attribution code shipped, both found only because the same
investigation that retracted incident 29's headline went adversarial on its
own pipeline rather than stopping at the first correction. **Anchor:** commit
`c5d1cfd4`, same PR as incident 29; new test `attribution-wiring.test.js`
extracts and executes the actual cookie regex from each page to prevent
regression. **Detection:** `LATER`.

---

**31 — A leak recalculated at a quarter of its first estimate.** 2026-09-04,
corrected 2026-09-05. `targeting_automation.individual_setting.gender: 1` is
Meta's Advantage+ audience gender expansion, and it overrides an explicit
`genders: [2]` targeting restriction — real, and Ads Manager turns it on by
default in the campaign-creation flow. The first measurement compared
lifetime insights against each ad set's *current* targeting and found
$241.81 of apparent cross-gender delivery. That comparison is invalid for any
ad set ever edited, and most were: the Graph API's `targeting` object returns
present-day state, not the state live when the spend happened.

Redone against `GET /<adset_id>/activities`, splitting each ad set's daily
spend at its own retargeting-change timestamp: $180.56 of the original
figure was spent while the ad set had **no gender restriction at all**,
before the lock existed. Only $57.53 landed after a female lock was
genuinely in force — a real leak, at roughly a quarter of the first number.

**Cost:** a headline dollar figure overstated by about 4×, sitting in a
memory file being cited, until a more careful re-measurement caught it. No ad
change was made on the original figure before the correction. **Anchor:**
memory `gender-expansion-serves-men`, corrected block dated 2026-09-05;
`reports/META_ADS_DELIVERY_DIAGNOSIS_2026-09-04.md` §7g. **Detection:**
`LATER`.

---

**32 — A parity audit assumed the wrong side was the reference.** 2026-09-06,
closeout of PR #453 (audit) via PR #456. Three checkouts are hand-copied with
no shared client bundle: `events.html` (a modal, treated throughout the audit
as *the* reference), `event.html`, and `lp.html` (the paid landing page,
excluded from the primary comparison). Framing every finding as "bring the
target in line with the reference" assumed `events.html` was correct by
default. In the closeout, four confirmed defects were in `events.html`
itself, and in every one of them `lp.html` — the file nobody treated as
canonical — had it right: the re-entrancy guard preventing a double Stripe
charge, guarding `Stripe()` so a blocked `js.stripe.com` cannot kill the page
module, matching the sold-out `409` string the server actually sends, and
sending `ref` so referral credit is not silently dropped.

Separately, the same audit's bulk pass (196 candidate findings) was measured
against its own verification: 43 of the first 100 re-checked findings did
not survive, including one that called a sticky card unreachable for "~62% of
the page's scroll" when the shipped page at 1366×768 measured 32%, reachable
from scrollY 1400 — a fix was written against the overstated figure, then
reverted once measured.

**Cost:** a near-miss rather than a realised one — had the audit's fixes
been applied in the direction it assumed, at least four real guards (one of
them a double-charge guard) would have been removed from the file that
actually had them correct, in the name of "fixing" it to match a file that
did not. Caught during the closeout itself, before any fix shipped in the
wrong direction. **Anchor:** memory `lp-html-is-often-the-correct-side`,
`checkout-audit-refute-rate`; PRs #453, #456, both 2026-09-06. **Detection:**
`LATER`.

---

**35 — The nightly told its operator the checkout was dead, in a week two
tickets sold.** 2026-09-11. The unattended 02:00 analysis wrote, as its
headline: *"No purchase has completed anywhere on the site since
2026-09-01 — confirmed independently by GA4's own transaction ledger and by
Meta's pixel data, while checkout-starts kept climbing on every landing page
and ad spend (~$130/week) kept flowing in."* It blamed the 09-02 checkout
rebuild and routed itself as an **urgent NEEDS TAYLOR INPUT** item, on the
grounds that deciding whether real revenue was being lost needed a live check
the session could not perform.

Two purchases had completed on 09-08. Firestore `tickets` held both, Vercel
logged `POST /api/purchase-ticket 200` for both, and Meta's own pixel receipts
showed `Purchase` in both hours. GA4 had no event from either buyer, because
GA4's `purchase` fires only from the buyer's browser — there is no
server-side path, so a buyer whose browser drops the tag is a real sale GA4
can never record. Neither buyer came from an ad, so ad insights could not see
them either. "Confirmed independently by two sources" was two views of the
same blind spot.

The second half of the headline was a tautology. The nightly diffs the
window-to-date `ga4-api-events-*.csv` tables between pulls; those only ever
rise, so "begin_checkout kept climbing while purchase froze" is what that
comparison always prints when the only two purchases in the window were both
missed. Per-day `begin_checkout` had in fact *fallen* after 09-02, about 6/day
against 12–17/day in late August — the opposite of the stated trend.

**Cost:** a false revenue emergency escalated to the operator by name, with a
merged PR behind it, against a live business. Retracted in-file in the same
PR. This is the same *shape* as incident 29 — a report reading an
attribution surface as ground truth for sales — through an entirely different
mechanism, nine days later, in the automated path rather than a hand-written
one. **Anchor:** `Night Tasks/logs/2026-09-11.log` line 130 (the headline as
written); commit `54e2c646`, "GA4 missed both 09-08 sales; the nightly's
'checkout sold nothing in nine days' headline is retracted in-file (#528)";
memory `ga4-purchase-count-is-not-sales`. **Detection:** `LATER`.

---

**36 — "Never run on this account," about the channel that was outselling
Meta 3:1.** 2026-09-11. Asked for the second time in four days what a better
alternative to Meta ads would be, a session wrote that Eventbrite Ads had
never run on this account. It had run on **every event since June** — 13
campaigns, $436 lifetime, 556 clicks, 21 attributed tickets at $20.77 each,
against Meta's $1,371 for 6 attributed purchases.

The claim was written without checking, in the one session commissioned to
find a better channel than Meta, about the paid channel already performing
best. Taylor's own framing in that conversation — *"rn I'm doing very small
budgets to get tickets in the geographies"* — was about Eventbrite Ads, and
was read as being about Meta.

**Cost:** a channel-alternatives analysis that omitted the only paid channel
on the account that was working, delivered to the person who had asked
precisely that question. Corrected within the session; the channel is now
synced nightly into `ad_spend/{date}__eventbrite` with 70 documents
backfilled. **Anchor:** memory `paid-alternatives-all-lost-to-meta`;
`reports/CHANNEL_ALTERNATIVES_2026-09-11.md` (commit `eb07acc9`, #534); the
sync built in response, commit `0c366c04` (#539). **Detection:** `HUMAN` —
Taylor's own description of what he was spending on contradicted it, and he
tested the undocumented endpoints himself.

---

**37 — A bug fixed in prose, not code.** 2026-09-07. The 09-06 nightly report
flagged a "$14.46 Google Ads still accruing" figure as a script bug. The
correction was written into that night's report text. The script was not
touched, so the next night's run **reproduced the figure verbatim**.

The root cause, found only on the second occurrence: `ga4-nightly-summary.js`
took the last 7 dates *present in* the Google Ads table rather than the last 7
calendar days. The account has been dark since 2026-07-24, so "last 7"
resolved to six-week-old rows that happen to sum to exactly $14.46. Real
state, unchanged since 08-25: $0 spend since 07-24, $37.91 lifetime.

**Cost:** a live-looking spend figure for a dormant ad account, published two
nights running, in the standing summary the operator reads first. The durable
cost is the method — correcting the output of a generator, in the generator's
output, leaves the generator wrong. **Anchor:** commit `4c697806`, 2026-09-07
13:47:29 EDT, "Same failure two nights running, and a bug fixed in prose, not
code (#476)"; fixed in code by `8b1ded04` (#478). **Detection:** `LATER`.

---

**38 — Three listing surfaces recorded as unworked while already carrying
live events.** 2026-09-10. `content/listing-sites.json` — the registry that
decides where events get syndicated — described LancasterOnline as
`not_pursued`, Nextdoor as "never been used", and Patch as `dormant`. All
three were wrong, and had been for some time: LancasterOnline had been live
since 2026-09-02 via Evvnt and its newsletter produced **136 sessions in a
single day**, larger than any other free surface on record except Eventbrite's
whole-window total; three Marion Court listings were already up on Nextdoor,
with the "bounces to login" gotcha in the registry being an artefact of
checking while logged out; and both current events were on Patch's Lancaster
calendar at the time it was recorded dormant.

**Cost:** the largest free acquisition channel the business has was recorded
in its own registry as never tried, so nothing was built on it and nothing
measured it. The registry's own `_evidence` line ("Zero. No meetup, no
allevents, no discoverlancaster, no lancasteronline…") was corrected in place
rather than deleted, so the date it stopped being true stays visible.
**Anchor:** commit `7e7ede31` (#485), 2026-09-10 01:13:13 EDT; memory
`lancasteronline-is-the-biggest-free-channel`. **Detection:** `LATER` — found
while syndicating an unrelated event.

---

**39 — Slide art priced every post by the day its sheet was rendered.**
2026-09-10. `framesForRow` in `scripts/build-campaign-export.js` priced a
carousel's fact frame — the slide carrying date, venue and "Doors · $price" —
with `currentPrice(pricing)`, which defaults to *today*. Sheets are rendered
weeks ahead of the posts they carry, so the price baked into the image is the
price on render day, not on post day.

Five approved Loxleys carousels (LX-17, 18, 19, 22, 23, square and TikTok
variants) carried **$24.99** on art scheduled to publish after that event's
early bird ended on 09-07, with `brand.json` holding $29.99 for all of their
post dates. The captions were correct; only the images were wrong. The queue
lint checks that a *caption's* price is one of the event's prices, and nothing
in the pipeline reads a price inside an image.

**Cost:** approved, scheduled social posts advertising a live event $5.00
below its actual ticket price. Fixed to use the row's own date, with a
regression test at `tests/campaign-export-price.test.js` — but the fix only
covers new renders, and the repair of the existing art cascaded through three
more failures (see incidents 43 and 45, and the count error below).
**Anchor:** commit `af8103d2`, 2026-09-10 21:16:12 EDT, "Slide art priced
every post by the day its sheet was rendered (#519)"; memory
`slide-art-price-is-baked-at-render`. **Detection:** `LATER`.

A tail worth recording separately, because it is the same mistake one level
up: the repair was scoped to "the five posts with the price bug" and the real
number was **eight** — a later re-export (#537) had made LX-20, LX-21 and
LX-25 stale too. The list was re-derived from the previous count instead of
from what had changed since.

---

**40 — This document reported a countermeasure as unbuilt three days after it
was built.** 2026-09-07, found 2026-09-12. The previous pass of these field
notes stated, twice, that the 02:00 task's `StartWhenAvailable` and
`RunOnlyIfNetworkAvailable` flags "are still both `False`" and that fixing
them "needs an elevated shell nobody has run," and carried incident 10 —
8 of 23 nights never running — forward as open and unmitigated.

Read live against the scheduled task on 2026-09-12:

| flag | previous pass said | actual |
|---|---|---|
| `StartWhenAvailable` | `False` | **`True`** |
| `WakeToRun` | not mentioned | **`True`** |
| `RunOnlyIfNetworkAvailable` | `False` | `False` (correct) |

`HANDOFF.md` records Taylor setting the first two in an elevated shell on
**2026-09-04**, three days before that pass was written, and the log directory
corroborates it independently: every night from 2026-09-04 to 2026-09-11 has a
nightly log — eight consecutive, against 8 missing out of 23 before the
change. The claim was checkable at the time from two places and was checked
against neither.

**Cost:** no operational loss — the fix was already in place and went on
working. The cost is to this document. A failure catalogue whose entire
argument rests on one ratio — how many failures anything automated catches —
reported a fix as unbuilt, and did so in the pessimistic direction, which is
the direction a reader of a failure catalogue is least likely to challenge.
**Anchor:** `reports/FIELD_NOTES_2026-09.md` as of commit `57be2a36`,
§DECISION and §C-10; `Get-ScheduledTask "Meta Ads Results Pull"` read
2026-09-12; `HANDOFF.md`, "Open threads nobody owns". **Detection:** `LATER`.

### B. The check passed for a reason unrelated to correctness — 6 incidents

**06 — Six sources agreed, and all six were wrong.** Built 2026-09-01, after
one afternoon turned up four different end times for the same event and
three descriptions of the run of show. An audit of the business's public
claims found six surfaces describing the event format identically. One
social caption disagreed with all of them — and was the only correct
description anywhere in the codebase.

**Cost:** months of marketing copy describing the product incorrectly. A
fact on six surfaces has six chances to be wrong and one chance to be
noticed. `node scripts/audit-facts.js` now cross-checks 26 public surfaces
against one named canonical value per fact — deliberately not in CI, because
several of its checks report things that are correct-but-worth-knowing and
would go permanently red. **Detection:** `HUMAN`.

---

**07 — The test command that never exits.** 2026-09-04. The package's
`test` script was bare `vitest` — watch mode. Run unattended it produces an
empty output file and holds until the timeout.

**Cost:** about seven minutes, burned looking like a hang rather than a
misconfiguration. Use `npm run test:ci` (`vitest run`), never `npm test`,
unattended. **Detection:** `HUMAN`.

---

**34 — A verification pass that never ran, read as a pass.** 2026-09-07. A
design workflow ran 13 agents: 3 proposing competing schema designs, the rest
spec-checking and mechanics-verifying them. Nine of the 13 hit a session
limit mid-run, including every mechanics verifier and the synthesis step.
Two of the three proposals came back from the workflow with `objections: []`
and `survives: true` — not because they passed review, but because no
verifier for them ever executed. `agents_error` in the workflow's own usage
block was the only field that said so; nothing else distinguished "reviewed
and clean" from "never reviewed."

**Cost:** none realised — caught by reading the usage block before acting on
`survives: true`, and the surviving proposal was checked by hand afterward
(see incident 33). The near-miss is the pattern this whole catalogue is
about: an empty objections list reads exactly like a pass, and here it was a
pass with no examiner in the room. **Anchor:** memory
`workflow-agents-can-write-files`, 2026-09-07. **Detection:** `LATER`.

---

**41 — A CI gate that existed only in a docblock.** Found 2026-09-09.
`scripts/build-listing-redirects.js` has advertised its `--check` flag as
*"CI: fail if stale"* in its usage block since the day it was written.
Nothing in `.github/workflows/` ever invoked it. The flag was real; the CI
step was not, and the docblock was the only thing asserting otherwise.

TL2 (Tellus AfterDark) was added to `content/brand.json` on 09-05 and
`vercel.json` was never regenerated, so **all 18 of its `/l/` short-link
routes did not exist**. `build/listing-pack.md` composes those links from the
registry and prints them, so they looked real at every point a human would
check. They were published to live listing sites in that state.

**Cost:** a Nextdoor post for TL2 went live carrying
`sparkdate.date/l/tl2-nextdoor-event`, which served the 404 page, for roughly
four days before anyone loaded one. This is the precise failure the short
links were introduced to prevent, described in the same script's own docblock
as "a working link that reports no attribution — the worst failure shape
available," one level worse: not a mangled link, an absent one. **A real gate
was built in response** — `.github/workflows/test.yml:62` now runs
`--check` on every build, added 2026-09-09 with the reason recorded in a
15-line comment above it. **Anchor:** commit `7e7ede31` (#485);
`.github/workflows/test.yml:44-62`. **Detection:** `LATER`.

---

**42 — Pre-filled asset names slipped past the guard that refuses art-less
rows.** 2026-09-08, cleared 09-10. A session filled `asset_files` on 13 TL2
rows in `content/queue.csv` with the filenames it expected the art to have.
The convention is that the field stays empty until the art exists; the lint
passes either way. Three things broke, with no error anywhere:

- `scripts/design-handoff.js` drops any row that already names files, so the
  TL2 Claude Design brief — the document that tells a designer what to make —
  came out as **1 post and 0 slides**. Cleared, it is 14 posts and 35 slides.
- `prep-social-assets.py` only appends a discovered export whose shape the row
  does not already name, so the real exports, when they arrived, would have
  been ignored.
- `social.js approve` refuses art-less rows by checking for an **empty**
  field. Rows naming art that did not exist would have passed it.

**Cost:** a design brief that asked for 1 of the 35 slides actually needed,
and a near-miss on the only gate standing between the queue and publication —
the check that exists specifically to stop a post going out without art would
have waved through 13 rows with no art, because it tests for the absence of a
string rather than the presence of a file. Caught before any of those rows
reached `approve`. **Anchor:** commit `af8103d2` (#519), which cleared them;
introduced in #493; memory `queue-asset-files-empty-until-art`.
**Detection:** `LATER`.

---

**43 — A cache-busting fetch verified a URL nobody would ever load.**
2026-09-12. After incident 39's corrected art was deployed, the check that it
had actually reached the world was a fetch with a `?cb=` cache-buster
appended. That request bypasses the CDN by construction: it is a different
cache key, so it returns the freshly deployed file and reports success — while
the real URL, the one in the scheduled post, keeps serving the old image.

`vercel.json` sets `Cache-Control: public, max-age=604800,
stale-while-revalidate=2592000` on every `/(.*)\.(svg|png|jpg|jpeg|webp|ico|woff2)`,
so everything under `/social/` is edge-cached for **seven days**. Hours after
the corrected art deployed, three of five replaced TikTok frames served the
new image and two — `LX-17_4of5_tt` and `LX-19_2of3_tt` — were still serving
$24.99 at `age=4068`, pinned and not moving on repeat requests.

A second verification defect sits underneath it: whole-image correlation
between the old and new fact frames differs by only about 0.003, because the
two images are identical apart from the price digits. That is well inside
noise, so the comparison "confirms" whichever answer you expected. Masking to
the pixels that actually differ and scoring inside that box separates them
properly — 0.98 against 0.80, with the crop legibly reading $29.99.

**Cost:** none realised beyond the stale window itself, because the mismatch
was caught. But the same edge cache is named as having served a superseded
slide for a week once already (MC-15), and the verification method in use
would have reported that as fixed on day one. **Anchor:** `vercel.json:64-69`
(header confirmed live, 2026-09-12); memory
`slide-art-price-is-baked-at-render`. **Detection:** `LATER`.

### C. Silent failure — the error was swallowed and read as absence of data — 4 incidents

**10 — A nightly that simply did not run.** Two flags on the scheduled
task — start-when-available and run-only-if-network-available — were both
false, measured against the nightly's operating window of 2026-08-13 through
this edition's 2026-09-04 writing date (23 nights). A machine asleep at the
trigger time, or online but without a network yet, produced no run and no
notice.

**Cost:** 8 of 23 nights lost. One run failed both network-dependent steps
fourteen minutes after a boot. The gap was only visible by counting log
files against a calendar. **Recurred 2026-09-03** — no `2026-09-03.log`
exists, while `review-2026-09-03.log` from the unrelated 09:00 review task
does, which would answer "is there a log for 09-03" with a misleading yes.
See Recurrences below. Not yet fixed: both flags require an elevated shell
that nobody has run. **Detection:** `LATER`.

---

**22 — The nightly's two halves run different versions of the code.**
2026-09-04. `run-nightly-claude-code.ps1` pulls data in steps 1–3 from the
main checkout's working tree, and runs its analysis in step 6 from a branch
cut fresh from `origin/main`. The two can silently disagree. PR #430 merged,
taking the GA4 pull from 28 tables to 46; a `-Force` run minutes later still
wrote 28 files, because the main checkout was five commits behind and had
not been pulled that session. The resulting report (PR #433) was written by
current analysis code against stale data — the run exited 0, the gate passed
it, and it opened a PR. Nothing in the log said the inputs were old, because
there is no line for it to write.

**Cost:** one full nightly report built on data a merged change should have
replaced. **Not fixed** — recurs after every merged pull-script change until
the launcher fast-forwards the main checkout or the pull runs out of the
nightly clone instead. See Recurrences for a second, independently-discovered
mechanism with the same root cause. **Detection:** `LATER`.

---

**44 — A Resend rejection counted as a skip, and a refused match email logged
as sent.** 2026-09-10. `resend.emails.send()` resolves with `{ error }` on a
4xx or 5xx — it does not throw. Every `catch` wrapped around a send in this
codebase was therefore dead to rejections, and the `else` branch was the only
place one could surface. Six of the seven passes in `cron-send-emails.js`
wrote that branch as a bare `else { skipped++; }` with no log, folding a
**refused send** into the same counter as "already registered" and
"unsubscribed."

`declare-connection.js` was worse. `notifyMatch()` awaited
`Promise.all(sends)` and then logged "match notified" unconditionally, so a
refused match email — the highest-value message this product sends, the one
telling two people they matched — was lost permanently behind a success line.
A match where neither side had an email address ran `Promise.all([])`, which
resolves, and logged as notified having sent nothing at all.

**Cost:** nothing is known to have been lost on the day it was found — the
09-09 run reported 100 sends and zero errors, and two match emails sent 19
minutes after the quota notice were delivered two seconds later. That is the
point: **the summary line would have read exactly the same if every send past
the cap had been refused.** How many refusals this swallowed over the life of
the code is not recoverable, because nothing wrote them down. Now a shared
`logRejected()` writes one line per refusal and each pass counts rejections
separately from skips; the lock records `notified` true/false plus
`notifyError`. **Anchor:** commit `dc89d901`, 2026-09-10 02:04:16 EDT, "A
Resend rejection counted as a skip, and a refused match email logged as sent
(#492)"; memory `resend-quota-notice-is-a-warning`. **Detection:** `LATER` —
found while answering a narrower question about a quota notice.

---

**45 — An existence probe that read an unrelated API error as "gone."**
2026-09-11. Clearing the stale $24.99 Facebook posts from incident 39 required
confirming each one had actually been deleted. The probe asked the Graph API
for `fields=is_published` and treated any error as proof of absence.

Queue rows whose post id carries no `<page>_` prefix (LX-16, LX-21, LX-25) are
a different node type that has no `is_published` field at all, and answer
`(#100) Tried accessing nonexisting field` — an error about the *query*, not
about the object. Read as absence, it **reported LX-21 and LX-25 deleted while
both were still scheduled**, still carrying the wrong price, in the one
exercise whose entire purpose was removing exactly those posts.

Compounding it, Facebook's own UI lies in the other direction on these
deletions: it shows "Unable to delete your post / Something went wrong" while
deleting the post anyway, seen on LX-17 and LX-18 the same day. So the UI
reports failure on success, and the probe reported success on failure.

**Cost:** two live scheduled posts advertising a $5.00-stale price were
recorded as removed and left in place; caught and cleared in #545. The correct
probe asks for `id` **alone** — `(#10) Object does not exist` then means
deleted — cross-checked against `/<page-id>/scheduled_posts`. **Anchor:**
memory `facebook-delete-error-is-a-lie`, 2026-09-11; commit `65d80081`, "Clear
the last six Loxleys fb ids; every stale Facebook post is gone (#545)".
**Detection:** `LATER`.

### D. Destructive, or unreported, writes that report as fine — 5 incidents

**13 — Replacing an array that should have been appended to.** Settled
2026-09-02, attaching a tracking pixel to two live Marion Court Traffic ads.
The natural implementation copies the shape used at ad creation, which sets
the tracking array to the pixel entry alone. Those live ads already carried
seven entries, including the one every landing-page-view number came from —
writing the pixel spec alone would have replaced the whole array and dropped
all seven.

**Cost:** none, because it was caught first — but Meta returns success
either way, and the damage would have shown up as metrics quietly going to
zero. `npm run ads:pixel` now dry-runs by default, appends, reads the field
back rather than trusting the write, and warns if the count shrank. The only
incident in this whole catalogue caught by an actual deterministic gate.
**Detection:** `GATE`.

---

**14 — A config loader that clobbered a working credential.** Early
September, ads-scripting work. The checked-in `.env.local` holds
two-character placeholders; the real ~200-character tokens live in the shell
environment. A one-off script loading that file the usual way overwrote a
good token with an empty one.

**Cost:** a failure that presents as `(#200) Provide valid app ID` — which
reads like a scopes or app-registration problem and sends you to the token
debugger for nothing. The fix (PR #374, 2026-09-01) is one conditional: only
set a variable if it is not already set — applied first to the Firebase
credential path, the same pattern this incident needed for the Meta one.
**Detection:** `HUMAN`.

---

**33 — A workflow wrote to a shared file when it was only asked to return a
proposal.** 2026-09-07. A design workflow ran three parallel agents, each
asked to design and *return* a competing `content/brand.json` schema
proposal as structured JSON. One of the three additionally wrote its
proposal directly into `content/brand.json` — about 47 lines, uncommitted, in
a file every other part of the pipeline reads — and nothing in the
workflow's own result mentioned it. It surfaced only because a later,
unrelated `Edit` failed with "string to replace not found": the anchor point
already had a `creative` block sitting after it that had not been there when
the file was last read.

**Cost:** none realised — the write's content turned out to be good work and
was kept after review, and no other file was touched. But that it caused no
damage this time was luck: nothing distinguishes "a subagent with full tool
access decided to also just do the thing" from a genuine result, unless
`git status` is checked immediately after every workflow run and before
trusting the working tree. **Anchor:** memory
`workflow-agents-can-write-files`, 2026-09-07. **Detection:** `LATER`.

---

**46 — A `HANDOFF.md` edit that ate another session's live thread.**
2026-09-08. A script updated `HANDOFF.md` by replacing everything between its
own entry and the next entry it recognised. Another session had inserted a
bullet in between — a live MC-12/MC-13 thread **with a same-day deadline in
it** — and the span replacement silently swallowed it.

Nothing caught it. `HANDOFF.md` is one flat list of bullets with no ids, no
test covers it, the lint does not read it, and a reviewer sees a large diff on
a file that always has large diffs. It **merged to `main` in #482** and was
found later by diffing `origin/main`. The repo's own contract says "never
delete another session's entry to make room"; the failure was not intent, it
was the edit shape — anchoring on a boundary you do not own is correct exactly
until someone writes between the anchors, which is this file's normal state,
since every session prepends to it.

**Cost:** another session's live work item, carrying a same-day deadline,
deleted from the shared handoff file and merged to the main branch. Recovered
verbatim from the pre-merge commit and restored in #483, with the restoration
noted in the text — a thread that silently reappears reads as continuous when
it was not. **Anchor:** commit for #482, 2026-09-08; recovery commit
`34dc15e6`, "handoff: restore an entry #482 deleted, and close it out with
evidence (#483)"; memory `handoff-span-replace-eats-entries`. **Detection:**
`LATER`.

---

**47 — Fixing 18 dead links deleted 19 live ones.** 2026-09-09/10. The remedy
for incident 41 was to run `build-listing-redirects.js --write`, which
regenerates the entire `/l/` block in `vercel.json` from `fetchUpcomingEvents()`
— *upcoming* events only. Marion Court had run on 09-08. The generator dropped
it the instant its start time passed, so the same run that created TL2's 18
missing routes **removed all 19 `/l/mc-*` routes**.

Those were not dead weight. They were live inside published listings on Patch,
AllEvents and Nextdoor, which stay up until a human takes them down. Every one
of them turned into a 404 overnight — the identical "listing looks fine, goes
nowhere" failure the script's own docblock was written to prevent, and the
identical failure being repaired, at larger scale, caused by the repair.

The write reported success. Nothing in the run flagged that it had removed 19
routes; the count was recovered by reading the diff.

**Cost:** 19 live listing links, on three third-party sites outside this
project's control, serving 404 for at least a night. A 404 is the worst
available outcome here — the event page still returns 200 for a past event,
says "no longer"/"ended", and puts upcoming events one click away, so a stale
listing pointing at it still recovers the visitor, where a 404 loses them.
Flagged rather than hand-patched, because the script's docblock forbids
hand-editing a `/l/` entry; the open question is whether the generator should
keep recently-past events for a grace window. **Anchor:** commit `7e7ede31`
(#485), third commit message in the chain, 2026-09-10. **Detection:** `LATER`.

### E. Concurrency — several agents, one working tree — 1 incident

**48 — Two sessions built the same fix, from the same review, on the same
day.** 2026-09-10. A session was handed the admin-dashboard ticket-cap fix.
The session brief printed at its own startup already listed a worktree named
`admin-ticket-cap`, marked `fresh, no commits of its own` — a second session
on the identical task, cut from the same review's DECISION list. Both wrote a
`startAfter` pager for the same collection.

`fresh, no commits of its own` reads as idle and means the opposite: a session
that has started and not committed yet. The name was the tell — session
worktrees get random slugs like `sweet-murdock-d8dfb5`, so a descriptively
named worktree matching your task is somebody's claim on it.

**Cost:** a full duplicate implementation. The other session's work merged as
#502 while this one sat in CI, and this one came back `CONFLICTING`. Recovered
by resetting to `origin/main`, reading their diff, and keeping only what this
one added — which turned out to be the two things their commit message said it
was leaving out — as a small additive PR (#503) instead of a conflicted
duplicate. Their version was also the better one: it paged both collections
and deduped by document id, and this one did neither. **Anchor:** PRs #502 and
#503, 2026-09-10; memory `brief-lists-other-sessions-tasks`. **Detection:**
`THREW` — GitHub marked the second PR conflicting after the first merged.
Nothing warned before the work was done, and the signal that would have
prevented it was printed at startup and read as idle.

## DECISION — what changed, recurrences, what's still open

**From edition 2's window (through 2026-09-04 20:10 EDT):**

- **`/field-notes` itself** (#437) — this document's own command, plus a
  monthly read-only sweep task. Three defects in it were found by running it
  by hand before it ever fired unattended, including the incident-10 false
  negative described below.
- **Customer PII removed from the public repo** (#434) — ten tracked files
  carried real attendee names and email addresses. Pseudonymised rather than
  redacted, because the duplicate-name pairing was itself a finding. Not an
  agent failure; recorded because it happened in the window.

**Nothing new was built in response to incidents 29–34.** All six surfaced
in the three days before this edition was written, and the fixes so far are
narrow and local: the two attribution bugs (30) got a regression test that
executes the actual cookie-parsing regex against realistic input; the pixel
tool (13, already fixed in September's first week) remains the only
purpose-built gate in the whole catalogue. Nothing generalizes "a workflow
can silently write files" (33) or "an empty objections list can mean nobody
checked" (34) into a check that would catch the next occurrence — both are,
for now, single incidents with a lesson written down and no enforcement
behind it.

**Built in response to incidents 35–48, during the window itself:**

- **A real CI gate for stale listing redirects** (#485, 2026-09-09) —
  `.github/workflows/test.yml:62` now runs `build-listing-redirects.js
  --check` on every build, closing incident 41. This is the second
  purpose-built gate in the entire catalogue, after the pixel tool (13). It
  exits 0 with a warning if the site is unreachable, so an outage cannot turn
  an unrelated PR red.
- **A regression test on slide pricing** (#519) —
  `tests/campaign-export-price.test.js` asserts the fact frame is priced by
  the row's date. It covers new renders only; nothing reads a price inside an
  already-rendered image, which is the gap incidents 39, 43 and 45 all sit in.
- **Rejection counters on every email pass** (#492) — refusals are now counted
  and logged separately from skips, and the match lock records
  `notified` plus `notifyError`. This does not prevent a refusal; it makes one
  visible, which is the whole of what incident 44 lacked.
- **A server-side GA4 purchase** (#531) — `lib/ga4-mp.js` sends `purchase`
  from the Stripe webhook via Measurement Protocol, which would have prevented
  incident 35's blind spot. **It does nothing until `GA4_MP_API_SECRET` is set
  in Vercel production**, which is Taylor's to do and is not done. Counted as
  built, not as working.
- **Eventbrite Ads synced nightly** (#539) — the channel incident 36 declared
  nonexistent now writes `ad_spend/{date}__eventbrite` every night, with 70
  documents backfilled.

**Still nothing generalises.** Every item above is named for the single
incident that produced it. Nothing checks whether a report's headline is true,
which is the failure class that dominates this month (35, 36, 37, 38) and the
one with the largest realised cost.

**Recurrences.**

*Carried from the previous pass:* incident 22's root cause resurfaced
2026-09-06 through a second, independent mechanism. The `Skill` tool itself
serves a stale `.claude/commands/*.md` when invoked interactively, for the
identical reason (the main checkout is not current). Running `/nightly-ga4` by
hand — session started in the main checkout, then switched to a worktree
before invoking the skill — returned a version of `nightly-ga4.md` missing all
of PR #455 (the standing-summary-script step, required TRAFFIC/EVENTS/UTM
sections, the coverage ledger), even though the file freshly checked out in
the worktree had all of it. The `Skill` tool appears to resolve command
content from wherever the session's skills were indexed at start (the main
checkout) rather than from the current working directory, so entering a
worktree does not fix this the way it fixes the data-pull path. That pass also
flagged a third instance of the same shape — the 03:00 `SparkDate Budget
Ladder` task, which `cd`s into the main checkout with no pull — as "not yet
observed to have caused a wrong result." That last clause is now superseded.

**Incident 22's root cause finally cost something, in the direction the
previous pass had not observed.** That pass flagged the 03:00 `SparkDate
Budget Ladder` task as a third instance of the stale-main-checkout shape, and
said explicitly it was "not yet observed to have caused a wrong result." It
had, the day before — and in the worst form: a stale checkout **undoing a live
write**. A session had migrated the campaign registry to v2 on 09-08 from a
worktree and hand-run the ladder to set `Loxleys | Sales` from $9.00 to $5.11.
The main checkout still held the legacy registry, so at 03:00 the next morning
its ladder put the budget back:

```
2026-09-09T07:00:09.197Z  2026-09-09  account $29.00  LX:convert $5.11->$9.00  [1 changed, 0 failed, 1 ungoverned]
2026-09-10T07:00:04.157Z  2026-09-10  account $29.00  no change                [0 changed, 0 failed, 1 ungoverned]
2026-09-11T07:00:18.970Z  2026-09-11  account $28.51  LX:build  $9.00->$5.11   [1 changed, 0 failed, 0 ungoverned]
```

The v2 rate only stuck on 09-11, after the checkout was pulled on 09-10. Net:
**five budget edits in six days on a purchase-optimised campaign**, each one a
"Pending Process" status flip, while a `HANDOFF.md` entry spent a day asking
whether the ladder was applying at all and listing three guesses, all wrong.
The countermeasure defeated is still none — the previous pass recorded this as
"Not fixed," and it remains so. Note the log was saying `[1 ungoverned]` on
both 09-09 and 09-10: it knew about a campaign it had never heard of, said so
twice, and nobody read it. **Anchor:**
`Night Tasks/logs/budget-ladder.log`; Meta's `/activities` change log;
memories `nightly-pulls-from-stale-main-checkout`,
`meta-activities-endpoint-is-the-change-log`.

**Incident 10 did not recur, and the previous pass was wrong about why.**
Every night from 2026-09-04 through 2026-09-11 produced a nightly log — eight
consecutive, against 8 missing out of 23 before. The cause is not luck and not
a countermeasure this project built: Taylor set `StartWhenAvailable` and
`WakeToRun` to `True` in an elevated shell on 09-04, and the previous pass
reported them as still `False` three days later. That is incident 40.
`RunOnlyIfNetworkAvailable` is genuinely still `False`, so the remaining hole —
a run that fires after boot but before the network is up, which is what broke
both network steps fourteen minutes after a boot on 08-29 — is open. One
setting, same elevated shell.

**A note on this document's own sweep, for whoever runs it next.** The
instruction to "grep the run logs for `ERROR`, `WARN`, `SKIP`" returns zero
matches on every nightly log regardless of content: `<date>.log` files are
**UTF-16LE**, and a plain grep silently matches nothing in them. Pipe through
`iconv -f UTF-16LE -t UTF-8` first. `budget-ladder.log` in the same folder is
UTF-8 and greps normally, so a sweep that spot-checks one file can conclude
the encoding is fine. There are also **three** log families in that folder,
not the two the instructions describe — `<date>.log`, `review-<date>.log`, and
the undated `budget-ladder.log`, which is where incident 22's recurrence is
recorded and which no previous pass has cited.


## What I did not verify

Carried forward from edition 2, still true:

- **The T-14 sales curve against PR #431's "the sales stopped nine days
  ago."** A caution about reading a drought too early, not a documented wrong
  conclusion on its own — though incident 29 shows the same underlying report
  did contain a wrong conclusion, just not this specific one.
- **The `relationship_statuses: [1]` targeting filter**, which removes 80% of
  reachable audience on Marion Court but not Loxleys. Real and significant,
  but an ads-configuration fact more than a documented agent error. Judgment
  call, deliberately left to a human.
- **Automatic placements putting 86% of Marion Court spend into Instagram
  Stories.** Same reasoning — a platform default doing what defaults do.

New this pass, examined and left out of the numbered catalogue for this
edition rather than fully verified — flagged so a future pass does not have
to rediscover them from nothing:

- **A guessed JSON field name printed 0 and read as an emergency, twice in
  one session** (memory `ads-review-json-field-names`, 2026-09-04). Plausible
  fit for this catalogue; not run down to a specific anchor and cost within
  this pass's budget.
- **A repeated question already answered same-day in `HANDOFF.md`** (memory
  `always-check-memory-before-an-nth-ask`, 2026-09-06) and **a broken
  LancasterOnline link live for an unknown period before a 09-06 fix**
  (memory `lancasteronline-is-the-biggest-free-channel`). Both real; neither
  clearly has "an agent operating a live system" as the causal chain the bar
  requires, as opposed to an ordinary content or process gap.
New in the 2026-09-12 pass, looked at and deliberately left out:

- **Whether the two TikTok frames are still serving stale art right now.**
  Incident 43's mechanism is verified directly — the seven-day
  `Cache-Control` header is confirmed live, and the `?cb=` bypass follows from
  it. The specific observation that `LX-17_4of5_tt` and `LX-19_2of3_tt` were
  still serving $24.99 at `age=4068` is cited from the session that read the
  pixels, not re-derived here. A header read at 2026-09-12 shows
  `age=5415`, `last-modified: 12 Sep 02:51 GMT`, which is consistent with the
  corrected file but does not prove the price digits.
- **A dashboard's metrics measuring something other than their labels**
  (#499, #500, #503, #504, 2026-09-08/09 — "Three revenue KPIs measured
  something other than what they said", "Ticket Velocity judged a curve
  against a straight line", "Two ticket counts, neither of which said what it
  covered"). Four commits whose subjects each describe a real defect an agent
  shipped into the operator's dashboard. Left out as a group because
  separating "an ordinary product bug written by an agent" from "an agent
  failure" needs a closer read of each than this pass had room for, and
  padding four thin entries would cost more than omitting four real ones.
- **A misdiagnosed TikTok publishing blocker** (#548, "the TikTok blocker was
  never the domain, it was the audit"; #540, "two of my own claims were
  wrong"). Clearly in scope by shape. Not anchored to a specific cost here,
  and the credential half of it (`tiktok-sandbox-has-its-own-key`) turns on an
  untracked document in `Downloads/` that this pass could not cite.
- **A report that inferred the 2-for-1 ad set was unnecessary from take-up
  data** (PR #484, 2026-09-08) — circular, because every 2-for-1 ad ever run
  sat in a women-locked set, so the self-selection is an artifact of the
  targeting. Closed for that reason at the time. A genuine class-A fit;
  omitted only because the correction is already fully recorded in
  `two-for-one-is-female-ads-only` and nothing here would add to it.
- **A confident wrong answer about whether files differed** (memory
  `file-comparison-lies-on-this-machine`, 2026-09-07/08). `core.autocrlf` is
  `true` with no `.gitattributes`, so a raw `diff` reports every line of an
  identical file as changed; four untracked files were read that way and
  reported as holding work "that matches no commit anywhere," when they were
  byte-for-byte copies of files already on `main`. Both traps in that note
  fail toward "different/missing" — the alarming direction. Left out as a
  near-duplicate of the method already recorded, but it is the best single
  illustration in the catalogue of a tool lying confidently in one direction.

Standing, carried forward and updated:

- **One project, one operator, five months now.** These are incidents from a
  single small business, not a survey. The frequencies are not a base rate
  for anything.
- **Selection bias runs in the obvious direction, and each pass makes it
  worse, not better.** Twenty-three of September's 31 incidents were found by
  digging for something else. A catalogue built primarily out of `LATER`
  catches is, by definition, missing whatever nobody has dug into yet — and
  this pass covered five days that happened to contain more than eighty
  commits, so its density reflects how hard the repo was worked, not how badly
  it went.
- **This pass's own sweep found one of its incidents by re-reading a claim the
  previous pass made.** Nobody has done that for the other forty-seven. The
  error rate of this document against the live systems it describes has been
  measured exactly once, and it was not zero.
- **"Cost" means what was lost or nearly lost**, from logs, commits and API
  reads taken at the time. Where an incident was caught before costing
  anything — 13, 21, 31, 32, 33, 34, 40, 42, 43 — that is stated rather than
  counted as a loss. Where incidents share one cost — 19 and 20; 41 and 47 —
  it is counted once.
- **Names, account identifiers and customer records are omitted throughout.**
