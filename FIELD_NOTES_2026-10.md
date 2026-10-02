# Twenty-six new incidents, three caught by anything automated, and the reviewer that caught three more ran as a stand-in all twelve times

**Window: 2026-09-12 23:00 EDT through 2026-10-02.** The previous edition,
`reports/FIELD_NOTES_2026-09.md`, closed at incident 48 with its last commit
(`73573165`) at 2026-09-12 23:34 EDT, so that is the watermark. This edition
continues the numbering: **49–74**. It is filed under October because it is
written in October and the September file is already 1,020 lines. Twenty-two of
its twenty-six incidents were found in September, two (54, 70) on 10-01/10-02,
and two (51, 69) by this pass itself. Where an incident began before the
watermark and was *found* inside the window, it is dated by both and counted
here, once.

The window held 255 commits on `origin/main` and 20 unattended nightly runs.
Six sources were swept; every anchor below was re-opened against git, the run
logs, GitHub, or the working tree rather than copied from a memory note. Where
one was not, it says so.

**The one-line finding:** the share of failures that anything *automated*
caught is 3 of 26 — **12%**, inside the 9–15% band it has occupied for five
months — and the nearest thing this project has to an automated detector, the
09:00 report reviewer, has been running as an improvised stand-in on twelve of
twelve nights, because the agent it was designed around was never registered.

| | |
|---|---:|
| New incidents, 2026-09-12 → 10-02 | 26 |
| Caught by anything automated (`GATE` + `THREW`) | 3 of 26 |
| `LATER` — found by an unrelated dig | 16 of 26 |
| Caught by the 09:00 *LLM* reviewer, counted as `HUMAN`† | 3 of 26 |
| This document's own errors, measured | 2 (incidents 40, 51) |

---

## EVIDENCE — how they were caught

| Detection | This edition (49–74) | Previous cumulative (48) | Cumulative (74) |
|---|---:|---:|---:|
| `GATE` — a deterministic check refused it | 1 | 1 | 2 |
| `THREW` — an actual error surfaced where someone reads it | 2 | 5 | 7 |
| `HUMAN` — someone distrusted a number or claim, within about a day | 7 | 12 | 19 |
| `OPERATOR` — reported as "data went missing" | 0 | 1 | 1 |
| `LATER` — found by an unrelated dig, or days on | 16 | 29 | 45 |
| **Total** | **26** | **48** | **74** |

**Rules used for the modes**, stated because three of them are judgement calls.
`HUMAN` includes a session acting at the operator's direction that distrusted
the specific claim and checked it within about a day; it does not include a
session that merely stumbled on it later (`LATER`). `THREW` means the error
reached a surface someone reads — a failed run, a red check — not a line in a
log nobody opens (that is why incident 69 is `LATER` though its error printed
on twelve consecutive nights).

**† The 09:00 reviewer is an LLM, not a deterministic gate.** Three incidents
(52, 53, 54) were caught by it before merge. They are counted `HUMAN` above.
**If you count the reviewer as automated, the share is 6 of 26 (23%) for this
edition and 12 of 74 (16%) cumulative.** That is the most generous reading and
it still does not leave the band's upper edge by much. Incident 69 is why the
reading is not safe to rely on.

**Is the automated share moving?** No. Per pass: 11% (2/18) → 9% (2/22) → 15%
(5/34) → 13% (6/48) → **12% (9/74)**. Edition 1's countermeasures were written
in September 2026 on the premise that this ratio would improve. Across 74
incidents and five months it has not left 9–15%. What this window shows instead
is *where* the automated catches come from, and the answer is narrow:

- **The one `GATE`** (61) is a guard written the same night, by the same
  session, that refused that session's own handoff command. It caught its
  author, not a stranger.
- **Both `THREW`s** (66, 73) are loud refusals from an external API — TikTok,
  Meta — to scripts that had been written and merged without ever calling it.
  The failures that dominate the window, 49–60, are wrong conclusions drawn
  from correct data. Nothing throws on a wrong conclusion.
- **Sixteen of 26 were `LATER`.** The incidents that cost the most — a false
  deduplication guarantee still sitting in the code (49), a refactor that has
  silently dropped every walk-in's profile email since 08-29 (67) — were found
  by someone digging into something else, from two days to over a month after
  they started.

**The nightly ledger** (the one section that catches silence by construction).
`<date>.log` exists for every calendar night 09-12 → 10-02: zero missing files.
That is the number the previous edition's method produces, and it is wrong.
Read from inside the files:

| Night | What the log actually says |
|---|---|
| 09-13 | scheduled 02:00 run: `Failed to authenticate: OAuth session expired` at 02:02:39; the hand re-run at 21:46 failed identically |
| **09-15** | **the scheduled run never fired.** The file's first line is `18:49:52 === Nightly run starting ===` — a hand re-run. Windows Update rebooted three times 01:29–01:33; Task Scheduler event 332, nobody logged on (`7e2ddc73`) |
| 09-22, 09-24 | `weekly limit` / `session limit`, 11 and 16 seconds after launch |
| **09-28** | fired at 02:03:37, then three `WARN`s (meta pull `fetch failed`, ga4, eventbrite-ads), `SKIP (analysis): no GA4 pull for today`, and **`=== Run complete ===`** — not `FAILED`. No report, no ladder line, nothing in HANDOFF |
| 09-29 | `weekly limit` after 32 minutes; branch never pushed |

Six of 20 nights produced no unattended report (09-13, 15, 22, 24, 28, 29); I
did not read the other fourteen individually. Two of the six are infrastructure
(auth, quota) with no agent in the chain and are not numbered. 09-15 and 09-28
are in Recurrences, because both defeat a claim the previous edition made.

---

## MECHANISM

### A. The agent was confidently wrong — 13 incidents

