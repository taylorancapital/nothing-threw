# Ten new incidents, one caught by anything automated, and two review findings that said "fix before merging" and were merged over in an eighteen-second batch

**Window: 2026-10-02 10:51 EDT through 2026-10-12.** The previous edition,
`FIELD_NOTES_2026-10.md`, closed at incident 74 with its last commit at
2026-10-02 10:51 EDT, so that is the watermark. This edition continues the
numbering: **75–84**. It is a ten-day window, filed under its end date because
the October file already exists. Where an incident began before the watermark
and was *found* inside it, both dates are given and it is counted here, once.

The window held 94 commits on `origin/main`, 11 unattended nightly runs and 11
report-review runs. Six sources were swept: 29 memory files written since the
watermark, `CLAUDE.md` (one change, no failure in it), the reports and commit
messages, the run logs (decoded from UTF-16), git history, and `HANDOFF.md`'s
unowned threads. Every anchor below was re-opened against git, GitHub, the logs
or the working tree on 2026-10-12. Where one rests on something else, the entry
says so.

**The one-line finding:** 1 of 10 was caught by anything automated, and that one
came from an uptime monitor reporting that checkout was down. The other new fact
is narrower and worse. The 09:00 reviewer wrote "needs a fix before merging" on
two pull requests, one for customers' personal email addresses and one for a
compiled venue contact list. Both were then merged in a batch of seven that took
eighteen seconds. A check worked and was ignored, which this catalogue has no
detection mode for.

| | |
|---|---:|
| New incidents, 2026-10-02 → 10-12 | 10 |
| Caught by anything automated (`GATE` + `THREW`) | 1 of 10 |
| Recurrences of catalogued incidents | 7 |
| Review findings marked "fix before merging" that merged anyway | 2 of 2 |
| This document's own errors, measured | 3 (incidents 40, 51, 75) |

---

## EVIDENCE — how they were caught

| Detection | This edition (75–84) | Previous cumulative (74) | Cumulative (84) |
|---|---:|---:|---:|
| `GATE` — a deterministic check refused it | 0 | 2 | 2 |
| `THREW` — an actual error surfaced where someone reads it | 1 | 7 | 8 |
| `HUMAN` — someone distrusted a number or claim, within about a day | 7 | 19 | 26 |
| `OPERATOR` — reported as "data went missing" | 0 | 1 | 1 |
| `LATER` — found by an unrelated dig, or days on | 2 | 45 | 47 |
| **Total** | **10** | **74** | **84** |

**Rules for the modes,** the same as the previous edition. `HUMAN` includes a
session that distrusted a specific claim and checked it within about a day.
`THREW` means the error reached a surface someone reads. **†** marks a catch made
by an LLM step, either the 09:00 reviewer or the nightly analysis. Those count
as `HUMAN`, never as automated. Two of this edition's seven `HUMAN`s are †
(80, 84).

**The previous edition's counts stand as published here.** The 2026-10-02 audit
of that edition (incident 75) verified that three of its chips should move: 58
from `LATER` to `HUMAN`, 64 from `LATER` to `HUMAN`†, and 52 from `HUMAN`† to
plain `HUMAN`. They have not been applied to `FIELD_NOTES_2026-10.md`, so they
are not applied in the table above. With them, cumulative `HUMAN` is 28 and
`LATER` 45. Neither changes the numerator. Not every incident before 10-02 has
had that kind of re-check, so the table measures what was noticed and how. It
does not measure what was true.

**Is the automated share moving?** No. Per pass: 11% (2/18) → 9% (2/22) → 15%
(5/34) → 13% (6/48) → 12% (9/74) → **12% (10/84)**. It has stayed between 9% and
15% for six passes and five and a half months. This pass alone is 1 of 10.