**49 — A deduplication guarantee, copied from documentation, written into the
code as fact.** Written 2026-09-11,
contradicted 09-13, 09-17, 09-19 and 09-26. `fc1cbe84` (#531) added a
server-side GA4 `purchase` as the countermeasure for incident 35, and its own
message says: *"Not verified: the browser/server dedupe on shared
transaction_id is from GA4's documentation, not yet observed live."* The same
change wrote it as fact — `lib/ga4-mp.js:71` ("that shared id is what lets GA4
count one transaction when both copies arrive") and `reports/ANALYTICS_METHOD.md:203`
("so GA4 counts one transaction when both arrive"). The file every nightly
report reads first now said the opposite of what GA4 does.

What GA4 did: `81ae1c25` (#634, 09-19) — four own-site orders, four successful
server sends with the buyer's real cookies, GA4 counted them **1, 2, 2, 1**; the
two counted twice were the two where the browser tag *also* worked. "The healthy
case is the case that double-counts." That week GA4 read $194.94 against $97.47
actually kept. `d9836a30` (#700, 09-26): 13 real charges read as 18
transactions, **$554.82 against $397.37 taken, 39.6% too much** — and says
"GA4 does deduplicate these, late and inconsistently." Nine minutes later
`c934f737` (#717, 12:28:57) says *"There is no deduplication at all,"* that the
browser tag has fired on only **3 of 10** real orders since 09-16, so the server
copy is now the primary record of own-site money, and that the webhook handles
`charge.refunded` and tells GA4 nothing (two refunds never subtracted). Two
commits nine minutes apart, one repo, opposite mechanisms.

**Cost:** own-site GA4 revenue overstated by up to 2× across at least five
nightly reports, and credited to the wrong ads (see 62). The countermeasure for
the *under*-count introduced an *over*-count; the previous edition counted it
"built, not as working." It turned out to be working, and wrong. **Still on `origin/main` as of
2026-10-02:** `lib/ga4-mp.js:71` and `ANALYTICS_METHOD.md:203` both carry the
false sentence (read today). **Anchor:** `fc1cbe84`, `81ae1c25`, `d9836a30`,
`c934f737`. **Detection:** `LATER` — found by nightlies checking whether the fix
worked, four times, each finding a different slice.

---

**50 — The nightly's standing instruction told every report to add a link
scanner to the humans, and Taylor was asked what a placement that never existed
cost.** 2026-09-14, found 09-15. `scripts/ga4-nightly-summary.js` §4d told every
report to treat the "Lancaster | Master List / email" row and its plaintext twin
as one: "real totals are the sum," 136–138 sessions, a dead channel. 129 of those
sessions, all on 2026-09-03, were a machine — **zero `view_item`** — against a
9-session human twin that fired 8 of 9 (`70c2687c`). The 09-14 report (PR #580)
then asked the operator *"what did the Fig Lancaster email placement cost, and
is it being repeated?"* **There was no placement.** The rows are
LancasterOnline's Evvnt newsletter; one Marion Court submission had been
filed carrying a Fig short link by mistake, and the nightly "re-derived it from
utm_content and got it wrong for the third time" (same commit).

**Cost:** a false dead-channel verdict and a false question put to the operator
by name; the weekly traffic comparison read −66% instead of −57% once the scanner
left the prior bucket (`81ae1c25`). **Fixed in code, not prose** —
`KNOWN_OBFUSCATED_CHANNELS` names the sender once and §4d now reads `view_item`
per row — which is the difference from incident 37. **Anchor:** `70c2687c`
(2026-09-15 19:40 EDT, #586); `reports/GA4_ANALYSIS_2026-09-14.md:98`.
**Detection:** `LATER`.

---

**51 — This document's own error, the second: a link scanner cited as the
biggest free channel.** Found by this pass. Incident 38 in the previous edition
says the LancasterOnline newsletter "produced **136 sessions in a single day**,
larger than any other free surface on record except Eventbrite's whole-window
total." **129 of the 136 were the scanner in incident 50.** The human reach was
9 sessions. `8b28b0c4` (#646, 09-20): *"'biggest free channel' describes one
day, not a rate"* — daily sessions are 09-03 (129 scanner + 6 humans), then 2,
then 1, then nothing. Incident 38's actual finding — the registry recorded the
surface as unworked while it was live — stands. Its cost line does not, and was
never carried back.

**Cost:** a wrong cost figure for a catalogued incident, in the document whose
only claim to value is that its numbers are right. This is the second time that
number has been measured against the live systems (the first was incident 40)
and the second time it has been wrong — and this one ran in the *flattering*
direction: it made a channel look bigger. **Not edited in place:** the previous
edition is closed and this edition changes only its own file; the correction is
here. **Anchor:** `70c2687c`, `8b28b0c4`; `reports/FIELD_NOTES_2026-09.md`
lines 369–390. **Detection:** `LATER`.

---

**52 — The 09-27 nightly: five conclusions that did not survive re-testing, and
$125.94 of Meta spend that was not in the sum.** 2026-09-27, corrected 09-28
01:16 before merge. The report said Meta spent **$568.90** over 08-22 → 09-26;
the Insights API says **$694.84** — "the daily solve lost $125.94, and the pull
chain is missing 08-26, 08-28 and 09-02" (`047b7a5c`). Four more fell with it:
`events / (not set)` was read as a hand-typed listing tag and a "listing audit"
proposed (it is our own page tag — incident 55); a webview test purchase was
counted as a customer order; a refunded duplicate checkout was counted as a lost
sale; and the free-versus-paid-Meta headline moved to **9 orders against 6**,
$115.81 per order, "an upper bound." The Cold ad set was "already locked to
women 70 minutes before the pull," so "no new lever" was false when written.

**Cost:** caught before merge, so no ask was sent. Meta spend understated 18%,
and the report's headline comparison led with a figure that was wrong in the
direction that flatters paid. **Anchor:** `047b7a5c` (2026-09-28 01:16:08 EDT);
merge `da465aab`; the CORRECTIONS block at the top of
`reports/GA4_ANALYSIS_2026-09-27.md`. **Detection:** `HUMAN`† — the PR review
re-ran the report against Firestore, the GA4 Data API and Meta Insights.

---

**53 — A defect fixed at 00:40 ran as "the most valuable open finding" in three
consecutive nightly reports.** 2026-09-16 → 09-18. The guest double-charge
(incident 62) was fixed by `454a2569` at 00:40 on 09-16 — an ancestor of PR
#597's branch, cut that night. The 09-16 review log: both reports "carry as open
a finding that merged overnight." 09-17: the report "carries the guest-checkout
idempotency defect at `api/purchase-ticket.js:750` as *the most valuable open
finding in this series* … That defect was fixed on 2026-09-16." 09-18: the same
claim again, "**closed** on 2026-09-16 by `454a2569` (#589) and `e0898907`
(#598)." Why the reports carried it forward is not recorded.

**Cost:** three nights of a payment-path finding carried as open work; the 09-18
report was also corrected "after review, at Taylor's request," because all three
of its asks to the operator were wrong or stale. No money. **Anchor:**
`logs/review-2026-09-16.log`, `-17.log`, `-18.log` (UTF-16); `454a2569`.
**Detection:** `HUMAN`† — the 09:00 reviewer, three times, each time one night
late.

---

**54 — The 09-30 nightly said the scheduler did not fire (it did) and built an
ask on a PR that shipped no code.** 2026-09-30, found 10-01. The report says
*"2026-09-28 — the 02:00 task did not fire. The run started at 08:35 … the gap
is the launcher's, not GA4's."* `logs/2026-09-28.log` line 1 is
`2026-09-28 02:03:37 === Nightly run starting ===`; the run fired, three pulls
threw, and it ran a 6½-hour night (see the ledger). Separately it says *"PR #760
shipped 2026-09-29 22:25 ET specifically to keep the plus-one mirroring the
buyer, so this is intended behaviour,"* and calls the mirror structural. `8ba9d63e`
(#760) touches `HANDOFF.md` and one report — **no code**. `api/purchase-ticket.js:1299`
writes `gender: plusOne.gender`, whatever the buyer typed.

**Cost:** a mis-diagnosis (scheduler rather than network) and a mis-scoped ask:
calling the mirror structural foreclosed the cheap lever — "bring a woman, she's
free" copy — for an event 14 of whose 15 paid seats were men. The PR (#766) is
still open. **Anchor:** `efd1b097:reports/GA4_ANALYSIS_2026-09-30.md` lines 26
and 121; `8ba9d63e` (`--stat`: two files); the automated review comment on #766;
`logs/2026-09-28.log`. **Detection:** `HUMAN`† — the 09:00 reviewer, 10-01.

---

**55 — Our own page tag was a reserved GA4 parameter, and five weeks of nightly
reports explained the rows it created.** Wrong since at least 2026-08-20; found
2026-09-28. `lp`, `event`, `events` and `matches` sent `{ source: '<page>' }` on
20 `gtag` calls. GA4 treats an event parameter named `source` as session
attribution, so direct visits were relabelled **`lp / (not set)` (170 sessions),
`matches / (not set)` (42), `events / (not set)` (20), `event / (not set)` (3)**
— **235 sessions, none carrying a UTM** (`9e2bf4b7`). The nightly kept
inventing causes. The 09-23 report: the only `lp` strings in the tree "are the
`source: 'lp'` *event parameter*," and then "the same artifact wearing a
different label." The 09-27 report read `events / (not set)` as a hand-typed
free listing and credited a sale to it (incident 52). The label `lp / (not set)`
or `lp / (none)` now appears **134 times in 27 files under `reports/`.**

**Cost:** 235 sessions mis-sourced for weeks, a listing-audit recommendation
chased and withdrawn, and a channel scoreboard distorted by the amount direct
traffic was hiding. **A real gate was built:** `tests/ga4-reserved-params.test.js`
scans every `gtag` event call under `public/` for the reserved names — 20 hits on
`origin/main` before the rename, none after. **Anchor:** `9e2bf4b7` (2026-09-28
01:27:34 EDT, #748). **Detection:** `LATER`.

---

**56 — The budget ladder was about to hand 65% of the Tellus money to a paused
leg whose audience could never contain the event.** Paused 09-16; found 09-23
23:22 EDT; would have fired 09-29 03:00 unattended. The retargeting ad set's
only pool is a `video_watched` rule last updated **2026-09-04T23:52Z**; every
video behind a live TL2 ad was created **09-19**. "A rule frozen on 09-04 cannot
name a video that did not exist until 09-19." `genders` was unset in a room that
was 9 men to 1 woman. At 03:00 on 09-29 the registry would have moved the leg
from $5.60 to $10.40/day, cut Cold — which produced 8 of the event's 9 checkout
starts — from $6.30 to $3.60, and floored the women-locked 2-for-1 at $2.00.
Its own trigger could not fire while paused: "a paused campaign reads as no
checkouts for the wrong reason."

**Cost:** caught before costing anything. About $111.20 aimed at roughly 1,395
people already at frequency 3.33 was avoided by parking the entry `managed:false`;
the two live edits it implied were left for Taylor. **Anchor:** `f09d8d6f`
(2026-09-23 23:22:58 EDT, #687); `reports/TL2_RETARGETING_GO_LIVE_2026-09-23.md`.
**Detection:** `HUMAN` — Taylor asked "is it time to unpause, and is there an
audience?" and the answer to both was no.

---

**57 — An unreviewed venue blurb said the bar was a former courtroom, and the
listing pipeline copied it to six public surfaces.** Event doc created 09-23,
propagated 09-24, caught 09-25. The Nov 10 event's Firestore `blurb` claimed the
bar sits "inside what used to be an actual courtroom." It does not: the venue is
named for its street (7 Marion Ct), and nothing on its own site, in 43 Yelp
reviews, or in the project's own `venues/` record mentions a courthouse.
`scripts/build-listing-pack.js` reads the blurb from the **live** event doc, so
it went verbatim to Eventbrite (which "published and started selling on it"), a
live Facebook event, Discover Lancaster, LancasterOnline/Evvnt, Visit Lancaster
PA and Fig Lancaster. "Exposed brick," "low lighting" and "a space with history"
were equally unverified. The claim had also reached the repo — the EVENT
section of the cover-art prompt — so the next cover would have asked a designer
for courtroom imagery.

**Cost:** three moderated calendars carry the false paragraph and **cannot be
edited after submission**; Eventbrite sold on it until a hand/API edit on 09-25.
The cover prompt would have asked a designer for courtroom imagery. A rule now
blocks "never assert what a building used to be." **Not verified:** who or what
wrote the original blurb — it lives only in Firestore, and git cannot say. The
agent link is the pipeline that propagated it unreviewed and the session that
re-used it. **Anchor:** `88266057` (2026-09-25, #701), with `81611e88` (#692,
which caught "a Sunday night" on a Tuesday event and a streetless address the
same week). **Detection:** `HUMAN` — Taylor caught it.

---

**58 — A deliberately blank TikTok key was read as a fault, and the operator
was walked through overwriting it.** 2026-09-17 evening. The 09-12 `HANDOFF.md`
entry said TikTok was "deliberately offline … do not re-add secrets"; the key
was empty because Taylor emptied it while reworking the OAuth app. The TL2
organic session "did not read that entry, called the blank key a fault, and
walked Taylor through re-setting `TIKTOK_CLIENT_KEY` and `TIKTOK_CLIENT_SECRET`
to Sandbox" (`da56d467`, in the session's own words). A change spawned the same
evening, "fail the run when TikTok can't auth," "rests on the same wrong
premise" and would have turned every run red during deliberate downtime.

**Cost:** credentials changed mid-rework against a standing instruction; **TL2-03's
TikTok leg lapsed** — `dba2cbd7` records it. No money. **Anchor:** `da56d467`
(2026-09-17 22:02:17 EDT, #618), `dba2cbd7` (#633). **Detection:** `LATER` — the
same session found it while writing its handoff, hours on, not at the time.

---

**59 — A "counter drift" diagnosis that was a comp count, and a recommended
write that would have authorised a 39-seat room.** 2026-09-17. The #609 brief
said the event counter "drifts low on four of six past events" and blamed
Eventbrite import drift; it recommended writing `confirmed` 21→26 and `spots`
→34. A same-day follow-up measured instead: *"the counter equals non-comp
confirmed+pending_3ds tickets EXACTLY on all six events, zero residual. The
reported drift (0,0,-1,-1,-4,-5) is the comp count (0,0,1,1,4,5), row for row.
The Eventbrite-import hypothesis is refuted"* (`14619975`). The real defect was
agent-built: the seat rule excluded comps, so "every seats-left number
overstated the room by exactly the comp count … **Loxleys advertised 9 seats
with 4 real chairs left**" and `api/add-guest.js` authorised that many extra
companions (`eaa827c9`). The backfill script also reported "already correct" on
all six events because it never printed the comp gap.

**Cost:** caught before the recommended write; the seats-left overstatement was
live on a nearly full event five days from the door. **Anchor:** `16a7a98a`
(#609), `14619975` (#610), `eaa827c9` (#613). **Detection:** `HUMAN` — a
follow-up session distrusted the diagnosis and measured it the same day.

---

**60 — A sync's docblock asserted the dashboard would be right; it printed
Eventbrite money as "Facebook / Instagram" and counted $150.54 twice.** From the
09-11 backfill; found 09-14. `scripts/sync-eventbrite-ads-spend.js` said its
`byEvent` field was summed the same way as Meta's, "so per-event cost per ticket
becomes correct with no dashboard change." The dashboard held one per-event map
for every source and used it as Meta's. Effects (`3077fe7b`): an event showed
$54.99 of "Facebook / Instagram" spend that was not Meta's; typed Eventbrite
costs on three events ($150.54) were added on top; $510.08 of unattributed
spend (Meta $436.50, Google $37.92) reached the cost chart but not CAC or net
revenue. **Blended CAC read $17.43; it was about $21.03.** "Nothing had tested"
`eventCosts`, `eventAdSpend`, `renderEventPnl` or `renderChannelPnl` before.

**Cost:** CAC understated by about 17% and the P&L wrong for roughly three days;
no live money moved. The agent-link is the weakest in this section — the
dashboard was agent-written, the sync's docblock is the specific false claim.
**Anchor:** `2d819a01` (#581), `3077fe7b` (#582). **Detection:** `LATER` — a
dashboard structure review, three days on.

---

**61 — A handoff command, refused twice by the guard written the same night.**
2026-09-16. `fb5766e6` (#593) added a guard that **refuses** any executing
Eventbrite sync window unless `no_enroll` is ticked, because the
enroll path emails every attendee it writes — "a wrong pass costs hundreds of
unwanted emails to real customers and is not undoable." The same session's
handoff (`3dfac2d2`) gave `gh workflow run sync-eventbrite.yml -f days=400 -f
execute=true` **without** `-f no_enroll=true`. It was dispatched twice; GitHub
confirms both: run `35056480411` (2026-09-16T04:40:06Z) and `35100022192`
(13:08:29Z), both `workflow_dispatch`, both `failure`. `8b2c8848` (#640) later
corrected the text.

**Cost:** none — stated plainly. A mass email to past attendees was prevented by
a deterministic check, which is why this is the only `GATE` in the window. It
caught the agent that wrote it, and only because the guard and the bad command
were written hours apart. **Anchor:** `fb5766e6`, `3dfac2d2`, `8b2c8848`;
Actions runs above (read live today). **Detection:** `GATE`.

### B. The check passed for a reason unrelated to correctness — 9 incidents

**62 — A reconciliation that matched exactly because two errors summed to the
right shape: a double charge counted as three sales, and a retracted headline
that merged anyway.** Orders 2026-09-12 21:50; claim 09-13; retracted 09-16.
`api/purchase-ticket.js` built its Stripe idempotency key as
`firebaseUid || paymentMethodId`. For guests the browser mints a new
`paymentMethodId` on every submit, so one buyer was charged twice, **49 seconds
apart** (21:50:27 and 21:51:16), refunded by hand at 22:20. The 09-13 nightly
read three PaymentIntents in Vercel's webhook logs, called them "three real
orders," wrote $97.47 as revenue, and concluded: *"there is no evidence any
customer was charged twice"* (`reports/GA4_ANALYSIS_2026-09-13.md`). The 09-14
report credited $97.47 to one ad because it was "exactly three times the $32.49
ticket price." Firestore: one real $32.49 sale, one refunded double charge, one
Measurement-Protocol duplicate of the refund — *"Two independent errors
happened to sum to a shape that reconciled, which is exactly why it convinced"*
(`bbe12247`). A webhook log shows successful payments; it cannot show a double
charge.

**The retraction merged to `main` at 09:34:53 on 09-16. The report it retracted
merged at 22:03:43 on 09-17, titled "One ad is finally creditable with a sale,
and it earned $97.47"** (`89f9b31a`, #580) — 36 hours after the correction, on
main. The structural fix (`454a2569`) also first
shipped a rule nobody had decided — refuse any buyer who already holds a seat,
with an HTTP 200 reading "we haven't charged you" — until Taylor said buying a
friend a ticket on one email is allowed: *"every inference this endpoint drew
from an email address was wrong in this business"* (`e0898907`). That rule was
live about nine hours; whether any real buyer was refused by it is not
established.

**Cost:** a customer double-charged and refunded by hand; the defect stayed live
for every guest checkout for about 3½ days; an ad credited at 3× its sales in
two published reports; a false headline on `main`. **Anchor:** `454a2569`
(2026-09-16 00:40 EDT, #589), `bbe12247` (#587), `04b1fb47` (#571), `89f9b31a`;
Stripe/Firestore not re-read (commit text quoted). **Detection:** `LATER` — the
next night read Firestore instead of Vercel's logs.

---

**63 — A call-to-action "verified" by storing it kept five live ads off
Instagram.** Set 2026-09-16; found 09-19. `935299de` (#603) chose
`GET_EVENT_TICKETS` and says it was "verified against the live account rather
than assumed, by creating a probe creative for each candidate and reading it
back before deleting it." That proves Meta *stores* the value, not that any
placement *serves* it. The Tellus Oct 6 ads ran with **0 Instagram impressions
of 1,166**, while the account's recent sales ads ran 26–60% on Instagram. Meta
raised no error and no `issues_info`; only its own preview says "Your ad won't
run on Instagram because the selected call to action is not supported." The CTA
is immutable, so all five ads needed new creatives. The audit that found it
was prompted by an attendee's email about a video being too fast — a different
complaint.

**Cost:** the first three days of the event's runway were Facebook-only, and five
ads were rebuilt; the audit says an Instagram preview render at creation "would
have caught it on 09-16." Dollars lost to the missing reach are not stated and
not derivable. **Anchor:** `935299de`, `d1a882b6` (2026-09-19 11:05 EDT, #635),
`cb4c17c0` (#637); `reports/TL2_CREATIVE_AUDIT_2026-09-19.md`. **Detection:**
`LATER`.

---

**64 — "ACTIVE" read as delivering: two campaigns spent $0.00 for two days
under a green status.** 2026-09-15 → 09-17. The Tellus Oct 6 Cold and 2-for-1
campaigns were switched on at the campaign level only; their ad sets and ads
stayed paused. `meta-budget-ladder.js --check` printed `ACTIVE/ACTIVE` and
`SKIP already at the seed rate` while spend read **$0.00 with zero impressions**.
Meta calls a campaign ACTIVE whenever its own toggle is on. A `HANDOFF.md` entry
of 09-16 recorded both as "ACTIVE with five ads attached."

**Cost:** two days of a 21-day runway with nothing delivered; about $16 of
planned spend that did not run (derived: $6.00 + $2.00 a day, two days). **Fixed with a gate that did not exist:**
`deliveryBlockers()` now prints `ACTIVE BUT CANNOT SPEND` and exits 1 in every
mode. **Anchor:** `928ce457` (2026-09-17 20:40 EDT, #614); `e4c50060`.
**Detection:** `LATER` — the commit does not say who noticed the $0.00.

---

**65 — An Advantage+ expansion defaulted ON for the "retargeting" ad set, and
nothing read it back — and the first read-back that did was wrong the other
way.** Built 2026-09-16; found 09-26. `scripts/build-paid-campaign.js` never set
`targeting_relaxation_types`, so Meta defaulted `custom_audience` to **1** on
`Tellus AfterDark | Retargeting | broad`: it "has read
`{"lookalike":0,"custom_audience":1}` since 2026-09-16 and can serve women
outside its pools … **No read-back checked it**" (`592e6c3f`). The gender
diagnosis that `/rebalance-event` and the TL2 reports read printed an absent key
as off (`3c4c4695`: "never reads an absent key as off"). The same night the new
lock script "printed **READ-BACK FAILED** and advised an undo" on a *correct*
lock, because it compared raw objects and Meta had written back an all-off
object (`73411576`) — an undo would have unlocked a correct women-only lock.

**Cost:** eleven days of a leg described as women-only retargeting that could
serve beyond its pools; spend outside the pools is unmeasurable, because
insights cannot separate pool members from expansion. It was left on until the
event day because turning it off shrinks the leg to Meta's 1,000-person floor.
**Anchor:** `592e6c3f` (2026-09-27 11:37 EDT, #727), `3c4c4695` (#729),
`73411576` (#725). **Detection:** `LATER` — found reading a Cold-gender
diagnosis, not by any check.

---

**66 — Twenty-six of twenty-six queued TikTok posts could never have posted,
and the first success would have been logged as a failure and re-sent.** Built
before 09-19; first refused 09-19 23:32Z; fixed 09-21 00:34 EDT. A TikTok photo
post's title holds **90 UTF-16 units**; `buildTikTok` sliced it at 150 and
copied it into the description. "All 26 buildable TikTok rows in the queue were
over the limit and none could ever have posted." LX-22 and LX-23 "failed at
init, in all five runs inside their windows, with *The request post info is
empty or incorrect*." Behind it sat a second fault that had never had the chance
to fire: TikTok puts an `error` object on every response and marks success with
`code: 'ok'`; `send()` read the object's *presence* as failure, so "the first
ACCEPTED draft would have been logged FAILED, recorded nowhere, and re-sent by
every later run in the row's 6h window." Reproduced end to end before fixing.

**Cost:** two posts' TikTok legs lost; every queued TikTok post unpostable; the
duplicate-draft path caught before it fired. Later rungs of the same ladder (a
domain-ownership token under the wrong app; a pending-draft cap that crowded out
the day's own post) are in the commits and are not counted separately.
**Anchor:** `a18ecce4` (2026-09-21 00:34:05 EDT, #659), `edcb8e86`. **Detection:**
`THREW` — TikTok refused each post and the Actions run said so.

---

**67 — A "byte-identical" refactor left a caller behind, and the swallowing
`catch` hid the `ReferenceError` for 24 days before anyone looked, and it is
still there at 34.** Shipped 2026-08-29; found
09-22/23 by the whole-repo review; **unfixed on `origin/main` as of 10-02.**
`74acddbc` (#326) moved the enrolment helpers to `lib/enroll.js` — "388 lines
moved unchanged … handleEventbriteEnroll behaviour is byte-identical … 360 tests
pass." `checkinProfileHTML` moved with them (`lib/enroll.js:113`) but is **not
exported** (`module.exports` at `:576` lists two names) and not imported, while
`api/lead-signup.js:520` still calls it. The call throws a `ReferenceError`
inside a `catch` documented as best-effort that logs and moves on, and
`emailSent` is always false with nothing reading it. I re-read all three
locations today.

**Cost:** every walk-in with an incomplete profile, at every event since 08-29,
received no "complete your profile" magic link — the link same-night matching
depends on. **How many were missed was not measured.** The review's own header:
"None of the 38 was going to be caught by any gate this repo currently runs."
**Anchor:** `74acddbc`; `reports/REPO_REVIEW_2026-09-22.md` §1.6; the three
locations above. **Detection:** `LATER` — 24 days on, found by a 28-partition
review.

---

**68 — An Eventbrite field changed meaning on 08-31 and nobody changed our code;
the memory that "settled" it had overturned the correct draft.** Drift began
2026-08-31; found 09-24, 24 days on. The sync reads `costs.base_price`, unchanged
since 08-29. Eventbrite began returning it **net of its own fee** on listings
that absorb fees: "51 rows hold gross, 25 hold net, and no reader can tell them
apart." The admin Channel P&L computes `gross = sum(amount)` then subtracts fees
again, so on the 25 rows the fee comes off twice — revenue and net both
understated by **$88.83** (13.3% of the affected rows), growing with each sale.
Reading each row under its own convention reproduces Eventbrite's Total Sales
line to the cent on two events: $394.86 and $289.90.

The mechanism of the 17-day delay is the finding. A **2026-09-07 draft was
right** — "$21.56 each, Eventbrite's net of its own fees." It was overturned by
adversarial verification on the grounds that "there is no fee-netting code path
to invoke." The verifier read our import code and found no subtraction, which is
true, and concluded wrongly: *"the netting happens upstream, inside Eventbrite's
`base_price`, where reading our code cannot see it."* The memory's stated
evidence was Stripe rows, which test the unit and not the Eventbrite path at all.

**Cost:** five earlier reports needed correction banners; **the repair has not
been run** — `HANDOFF.md:549` reads "0 rows repaired, 25 still holding net,
$88.83 outstanding." **Anchor:** `ef66ed68` (2026-09-24 11:05 EDT, #693);
`reports/EVENTBRITE_AMOUNT_GROSS_OR_NET_2026-09-24.md:127-148`. **Detection:**
`LATER` — found tracing an unrelated 2-for-1 order.

---

**69 — The nightly reviewer's designated agent is not registered; twelve of
twelve reviewing runs say so, and two runs died without a word.** In every
reviewing run from 09-14. `REVIEW_PROMPT.md` step 4 specifies `subagent_type:
pr-reviewer`, whose definition (`.claude/agents/pr-reviewer.md`) is the thing
that makes the review read-only. The harness never registers it. **I read the
review logs: 09-14, 16, 17, 18, 19, 20, 21, 25, 26, 27, 28 and 10-01 — 12 of
12 reviewing runs — each report "the `pr-reviewer` subagent type is not
registered," listing only `claude, Explore, general-purpose, Plan,
statusline-setup`.** The runs improvise with `Explore` or `general-purpose`,
where read-only rests on a prompt instruction. The reviewer also says what it
cannot check: on #766 its "Unverifiable from here" section lists every GA4
session, event and revenue figure, the Firestore roster, Eventbrite Ads spend and
the Meta city tables — "external data, no way to check from source." Its catches
(52–54) were claims it could set against code and logs; a wrong number it cannot
re-derive passes.

Two logs end after `Using Claude CLI: …` with no exit line — **09-23 and 09-30**
(10-02 reads the same shape at the time of writing and may still be running).
Per `gh`, the 09-23 report's PR was merged 09-24 with only the Vercel bot's
comment.

**Cost:** no realised loss. The read-only guarantee `8b897ed3` (08-15, #176,
"Hard-gate the nightly reviewer's tools instead of trusting the prompt") was
built to provide did not hold in any of the twelve runs I read, and a report PR
(#681) merged unreviewed. The prompt audit of 10-02 (`c0befa67`, #787) audited
`pr-reviewer.md` line by line as the subagent `REVIEW_PROMPT.md` step 4 calls
(`PROMPT_AUDIT_2026-10-02.md:48`) and does not mention that the type does not
register. **Anchor:**
`logs/review-2026-09-14.log` … `-10-01.log` (UTF-16; read for this edition);
`reports/PROMPT_AUDIT_2026-10-02.md`. **Detection:** `LATER` — the error printed
every night and reached nobody; this pass is the first reader.

---

**70 — The PII clean-up was recorded complete at 09:14, and a sweep at 10:21
found 18 files that still named buyers.** 2026-10-02. `3981450d` (09:00) and
`dbd26904` (09:14) record the clean-up finished — GitHub Support "removed
`refs/pull/1-559` and ran the GC (verified 10-01)." `7dbcbf45` (10:21) then
says "a sweep of the working tree found **buyers still named in full**": three
Tellus buyers in `HANDOFF.md` and one report, a Loxley's buyer with a personal
Gmail address in a report, eight more beside order numbers, five in the
Eventbrite amount report, "a customer email in HANDOFF, and real names in code
comments and one test fixture." Several were written by agents *after* the 09-04
scrub (#434) and the 09-12 rewrite — including a commit message on 09-24
(`ef66ed68`, incident 68) that lists five buyers by surname, and the Eventbrite
amount report the same day. The rewrite's own verification, per `d8c3b923`,
"checked exact literals only." The 10-02 prompt audit had quoted a first name too
(`7dbcbf45`'s second commit removes it).

**Cost:** named customers, in a singles-event context, were written into tracked
files from 09-24 onward and sat there up to eight days; the clean-up was declared
complete an hour before the sweep that found them. `gh repo view` reports
**PRIVATE**, so public exposure is not established. `7dbcbf45` states "Git
history and refs/pull/* still hold the real names." **Not verified:** whether
anything beyond what that commit lists remains. **Anchor:** `7dbcbf45`
(2026-10-02 10:21:13 EDT, #788), `3981450d`, `dbd26904`. **Detection:** `LATER` —
a working-tree sweep prompted by an aside in the 10-02 prompt audit.

### C. Silent failure — the error was swallowed and read as absence of data — 2 incidents

**71 — The Eventbrite sync counted attendees it could not import as "already
enrolled."** Since the sync was written on 2026-08-29; found 09-24; fixed 09-26.
`scripts/sync-eventbrite.js` filtered with `em && !existing.has(em)` and then
`totalExisting += live.length - fresh.length`, so an attendee with a blank or
duplicate email was reported as enrolled. On a 2-for-1 order the order form
collected only the buyer, so the friend's ticket carried the buyer's own
address, and was invisible to the seat counter, the pre-event emails and the
seating tool. The 09-24 dry run printed "6 attending, 0 new" against 5 tickets
and said nothing about the sixth; the nightly counted it "missing" four nights
running without finding the cause. The sync runs unattended every six hours.

**Cost:** on Tellus Oct 6 one paid Bring-a-Friend seat was a guest nobody could
name or seat. Whether older half-price pairs lost half their revenue in
Firestore is flagged in the commit as **not checked.** **Anchor:** `323b45e2`
(2026-09-26, #720): "…them as 'already enrolled' and said nothing"; `e5e15477`
(#697). **Detection:** `LATER`.

---

**72 — The returning-attendee invite to 120 people left no usable trace, and
every send would have erased the last.** Sent 2026-09-23; found 09-27 building
the Retention tab. The cron stamped each lead with the event and **discarded
Resend's email id**, so every delivery and click reached the webhook "as
'unmatched'" and was dropped; `resend_events` keeps only `type` and
`receivedAt`. The pair `returningInviteEventId`/`returningInviteSentAt` was
flat, so "the next event's first send would have erased most of them." And
`getNextEvent` skips `status:'full'`, so an event closed and reopened "would
have been invited twice."

**Cost:** *"The 2026-09-23 send for Tellus Oct 6 is dark for good"* — whether
120 invitees opened or clicked cannot be recovered, and the `HANDOFF.md` line
"none bought" was unmeasurable when written. **Anchor:** `65630938` (2026-09-27
20:27 EDT, #737), `684b4368` (#741). **Detection:** `LATER`.

### D. Destructive, or unreported, writes that report as fine — 2 incidents

**73 — A `200` from `/adcreatives` that silently discarded the rules, and an
epilogue that said "Every ad above is PAUSED" after a run that created zero ads.**
2026-09-16. `01000d72` (#601) built per-placement video via `asset_feed_spec`,
its docblock saying it "HAS NOT BEEN EXERCISED AGAINST THIS ACCOUNT." On the live
account `POST /adcreatives` returned 200 for four of five ads and **every one
came back with three videos stored and ZERO `asset_customization_rules`** —
"Meta accepted the creative and silently discarded the whole rules array." The
read-back that would have caught it never ran, because `POST /ads` failed first
and the code continued past it. "The worse bug of the two": the epilogue printed
*"Every ad above is PAUSED. Review them in Ads Manager"* after a run that created
**zero** ads, because it was unconditional.

**Cost:** caught before costing money. Five ads planned, none created, four
orphaned creatives deleted, and a re-upload of video ("each guess costs orphaned
creatives and re-uploaded video").
Why Meta dropped the rules "is NOT established." **Anchor:** `8588ea87`
(2026-09-16 17:59:46 EDT, #602). **Detection:** `THREW` — `POST /ads` refused
each pairing with `100/2446485`; nothing but that refusal flagged the discard.

---

**74 — The history rewrite reported success twice and was wrong twice.**
2026-09-12 night, found 09-12/13. (a) `git filter-repo` plus `push --mirror`
updated all 315 branches, "every line *(forced update)*," and the pre-rewrite
commits **were still served by GitHub's API minutes later**, because
`refs/pull/*` are read-only and the rejections print *after* the successes
(`d8c3b923`: "a remediation that reported total success while the exposure it
existed to close stayed open"). (b) The case-sensitive `--replace-text` renamed
a test fixture but not the same address in lowercase two lines below, so
`tests/gender-backfill.test.js` failed and **`main` was red** — and 37 minutes
later a second session diagnosed the same test as one that "has failed since it
was written in #354 on 2026-08-30 and could never have passed" (`9daeeed8`). In
the rewritten history even #354 contains the mismatch (`28ba16d9`,
`tests/gender-backfill.test.js` lines 67 and 70 name two different addresses), so
the second account is an artefact of the rewrite read as an old bug.

**Cost:** pre-rewrite history, including customer names, remained retrievable
from GitHub for about 19 days, until Support removed `refs/pull/1-559` and ran
the GC (`3981450d`, "verified 10-01"); `main`'s CI was red from 09-12 to 09-13.
**Anchor:** `d8c3b923` (2026-09-13 08:42 EDT, #560), `9daeeed8` (#561),
`3981450d`. **Detection:** `HUMAN` — the session tested the exposure itself
(`gh api …/commits/<old-sha>` → 200) instead of trusting the push output.

### E. Concurrency — several agents, one working tree — 0 new incidents

None numbered. One recurrence is recorded below.

---

## DECISION — what changed, recurrences, what's still open

**Recurrences.** A recurrence is the strongest evidence a countermeasure did not
work; each names the one it defeated.

- **Incident 22 — a stale main checkout undoing a live write, 2026-09-16.** The
  09-15 `managed:false` parking of Loxley's retargeting was committed from a
  worktree. At 03:00 the ladder read the main checkout, which had not been
  pulled: `budget-ladder.log:8` — `2026-09-16T07:00:07.220Z … account $29.73
  LX:close $8.00->$3.41 LX:close $2.00->$6.32 [2 changed, 0 failed, 0
  ungoverned]`. The restore (`3e173d60`) ran about 41 hours later (09-17 23:58 UTC), and
  `16a7a98a` records the cost as the broad leg getting $6.32/day to the
  women-locked cell's $3.41, with $15.14 spent reaching 206 men at frequency
  4.38 for zero checkout starts. **Countermeasure defeated: none existed** — the
  previous edition recorded this as "Not fixed," and parking takes effect only
  when the main checkout is pulled, so `--check` from a worktree is not a test of
  what 03:00 will do. A second form of the same delivery problem: `36c0df7c`
  (#611) records the Skill tool delivering `/fix-eventbrite-listings` with its
  price-verification tokens replaced by page text, after the session had told
  Taylor the file was corrupted ("it was not").
- **Incident 10 — a nightly that did not run (09-15, 09-28), defeating this
  document's claim that it had stopped.** The previous edition reported "Incident
  10 did not recur" and credited `StartWhenAvailable` and `WakeToRun`. **09-15 is
  the first night in the record since those flags were set that the scheduled run
  did not fire** — a different
  mechanism: nobody logged on after three Windows Update reboots (event 332),
  all three scheduled tasks `LogonType Interactive` (`7e2ddc73`); the flags
  cannot help when nobody is logged on. It left no log and read as a quiet night
  until a hand re-run at 18:49. **09-28** fired and still produced nothing
  (ledger above), with `=== Run complete ===` where `FAILED` belonged. Fix for
  the former needs Taylor's Windows password.
- **Incident 38 — a registry wrong about live surfaces.** `9daeeed8` (09-13):
  Meetup recorded `not_pursued` with "ZERO events on it" when both live events
  were posted there; `5714ea09` (#645, 09-20): "four registry statuses were
  wrong"; the HANDOFF entries on the Nov 10 event said Eventbrite was "still
  Draft" when it was published and selling (`6e71a5b6`, #703). And incident 38's
  own cost line is incident 51.
- **Incident 39 — art carrying a dead price.** `5f627611` (#683, 09-23) on the
  LX-27 recap: "This is the MC-15 situation again." (It also found `[REAL NUMBER]`
  in the caption and links pointing at the event that had just ended; the row was
  fixed before it was approved at 21:30, so the approve gate's placeholder check
  was not what stopped it — I could not tell which.)
- **Incident 06 — a fact on six surfaces, all wrong.** `2d0b678b` (#657, 09-21):
  the welcome email "described the evening backwards, and it carries 76% of
  conversions" — open mixing first, tables second, a duration quoted against a
  standing "state the shape, never a duration" rule, no 1-on-1s or match. Two
  on-site guides carried the same timeline. `audit-facts.js` did not read the
  cron email.
- **Incident 35's shape — a report's headline contradicted within a night or two.**
  Incidents 49, 52, 53, 54 and 62 are all this, in the automated path. The one
  that reached `main` uncorrected is 62.
- **Stranded commits — a class the memory already counts three times
  (#100, #101, #182) and no edition has numbered.** PR #755 squash-merged
  `b4d53039` as its only commit at 2026-09-29T02:39:30Z; `d26c3409` ("Thumbnails
  show the opening scene, not the end card laid over it"), committed 23:26 EDT on
  09-28, exists only on its branch and is not an ancestor of `origin/main`. The
  fix reached `main` about a day later inside #757 (`217b788c`). Not
  numbered: how it was noticed is not recorded, and nothing shipped wrong.

**Built in response, during the window.** Most of these are deterministic checks
that did not exist before. Each is named for the incident that produced it, and
the one-at-a-time pattern the previous edition described is unchanged.

- **A scan for reserved GA4 parameter names** — `tests/ga4-reserved-params.test.js`
  (`9e2bf4b7`), closing 55.
- **`ACTIVE BUT CANNOT SPEND`** — `deliveryBlockers()` exits 1 in every mode
  (`928ce457`), closing 64.
- **A read-back that fails and rolls back** if `custom_audience` is not exactly as
  sent, and `ads:review` exit 3 for any unacknowledged expanding set (`592e6c3f`),
  closing 65.
- **A test that builds every TikTok row in the real queue against both limits**,
  and a runner that asks TikTok for status instead of trusting the init
  (`a18ecce4`), closing 66.
- **Scanner detection in code** — `KNOWN_OBFUSCATED_CHANNELS` and a twin-relative
  `view_item` test (`70c2687c`), closing 50.
- **A structural double-charge fix** — one PaymentIntent per buyer × event ×
  price, create then confirm, 20 cases verified non-vacuous by mutation
  (`454a2569`), closing 62's cause.
- **The wide-window / `no_enroll` guard** (`fb5766e6`) — the window's one `GATE`.
- **Content rule: never assert what a building used to be** (`88266057`), after 57.
- **Named un-importable attendees** (`323b45e2`) and **the money helpers with 14
  tests** (`ef66ed68`), after 71 and 68.
- **`c0befa67`** (#787) fixed this command's own stale instruction — it said the
  previous catalogue ended at 18 and the next number was 19, which would have
  collided with 19–48.

**Still open, as of `origin/main` on 2026-10-02:**

- **49** — the false deduplication sentence at `lib/ga4-mp.js:71` and
  `ANALYTICS_METHOD.md:203`, and the missing refund reversal.
- **67** — `checkinProfileHTML` is still not exported; every incomplete-profile
  walk-in is still getting no link.
- **68** — "0 rows repaired, 25 still holding net."
- **69** — `pr-reviewer` is still unregistered; no one has been told.
- **Unnumbered** — `lint-content-queue.js` still does not check a row's column count
  (`da56d467` lists it as an open follow-up; the 09-16 queue truncation to 49 of
  62 rows is in the commits but was caught by a thrown `ValueError` and is not
  numbered).

---

## What I did not verify

- **No live reads.** No Meta, GA4, Stripe, Firestore, Eventbrite or TikTok call
  was made for this edition. Every dollar figure and count is from a commit
  message, a report, or a log, as quoted. I did re-open: the run logs (UTF-16),
  the two guard-refused Actions runs, PR #755 and #766, the working-tree code for
  67 and 49, and `gh repo view`.
- **Whether the 14 other nights ran clean.** I read six nights' logs and the ones
  needed for the ledger; I did not read the other fourteen individually. One
  sweep counted at least nine nightly reports carrying their own retraction or
  correction (09-14, 16, 17, 18, 23, 26, 27, 30, 10-01). That is a count, not an
  audit.
- **The author of the courtroom blurb** (57). It lives only in Firestore.
- **Incident 69's end state on 10-02.** The review that started at 09:00 may
  still have been running when I read it.
- **Whether the reviewer is Taylor or the automated step** for 52. The commit says
  "after the PR review re-ran this report," and I did not trace who commissioned
  the re-run. It is classed `HUMAN`† either way.
- **How many walk-ins (67) or older half-price pairs (71) were affected.** The
  commits say it was not measured.
- **This repository's own README and essay.** The 09-12/09-17 HANDOFF pre-list
  (`d8c3b923`, `670f263c`) records five more incidents about them — a README that
  repeated incident 40's error, an essay figure that matched nothing, a render
  check that read an error as zero — that I could not verify from the private
  repository the notes are written in. They are **not numbered**.

Looked at and left out, so a later pass need not rediscover them:

- **A free +1 offer typed into a lead's reply was refused by the auto-mode
  permission layer on 10-01** — memory `offers-in-customer-messages-need-taylors-word`.
  That would be a second `GATE`. It has no anchor outside the memory note, so it
  is not counted; leaving it out is the conservative direction for this table.
- **Partner-discovery tooling** (`29cd5cf9` #691, `5688bb03` #706, `80556c17`
  #707): a worklist that lost four real businesses while the suite stayed green,
  and a "deliberately strict" site matcher that accepted a retirement community.
  Real, anchored, thin cost (nothing sent). Left out as near-duplicates of 66's
  mechanism.
- **A published recap number of 29 for Marion Court** labelled "counted check-ins"
  (`550dba64`), which `5f627611` says was the registration count against 20
  check-ins. The queue note says Taylor confirmed the figure; what he was
  confirming is unclear, so no agent error is claimed.
- **Gmail connector link rewriting, Facebook composing as the Page, an Instagram
  DM refused but rendered as sent** — anchored only to memory notes.
- **Quota and OAuth nights (09-13, 09-22, 09-24, 09-29)** — platform auth and
  capacity; no agent in the chain.

Standing, carried forward and updated:

- **One project, one operator, five months.** These are incidents from a single
  small business, not a survey. The frequencies are not a base rate for anything.
- **Selection bias runs in the obvious direction.** Sixteen of this edition's 26
  were found by digging for something else, and this window held 255 commits, so
  its density reflects how hard the repo was worked as much as how badly it went.
  A catalogue of noticed failures is missing whatever nobody has dug into yet.
- **This document's error rate against the live systems has now been measured
  twice, and was not zero either time** (incidents 40 and 51). Nobody has re-read
  the other seventy-two.
- **"Cost" means what was lost or nearly lost,** from logs, commits and reads
  taken at the time. Where it was caught before costing anything — 56, 61, 73 —
  that is stated, not counted as a loss. Where incidents share a cost — 49 and 62;
  52 and 55; 68 and 70 — it is counted once.