The mix is different. Only 2 of 10 were `LATER`, against 16 of 26 last time,
and five were caught the same day. Do not read that as improvement. A ten-day
window cannot contain anything found weeks later, so the slow finds from this
window have not been dug up yet. The last edition's slow finds took two days to
over a month to surface. The one comparison that holds across windows is the
automated share, and it has not moved.

**What the one automated catch was.** UptimeRobot's "SparkDate Health is down"
reported the symptom, an empty Firestore quota. The cause was a refresh interval
an agent had set in July, describing it as the fix for the same outage (76).
The monitor found that checkout was down. It said nothing about why.

**The nightly ledger.** `<date>.log` exists for every night from 10-02 to 10-12,
eleven of eleven. Ten ran to a report. **10-12** ended `Failed to authenticate:
OAuth session expired` at 02:02:49, then `=== Run complete (analysis FAILED) ===`,
and the 10:12 review died the same way. That is infrastructure with no agent in
the chain, so it is not numbered. The **review** logs:

| Run | What the log says |
|---|---|
| 10-02 09:00 | two lines, then nothing: no exit line (incident 69's shape, now confirmed rather than "may still be running") |
| 10-03 | `pr-reviewer` not found; the run **chose to skip** rather than substitute, so two report PRs went unreviewed that day |
| 10-04, 05, 06, 08, 09, 10, 11 | `pr-reviewer` not found every time; each substitutes `Explore` |
| 10-07 | **the 09:00 run never fired.** The file's only start line is 22:12, and that run failed on a 401 |
| 10-12 | OAuth expired |

---

## MECHANISM

### A. The agent was confidently wrong — 5 incidents

**75 — This document's own error, the third: the fifth pass went public with a
failing claim in 25 of its 26 entries.** Published 2026-10-02 13:56 EDT; audited
the same evening. The 49–74 edition stated that "every anchor below was
re-opened." The anchors were. The sentences around them were not. The audit
(`8ecae871`, #792, 19:25 EDT) re-checked every claim in 49–74 and found at least
one failing claim in 25 of the 26. These were verified by hand:

- **61, the edition's only `GATE`, prevented nothing.** The 400-day dry run four
  minutes before the refused execute printed `new 0 · already enrolled 76`, so
  the refused run would have emailed nobody. The guard and the refusal were nine
  minutes apart, not "hours apart."
- **52 was not caught by the 09:00 reviewer.** Its comment listed the Meta daily
  solve as unverifiable; a hand session corrected it.
- **58 and 64 carry the wrong chip.** See the table note above.
- **59's headline inverts its source.** The 39-seat figure was the brief's own
  warning, not its recommendation.
- **68's "the repair has not been run" was false.** It ran on 09-24 (run
  36022736611), and that run is incident 83.

**Cost:** the public catalogue has carried these claims since 10-02, and as of
10-12 its October file is unchanged (last commit there: 10-02 18:19 UTC). The
headline held: 9 of 74 was right. Its supporting line, "forty-five were found by
accident," should read 43. This is the third time this document's numbers have
been measured against the live systems (after 40 and 51), and the third time
they were wrong. **Anchor:** `8ecae871` (#792);
`reports/NOTHING_THREW_AUDIT_2026-10-02.md`; `nothing-threw` merge `02f3826`.
**Detection:** `HUMAN` — an audit commissioned the same day.

---

**76 — A refresh interval chosen in July as "the fix" for a Firestore quota
outage caused the same outage in October, and checkout went down with it.**
Interval set 2026-07-06; outage 2026-10-04 23:20 EDT. In July a session
investigated `RESOURCE_EXHAUSTED` errors and found the admin dashboard
re-reading whole collections every 30 seconds: "plausibly generating hundreds
of thousands of document reads on its own." It changed the interval to five
minutes, "10x reduction, zero functional change," and called the plan upgrade
"a billing decision, not code" (`5e624b3c`, #70). The comment it left in
`public/admin.html` contains the arithmetic that rules its own fix out. At "500–1000+
document reads per refresh," a five-minute tick in one open tab is 144K–288K
reads a day, against a quota of 50K a day. Nobody multiplied it out.

On 10-04 the project read **476K documents in 24 hours** on the 50K/day plan,
"~20K reads an hour around the clock, overnight included." The fit is "one or two
tabs left open all day" (`d8652d1e`). Firestore began answering
`RESOURCE_EXHAUSTED` at about 23:20 EDT. Checkout, the event pages, the hostess
roster and `/api/eventbrite-live` all read Firestore, so all of them went down.
The health check made the cause invisible: the Admin SDK retries quota errors
over gRPC, so `/api/health` reported "firestore timed out after 5000ms" for an
hour while the REST API was answering `429 Quota exceeded` (`85e7f21c`).

**Cost:** checkout and event pages down from about 23:20 EDT until after the
project moved to a paid plan at about 00:05, plus a redeploy to clear instances
stuck in retry. That was thirty-six hours before the largest event the business
has run, and on what was then its biggest sales day (10 seats, #808). Whether any
sale was lost in that hour was not measured. The project now pays for Firestore.
**Anchor:** `5e624b3c` (2026-07-06, #70); `d8652d1e` (#805), `85e7f21c` (#806),
`6504acb2` (#807). **Detection:** `THREW` — UptimeRobot's "SparkDate Health is
down" at 03:22 UTC.

---

**77 — A duplicate-account merge re-imported the buyer ninety minutes later, and
his duplicate tickets read as the day's sales.** 2026-10-10. A repeat guest had
two logins on two Apple addresses. A session merged them, kept the newer address,
and deleted the login on the address his Eventbrite orders were placed under. It
ran in auto mode, Auth delete included. The Eventbrite sync's pre-check matched
attendees by **exact email**, so it no longer recognised him. "enrollEventbriteOne
created a new … account with four second tickets for seats he already held. They
showed on the dashboard as the day's sales" (`c11f744e`). The merge's own warning
that three ticket docs pointed at one seat was first read as duplicates. The same
day that reading was corrected to one import duplicate plus one real second
purchase.

**Cost:** four phantom tickets, a phantom account and lead, and event counters
each one high. All were cleaned by hand the same day. The sales figure on the
dashboard was wrong for some hours. **Anchor:** `c11f744e` (2026-10-10 16:39 EDT,
#844); memory `retiring-a-duplicate-login`. **Detection:** `HUMAN` — found and
fixed the same afternoon. The commit does not say who noticed.

---

**78 — A customer email said "see you at 6:30" five hours after doors.**
2026-10-06. A buyer wrote at 16:14 EDT about a sign-in problem. The reply,
written as though it were still that afternoon, said she did not need a profile
"to come tonight … See you at 6:30!" Gmail Sent records it at **23:45:34 EDT**.
The event had opened at 18:30. A correction ("I sent that one late") followed at
23:46:03. The session had no clock. The conversation carried the time the
customer wrote, not the current time.

**Cost:** one customer told to arrive at a door that had closed hours earlier,
corrected 29 seconds later. No money. **Anchor:** Gmail Sent, read 2026-10-12:
two messages in one thread at 03:45:34Z and 03:46:03Z on 10-07; memory
`check-the-clock-before-time-relative-copy`. **Detection:** `HUMAN` — the
session read the send time back.

---

**79 — The door check-in overwrote every ticket holder's registration, and a
"47% of women walk in" figure was built on the result.** Flagged 2026-09-22;
cost realised 10-06; fixed 10-08. `api/lead-signup.js`'s check-in wrote
`ticketId: null, paymentIntentId: null, source: 'checkin'`, plus whatever name and
gender the door form held, over the buyer's own registration with `merge: true`.
The function's header says "just mark checked in, never duplicate." The
whole-repo review found it on 09-22 (`REPO_REVIEW_2026-09-22.md` §1.2, at
`api/lead-signup.js:450`). It was not fixed, the same fate as incident 67 from the
same review. On 10-06 it rewrote **21 of 27** check-ins. Across history, **104
of 139** rows reading `source: 'checkin'` were tickets it had relabelled; 35
were made at the door.

The figure built on it: the 09-10 measurement for the "seat friends together"
feature read 47% of women's registrations as door walk-ins. Re-measured on 10-08
it is **13%** (`reports/DOOR_CHECKIN_GAP_2026-10-08.md:78-80`). The same write
blanked the 2-for-1 order key the feature uses to keep a pair together. On the
night, "the hostess stopped using /host at the door and built the first round
from a handwritten list" (`a17d7c7e`).

**Cost:** 89 registrations with their purchase links wiped (restored 10-08 by
`891ad0a0`). A walk-in share wrong by a factor of three fed a shipped feature. The
host tool was abandoned on the night. **Anchor:** `REPO_REVIEW_2026-09-22.md`
§1.2; `a17d7c7e` (#822), `891ad0a0` (#824), `daa9e024` (#826). **Detection:**
`LATER` — a whole-repo review, and then sixteen days in which nobody acted on it.

### B. The check passed for a reason unrelated to correctness — 1 incident

**80 — Two "needs a fix before merging" findings, merged over in an
eighteen-second batch.** Flagged 2026-10-06 and 10-10; merged 2026-10-11. The
09:00 reviewer posted on #815 (10-06): "needs a fix before merging: the report
commits two real attendees' personal email addresses, unredacted, into a tracked
`reports/*.md` file." On #833 (10-10) it flagged that a venue pipeline report
"commits a compiled venue contact list" against an explicit rule: **21 email
addresses** and 4 phone numbers, under a `.gitignore` block headed "Scraped venue
contact data (NEVER commit)." On 10-11 the operator's account merged **seven
PRs, #815 and #833 among them, between 19:36:45Z and 19:37:03Z.** Neither file
was changed first. Merging is gated on CI, and CI does not read review comments.

The reviewer also missed most of it. From 10-06 the nightly reports began naming
customers. Six reports on `main` carry **118 lines** naming customers or their
email addresses: 10-06 (2), 10-07 (29), 10-08 (19), 10-09 (15), 10-10 (34),
10-11 (19). It flagged one of the six. The names are also in the squash-commit
messages, which no file edit can remove.

**Cost:** customers' names and personal email addresses, plus a compiled list of
business contacts, are on `main` in a private repository (`gh repo view`:
PRIVATE), nine days after the clean-up recorded as incident 70. Public exposure
is not established. **Not verified:** whether Taylor clicked those merges or a
session ran them on his word. **Anchor:** review comments on #815 and #833;
merge times from `gh pr view`; `git grep` on `origin/main` (2026-10-12).
**Detection:** `HUMAN`† — the 09:00 reviewer caught both, and its catch changed
nothing.

### C. Silent failure — the error was swallowed and read as absence of data — 2 incidents

**81 — Probing a new image URL before its deploy finished poisoned the CDN for a
week.** 2026-10-11, twice. `sparkdate.date` sits behind Cloudflare. A session
requested the new link-preview cover "before #870 deployed." Cloudflare cached
the 404 and applied the image rule's week-long `Cache-Control` to it, so the
cover "kept 404ing after" while Vercel's own origin served it with a 200
(`4d7407ba`). The same day the follow card's QR code "cached a 404 for the bare
URL mid-deploy … so the printed card showed no QR" (`9db1f97e`). Earlier, the same
TTL had served MC-15's replaced slide stale.

**Cost:** printed table cards with no QR code, and an event's link previews with
no image, both fixed by busting the URL. Whether the cards were used at an event
is not recorded. **Anchor:** `4d7407ba` (#871), `9db1f97e` (#857). **Detection:**
`HUMAN` — noticed the same day.

---

**82 — The unattended social check has never completed a run: it is waiting on
a permission prompt.** Set up 2026-10-11 15:52; stalled 16:51. Taylor's new
engagement standard sends replies with nobody watching: women's DMs answered
"immediately," facts-only replies to men and to comments. It runs as a desktop
scheduled task every two hours. "The 4:00 PM run on 10-11 has waited on a
PowerShell permission prompt since 16:51 local, and the log is still only its
header. Runs don't overlap, so every later run is skipped too." The task that
schedules the last four reels "stalled the same way on 10-10." A stalled task
looks exactly like a quiet day.

**Cost:** from 16:51 on 10-11, none of the scheduled inbox passes ran. That
includes the promise to answer women's DMs immediately. Nothing was sent wrongly,
because nothing was sent. **Still stalled** per `HANDOFF.md` on `main` on 10-12.
**Anchor:** `32529a94` (2026-10-11 19:39 EDT, #869); `HANDOFF.md:39-60`.
**Detection:** `HUMAN` — the session that built it checked the log.

### D. Destructive, or unreported, writes that report as fine — 2 incidents

**83 — The repair for incident 68 halved four two-seat rows and reported each
change as a repair.** Ran 2026-09-24; found 10-02; restored 10-03. The amount
repair looped over live Eventbrite attendees and compared each one against *the
one* ticket row for its email. A buyer-only 2-for-1 puts the buyer's email on
both seats, so the row met two attendees and kept the last one's gross. Run
36022736611 "halved four 2-for-1 rows that way, 2598 -> 1299, twice each," and
printed every change as a `repair amount` line. The 400-day dry run then planned
a further wrong write, 2749 → 2499 over a refunded site ticket, because the
planner took whichever row came first. It was never executed.

**Cost:** four two-seat rows understated by $12.99 each ($51.96) for nine days.
Restored 10-03 after a read-only confirmation. Whether a report summed them in
that window was not checked. **Anchor:** `a488f3ad` (#793), `434978c9` (#796);
`NOTHING_THREW_AUDIT_2026-10-02.md`, "Found along the way." **Detection:**
`LATER` — found by the audit while checking incident 68's cost line.

---

**84 — The check-in page moved itself onto the next event at 20:30 on event
night, and two guests became attendees of a night twelve days away.** 2026-10-06.
`lib/next-event.js:92` sets `DEFAULT_EVENT_DURATION_HOURS = 2`, the Oct 6 event
document had no `durationHours`, and the door's link carried no `?eventId=`. So
"the guard flipped at 20:30 ET while check-ins were still arriving." Check-ins at
21:42 and 23:02 "wrote confirmed Oct 20 registrations with no ticket and no
PaymentIntent." `event_registrations` drives the host roster and every pre- and
post-event email, so Oct 20 read 8 attendees on 6 tickets. Both the default and
the link are agent-written. `891ad0a0` dates the drift to midnight UTC (20:00
EDT) instead. I did not reconcile the two accounts.

**Cost:** four phantom documents (two registrations, two attendance logs),
deleted before Oct 20's first pre-event email (`0e01ab13`). Capacity was
unaffected. **Anchor:** `e68c42d9` (#817, the 10-07 report), `72178469` (#827);
`lib/next-event.js:92`. **Detection:** `HUMAN`† — the next night's analysis.

### E. Concurrency — several agents, one working tree — 0 new incidents

None this window.

---

## DECISION — recurrences, what changed, what's still open

**Recurrences.** Each one names the countermeasure it defeated.

- **Incident 44: a Resend refusal that resolves instead of throwing, this time
  with a real loss.** 2026-10-07. #492 (09-10) fixed that shape in the cron's
  passes and the match notice, and nowhere else. On 10-07 a returning-invite
  burst (151 sends at the 13:00 UTC cron) took the account past the free plan's
  100-a-day cap, and this time Resend enforced it. **Fifteen emails were refused
  and marked Failed,** five of them buyers' "your ticket is in" emails and six
  match notices, and two companions' ticket emails never reached Resend. "The
  logs said enroll succeeded" (`d3f14fcc`). The PR's own body still reads
  "Everything else in the window was fine … 151 returning invites," written three
  minutes before its second commit found the refusals. All were re-sent the next
  night (`cdcd51bf`, #819). **Defeated:** #492, which fixed a pattern one file at a
  time. **Caught by:** the operator, who brought Resend's own Failed list ("Reached
  the daily sending quota") to a session. The memory note from 09-09 had predicted
  "a ~100-email day at every event rollover."
- **Incident 67 — its cost became visible.** Vercel logged eight
  `checkinProfileHTML is not defined` errors between 18:06 and 18:46 EDT on 10-06,
  one for every walk-in with an incomplete profile at the biggest event yet.
  Fixed 10-07 (`d3f14fcc`). **Defeated:** the review that found it (09-22), and
  this document's "still open" list, which named it on 10-02. Neither caused a
  fix. An event night did.
- **Incident 69 — `pr-reviewer` is still unregistered.** Eight of eight
  reviewing runs say so (10-03 to 10-11). On 10-03 the reviewer skipped rather
  than substitute. The 10-02 run has no exit line. The 10-07 09:00 run never
  fired. **Defeated:** nothing was built against it. The previous edition's "no
  one has been told" is now false, because the reviewer tells Taylor every
  morning, and it is still unfixed.
- **Incident 70 — customer names in tracked files, again.** See 80: six nightly
  reports and their commit messages from 10-06, plus a HANDOFF PR title on 10-11
  that names a guest in full. `.claude/commands/nightly-ga4.md` contains no rule
  against naming customers. **Defeated:** #788's scrub, which cleaned the files
  but put no check in the path that writes new ones.
- **Incident 71 — attendees the importer could not import.** "Ten Eventbrite
  orders sold two seats and imported one" (`c792833b`, #823). The ten are
  half-price pairs, recovered from the fee column to the cent (`9799d127`).
  Every report's headcount was short by one or two. #643's order-form fix was
  applied to the two listings on sale on 09-19 and never to Oct 20 or Nov 10,
  so its docblock's "every Buyer Only 2-for-1 before 2026-09-19" is false.
  **Defeated:** a fix scoped to the listings that existed the day it shipped.
- **Incident 72 — Resend ids discarded.** 1,216 of 1,663 lead emails (73.1%)
  store no Resend id, because `api/cron-send-emails.js` discards `result.data.id`
  on two more tracks. Two misspelled addresses are still being mailed, one on an
  impossible `.comm` domain (`843e668c`, #838). **Defeated:** #737, which fixed
  the returning invite only.
- **Incident 49 — the false deduplication sentence.** `lib/ga4-mp.js:71` and
  `reports/ANALYTICS_METHOD.md:203` still say GA4 counts one transaction when
  both copies arrive (read 10-12). The 10-04 nightly cited it as though it
  predicted the double count. Nightlies from 10-05 to 10-10 retired it five
  times with fresh counter-examples (22 by 10-07). Nobody edited the sentence.
  **Defeated:** retiring a claim in a report is not editing it in the file every
  report reads first.

**Built in response, during the window.** Every item is named for the incident
that produced it, the same pattern every previous edition described.

- **`scripts/check-nothing-threw.js`** (#792): a gate on this catalogue's own
  counts. It would have refused the 10-02 public merge. Note that it reads only
  `FIELD_NOTES_YYYY-MM.md`, so this dated file is invisible to it as written.
- **`planEventbriteTicketRepairs`** (#793): plans repairs per row, not per
  attendee. Closes 83.
- **`splitAlreadyTicketed`** (#844): matches on email identity and order id.
  Closes 77.
- **`lib/door-checkin.js`** (#822): an existing registration is only marked and
  has its blanks filled, plus a backfill (#824) and "Did you buy under …?" at the
  door (#841). Closes 79, and the "door records the same person twice" thread
  that had sat unowned in HANDOFF since 08-31.
- **The dashboard skips hidden-tab refreshes, and `/api/health` names the
  cause** (#805, #806); the project is on a paid plan. Closes 76.
- **Resend refusals logged on the purchase and check-in paths** (#818); a
  re-send script with a ledger (#819). Partial for 44: still path by path.
- **`checkinProfileHTML` exported, with a structural test** (#818). Closes 67.
- **Cloudflare keeps images five minutes** (#871). Closes 81.
- **`scripts/reply-facts.js --check`** (#860): what an unattended reply may say.
  It has not yet run against a real reply, because of 82.

**Still open, as of `origin/main` on 2026-10-12:**

- **49** — both false sentences, unedited.
- **69** — `pr-reviewer` unregistered.
- **70 / 80** — six reports naming customers; no rule in the nightly prompt; no
  merge gate that reads a review verdict.
- **75** — the audit's corrections are not in the public October file.
- **82** — the social check is still waiting on its prompt.
- **71** — the order-form fix on the Oct 20 and Nov 10 listings (asked 10-09).

---

## What I did not verify

- **Live systems.** No Meta, GA4, Stripe, Firestore, Eventbrite, Resend or TikTok
  call was made. I read Gmail Sent metadata for one thread (78), PR comments and
  merge times, repo visibility, and the run logs. Dollar figures and counts come
  from commit messages and reports as quoted. Several of those are themselves
  nightly-analysis output, which the reviewer called accurate.
- **Who merged the batch** in 80, and **who noticed** 77. Neither is recorded.
- **The two dates for 84's drift:** 20:30 EDT in one commit, after midnight UTC
  in another.
- **Whether 76's hour lost a sale.** Not measured.
- **Whether the QR-less cards (81) were used at an event.**
- **Incidents 01–48 and 76–84 have not had a claim-by-claim audit.** This
  edition's own claims were re-opened against their anchors, the same standard
  incident 75 shows is not enough.

Looked at and left out, so a later pass need not rediscover them:

- **A real member's profile deleted in the database console on 10-04,** after a
  vague "delete it" instruction meant for a leftover login, then restored with a
  point-in-time read. Anchored only to memory `retiring-a-duplicate-login`; no
  commit or log records it.
- **Gmail inline replies that re-sent the quoted original, with an expired
  wrapped link, to two partners** (10-02, 10-04; `22dcd5c6`, `b1e8f7f2`). Real
  and anchored, but the cost was thin: the quoted offer was a standing one.
- **A duplicate Eventbrite Ads campaign on Oct 20** ($48.08, 27 clicks, zero
  orders, paused 10-05). It was created 09-30, and by whom is not recorded.
- **`page_path` set with the query string on thirteen pages,** splitting 21.5%
  of landing sessions into duplicate rows (#852). Who wrote those lines was not
  traced.
- **Stray carriage returns in queue captions** (`f32a3d10`, `027e8c73`) and **a
  daylight-saving hour error in a handoff** (#830). Both were caught before they
  did anything.
- **OAuth expiry** (the 10-07 review, the 10-12 nightly and review): platform
  auth, no agent in the chain.

Standing, carried forward and updated:

- **One project, one operator, five and a half months.** These incidents come
  from a single small business, not a survey. The frequencies are not a base
  rate for anything.
- **Selection bias runs in the obvious direction, and this window shows it.**
  This catalogue holds the failures that were *noticed*. In a ten-day window
  that means mostly the fast ones: eight of ten were caught within a day. The
  slow failures from these ten days will turn up in later editions, under
  `LATER`, or not at all.
- **This document's error rate against the live systems has been measured three
  times and was never zero** (40, 51, 75).
- **"Cost" means what was lost or nearly lost.** Where an incident was caught
  before costing anything (83's dry-run write, 82), that is stated rather than
  counted as a loss.
- **Names, account identifiers and customer records are omitted throughout.**
