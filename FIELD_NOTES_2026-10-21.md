# Ten new incidents, none caught by anything automated, and the nightly that exists to catch them failed eleven nights running

**Window: 2026-10-12 21:10Z through 2026-10-21.** The previous edition,
`FIELD_NOTES_2026-10-12.md`, closed at incident 84; its commit in
`nothing-threw` is `427ab433` at 2026-10-12 21:10:55Z, so that is the
watermark. This edition continues the numbering: **85–94**. It is the seventh
pass, filed under its end date because the October file name is taken.

Five of the ten happened inside the window. The other five happened on
2026-09-12 and 09-17 and were written down at the time in `HANDOFF.md`, in two
entries addressed to "the October field notes" (`d8c3b923`, #560;
`670f263c`, #620). The fifth pass (49–74) numbered one item from those lists
(incident 74), ruled the 09-17 README and essay items "not numbered" because it
could not check them from the private repository, and said nothing about six
of the seven 09-12 items. This pass checked the 09-17 items against the public
`nothing-threw` commit history, where they can be read, and swept the six. Each
is counted once, here, with both dates.

The window held 74 commits on `origin/main` from the watermark to `ab368425`
(#875 to #947), 10 nightly runs and 10 report-review runs (10-13 to 10-22),
and 41 memory files written from the watermark through 10-21. Six sources were swept: the memory files, `CLAUDE.md` (four
changes, all new rules, none a failure record), the reports and commit
messages, the run logs (decoded from UTF-16), git history, and `HANDOFF.md`'s
unowned threads. Every anchor below was re-opened against git, GitHub, the
logs or the working tree on 2026-10-22.

**The one-line finding:** 0 of 10 were caught by anything automated. The one
countermeasure built this window that would have been a `GATE` (a spend sync
that refuses to write when its daily rows miss the provider's own total) was
built after the operator noticed the number was wrong, 29 days in (94).

| | |
|---|---:|
| New incidents, numbered 85–94 | 10 |
| Caught by anything automated (`GATE` + `THREW`) | 0 of 10 |
| Nightly analysis runs that ended `analysis FAILED`, 10-12 → 10-22 | 11 of 11 |
| Recurrences of catalogued incidents | 3 |
| This document's own errors, measured | 6 (incidents 40, 51, 75, 90, 91, 92) |

---

## EVIDENCE — how they were caught

| Detection | This edition (85–94) | Previous cumulative (84) | Cumulative (94) |
|---|---:|---:|---:|
| `GATE` — a deterministic check refused it | 0 | 2 | 2 |
| `THREW` — an actual error surfaced where someone reads it | 0 | 8 | 8 |
| `HUMAN` — someone distrusted a number or claim, within about a day | 5 | 28 | 33 |
| `OPERATOR` — reported as "data went missing" / "this looks wrong" | 1 | 1 | 2 |
| `LATER` — found by an unrelated dig, or days on | 4 | 45 | 49 |
| **Total** | **10** | **84** | **94** |

**Rules for the modes,** the same as the previous edition. `HUMAN` includes a
session that distrusted a specific claim and checked it within about a day.
**†** marks a catch made by an LLM step. Those count as `HUMAN`, never as
automated. One of this edition's five `HUMAN`s is † (87). Three of the
`HUMAN`s (89, 90, 93) were recorded at the time as `LATER` in the `HANDOFF.md`
pre-lists; each was caught by the same session within hours, so by this
catalogue's rules they are `HUMAN`.

**Is the automated share moving?** No, and it went down. Per pass: 11% (2/18)
→ 9% (2/22) → 15% (5/34) → 13% (6/48) → 12% (9/74) → 12% (10/84) → **11%
(10/94)**. This pass alone is 0 of 10. Seven passes and five and a half months
have stayed between 9% and 15%.

**The nightly ledger.** `<date>.log` exists for every night from 10-12 to
10-22. All eleven end `Failed to authenticate: OAuth session expired and could
not be refreshed`, then `ERROR: Claude Code exited 1` and
`=== Run complete (analysis FAILED) ===`, between 02:02:41 and 02:03:51. All
eleven `review-<date>.log` files end the same way, 13 seconds to 2 minutes 48
seconds after starting.
The data pulls before the analysis step kept landing every night. No commit on
`origin/main` between 10-12 and 10-21 mentions the outage. It was first named in
the 10-21 compute ranking (`reports/COMPUTE_PRIORITIES_2026-10-21.md`, job 5,
on branch `compute/ranking-1021`, PR #943), ten failed nights in. The previous two
editions ruled an OAuth expiry "infrastructure with no agent in the chain, so it
is not numbered," and this edition keeps that ruling (see DECISION).

| Nights | What the log says |
|---|---|
| 10-12 → 10-22 nightly (11) | `OAuth session expired` at ~02:02–02:03, `analysis FAILED`; pulls completed |
| 10-12 → 10-22 review (11) | `OAuth session expired`, `review run failed (exit code 1)`, 13–168 s after start |

---

## MECHANISM

### A. The agent was confidently wrong — 8 incidents

**85 — A diagnosis script called three women the best lever for a short room,
and all three were already accounted for.** 2026-10-13/14.
`scripts/event-gender-diagnosis.js` printed in FLAGS, for Loxley's Oct 20: "3
women hold a ticket with an unused free +1 — the highest-yield lever; invite
them personally." Its list was every confirmed woman with a ticket who was not
herself a +1, which is who the reminder emails send the +1 link to. Two of the
three were comps. The third was a buyer who had already named her friend, and
the friend had been entered by hand as the second comp. The import screen has
no notes field and writes no link between the two tickets, so nothing in the
data tied them. "Two analysis sessions repeated the number to Taylor as the best
lever" (`9b5a3a82`). The T-7 report said the same and that one woman's +1 note
was an unsent draft; its correction says the note went out on 10-09 and was
answered the same afternoon (`reports/LX2_ADS_FIXES_2026-10-13.md:171-176`).

**Cost:** the event's best-lever recommendation pointed at a room's three
women who had nothing left to give, at T-7, while the room was short on women.
The real reachable pool was 68 past women attendees. Whether any of the three
was contacted on the strength of it is not recorded. **Anchor:** `9b5a3a82`
(2026-10-14 00:38 EDT, #889); `reports/LX2_ADS_FIXES_2026-10-13.md:171-176`.
**Detection:** `HUMAN` — corrected within a day. Neither the commit nor the
report says who noticed.

---

**86 — The pre-event email told four women their +1 was "still free" after
their friend was already booked.** 2026-09-07 to 10-05; found 10-14. The T-7 and
T-1 pass in `api/cron-send-emails.js` rendered the add-guest link for every
confirmed woman holding a ticket "and never looked for a companion." A buyer who
had named her friend at the checkout 2-for-1 was told her +1 was still free. The
link then answered "Your +1 is already booked." Per the send locks, **7 such
emails reached 4 women**: Real Court Sep 8 (1 woman, 1 email), Loxley's Sep 22
(2 women, 4 emails), Tellus Oct 6 (1 woman, 2 emails). The next was due for a
Nov 10 buyer about Nov 4 (`95632971`).

**Cost:** seven customer emails that made an offer the system then refused,
across three events. No seat was mis-sold, because the endpoint refused the
second guest. **Anchor:** `95632971` (2026-10-14 01:24 EDT, #891);
`tests/pre-event-plus-one-booked.test.js` (17 of its 21 tests fail on the old
cron). **Detection:** `LATER` — found five weeks after the first email, by the
same dig that produced 85.

---

**87 — The job that commissioned this edition started from a false premise:
rescue a draft that had been public for nine days.** 2026-10-21. A finder agent
in the 10-21 compute ranking proposed rescuing "the stranded 10-12 draft"
(incidents 75–84) from the worktree `field-notes-2026-10-ed2`, and cited
`scripts/check-nothing-threw.js` reporting 8 FAILs on that directory and 4 on
`main`'s `reports/`. The draft there is byte-identical, line endings aside, to
the public `FIELD_NOTES_2026-10-12.md`, merged as `nothing-threw` PR #4 at
2026-10-12 23:44Z. `9aede3d2` (#875, 19:51 EDT that day) says so in its body:
"The catalogue's sixth pass ships as FIELD_NOTES_2026-10-12.md." The FAILs came
from pointing the gate at the wrong directory. Its usage line asks for "a clone
of nothing-threw"; `reports/` has no README and holds stale copies of
2026-05..09 and nothing of 49–84. Run on the public files, it reports 84
incidents and 0 FAIL. A skeptic agent refuted the premise. The ranking report
still says "a live worktree holds the edition" (line 106), and the task text
this pass was given repeated that the worktree "may be a live session." Its
branch has 0 commits ahead of `origin/main`, 74 behind `ab368425`, and a tip
of 10-12 13:39 EDT.

**Cost:** nothing was built on the premise, because the job was rewritten
before it ran; the rewrite is this edition. The false line is in an open PR.
**Anchor:** `origin/compute/ranking-1021` (`3235be0d`, PR #943's head),
`reports/COMPUTE_PRIORITIES_2026-10-21.md:106`; `9aede3d2` (#875);
`nothing-threw` `6c86a0d2`. **Detection:** `HUMAN`† — the ranking's own skeptic
stage.

---

**88 — The first PII scrub was declared clean while seven real names stayed at
HEAD in a public repository.** Scrub 2026-09-04 (#434); found 09-12. "Seven
real names were live at HEAD, three in reports/docs", one of them in a report
that "names an attendee and how she got her ticket. The audit had searched
emails, phones and paths, never bare first names in prose." The repository was
public until 09-12. This is the first occurrence of the shape incident 70
records a month later.

**Cost:** customers' first names readable at HEAD of a public repository for
eight days after the scrub that was meant to remove them, in a singles-event
context. **Anchor:** `HANDOFF.md:2378-2383` on `origin/main`, written 09-13 by
`d8c3b923` (#560). The fixing commits were folded into the 09-12 history
rewrite, so this entry is the only record. **Detection:** `LATER` — found eight
days on, while preparing the rewrite.

---

**89 — A name-replacement map for the history rewrite would have renamed an
ordinary word 476 times.** 2026-09-12. The map was derived by pairing removed
and added lines in the #434 diff. It "silently paired capitalised ordinary
words," producing five false positives, one matching **476 times across 14
files**, including `content/brand.json` and `content/queue.csv`. Applied, it
would have rewritten every commit in history and renamed live testimonial
attributions to invented people. The session counted occurrences per token
before running it. The repair commit the next morning says the same mistake
was avoided again in 39 case-variant matches, and "that the verification which
cleared that rewrite checked exact literals only."

**Cost:** none; caught before it ran. **Anchor:** `d8c3b923` second commit
message (#560, 2026-09-13); `HANDOFF.md:2371-2377`. **Detection:** `HUMAN` —
the session's own per-token count.

---

**90 — This document's own error, found in 2026-09 and never catalogued: "the
automated share falls every pass."** 2026-09-12. The fourth pass claimed
"every edition so far has reported a lower number than the one before it." The
sequence was 11%, 9%, **15%**, 13%. It went public in `nothing-threw`
`a38dcc59` at 15:27Z and was corrected in `83d661ce` at 18:44Z. The correction
was "caught by re-reading edition 1's own line, 'Two of eighteen were caught by
the system itself'" (`0fcfe34c`).

**Cost:** the catalogue's headline trend was false in public for 3 hours 17
minutes, in the markdown, the artifact and the public repository. Corrected in
the file, not silently. **Anchor:** `0fcfe34c` (2026-09-12 14:44 EDT, #557);
`nothing-threw` `a38dcc59`, `83d661ce`. **Detection:** `HUMAN` — the same day.

---

**91 — This document's own error: the essay's "roughly a fifth of the
cost-per-ticket" matched no figure.** Published 2026-09-12 18:44Z; corrected
09-18 02:38Z. The essay said a cheaper ad channel had run "at roughly a fifth of
the cost-per-ticket of the channel I was being advised to keep." On each
platform's own attribution it was $20.77 a ticket against $228.50, about a
tenth. It was found while re-deriving the numbers for the rebuilt README.

**Cost:** a headline cost comparison in the public essay understated the gap
by half for five days. **Anchor:** `nothing-threw` `70389aea` (the removed line
is in its diff of `SIX_OF_FORTY_EIGHT.md`); `670f263c` (#620);
`HANDOFF.md:2410-2414`. **Detection:** `LATER` — five days on, while doing
something else.

---

**92 — This document's own error: four sentences in the public essay disagreed
with the catalogue it summarises.** Live by 2026-10-12; corrected 10-14 13:22Z.
The essay said "129 of its 136 sessions" after incident 51 had been corrected
to 135 on 10-12. It said "Seventy-two entries have not had that treatment,"
written at 74 and before the third audit. It said "None of them caught
anything" directly under a table showing `GATE` 2. It said "roughly fourfold
(18 to 84)" for a ratio of 4.7. The gate checks the essay's table against the
chips, so the table was right and the prose around it was not. The fixing
commit's subject says "three lines" (two of the four sat in one paragraph);
its body lists all four. They were found while renaming the essay's file to match its title, which the operator
had asked for that morning.

**Cost:** four false sentences in the public essay for about two days, one of
which denied the catalogue's own two automated catches. **Anchor:**
`nothing-threw` `919c2586` (2026-10-14), `9fcae226`; `ec5afb6d` (#898).
**Detection:** `LATER` — found during an unrelated rename.

### B. The check passed for a reason unrelated to correctness — 1 incident

**93 — "Make the repository private" printed a usage message, and both parties
read it as done.** 2026-09-12. During the PII remediation,
`gh repo edit --visibility private` without
`--accept-visibility-change-consequences` "prints usage and changes NOTHING
while looking like it ran." The repository stayed **public** "while both of us
believed it was closed." It was found by running `gh repo view --json
isPrivate` on a hunch. The repository is private today (`gh repo view`,
2026-10-22).

**Cost:** a repository whose history still held customer names stayed public
for an interval nobody recorded, after its owner believed it had been closed.
**Anchor:** `HANDOFF.md:2367-2370`, written by `d8c3b923` (#560); memory
`history-rewrite-leaves-pr-refs`, step 1. **Detection:** `HUMAN` — the same
session checked the state instead of the command's output.

### C. Silent failure — the error was swallowed and read as absence of data — 1 incident

**94 — The Meta spend sync wrote $0 for most ad-set-days for a month, and Net
Revenue read $533 high.** Began 2026-09-22; found 10-21. `scripts/sync-meta-spend.js`
asked for `spend,clicks,inline_link_clicks,impressions` at ad-set level with
`time_increment=1`, a field list set on 08-24 (`c8506f8f`, #296). From the
campaigns built on 09-22, "with `inline_link_clicks` in that daily request, Meta
answers `spend: "0"` on 50 of the 71 October ad-set-days." There was no error,
so the sync wrote the low numbers every four hours. Firestore `ad_spend` held
September $149.94 short and October $298.67 short: Loxley's Oct 20 at $38.51
against Meta's $290.82. Net Revenue and blended CAC both read the missing spend
as never spent. The same commit found the Net Revenue card had also never
subtracted Stripe's fees ($84.47).

**Cost:** **$448.61** of ad spend missing from the books for 29 days, across
the two events that ran in that window. Net Revenue read $1,676.70 against a
correct $1,143.62; $448.61 of that $533.08 gap is this incident, and $84.47 is
the separate Stripe-fee omission. CAC read $17.17 against $20.37. Any decision made from
those two cards in that month used them. Firestore stays short until a one-off
45-day run of the fixed sync. **Anchor:** `903532f2` (2026-10-21 23:49 EDT,
#940); `reports/ADMIN_NET_REVENUE_AUDIT_2026-10-21.md`; `c8506f8f`.
**Detection:** `OPERATOR` — "Net Revenue in the admin looks adjusted higher and
does not look accurate."

### D. Destructive, or unreported, writes that report as fine — 0 new incidents

None this window.

### E. Concurrency — several agents, one working tree — 0 new incidents

None numbered. One recurrence of 18 is recorded below.

---

## DECISION — recurrences, what changed, what's still open

**Recurrences.** Each one names the countermeasure it defeated.

- **Incident 40 — the rebuilt README repeated the error 40 exists to correct.**
  Until 2026-09-18 the public README's incident 10 still read "Neither flag has
  been fixed; both need an elevated shell nobody has run," the claim incident 40
  (found 09-12) shows was false. Removed in `nothing-threw` `70389aea`.
  **Defeated:** 40's correction, made in the field-notes file and not carried to
  the page readers land on.
- **Incident 18 — a staged file in the shared main checkout, by a new route.**
  2026-09-12. A command handed to the operator ran `git clone … ~/path` (PowerShell
  passes `~` literally to native programs, so it cloned into a folder named `~`
  inside the repository), the next `cd` failed, and `; git add -A` then staged
  five unrelated untracked files in the main checkout. Reset before any commit.
  `670f263c` records the same `git -C ~/…` shape being copied back in on 09-17.
  **Defeated:** the worktree rule, which governs sessions but not the commands
  they hand to a person. Memory `powershell-tilde-and-cd-chains`.
- **Incident 70 — customer first names in a tracked report, again.**
  `reports/LX2_ADS_FIXES_2026-10-13.md:171-174` on `origin/main` names three
  guests in its correction paragraph; #889's PR body prints the same three
  names. **Defeated:** #788's scrub, which put no check in the path that writes
  new files, and the 09:00 reviewer, which has not run since 10-11.

**Not numbered, again: the OAuth expiry.** Eleven nightly analyses and eleven
reviews failed from 10-12 to 10-22, and nothing in git named it for ten
nights. This catalogue's standing ruling is that an expired platform login is
infrastructure with no agent in the chain. That ruling is the operator's to
change. If it changes, this is incident 95, class C, detection `LATER`; the
outage began on 10-12 (the previous edition's last night), and the earlier
expiries (09-13; the 10-07 review) become its recurrences. What
did not catch it: the launcher's own check matches only `Not logged in`
(ranking job 5), and no session reads the logs at start.

**Built in response, during the window.** Every item is named for the incident
that produced it, the same pattern every previous edition described.

- **`check-nothing-threw.js` reads dated editions** (#875) and **requires the
  essay's file name to match its title** (#898). The first closes the previous
  edition's note that its own file was invisible to the gate.
- **`plusOneInventory`** (#889): splits paid buyers from comps, marks a booked
  +1 by `linkedTicketId`, never says "unused." Closes 85.
- **The pre-event email withholds the +1 box once a companion is booked**
  (#891), with a test that runs the real pass against the endpoint's own rule.
  Closes 86.
- **The spend sync reconciles before it writes** (#940): spend and link clicks
  are separate requests, and the sync refuses to write when the daily rows miss
  Meta's window total by more than 1%. Closes 94's mechanism. It is the first
  countermeasure in this catalogue that checks a written number against an
  independent total, and it is the shape that would have turned 94 into a
  `GATE`.
- **Actions caches `node_modules`** (#931) after the account used its 2,000
  free minutes on 10-18. Not an incident (see below).

**Still open, as of `origin/main` on 2026-10-22:**

- **49** — `lib/ga4-mp.js:69-71` and `reports/ANALYTICS_METHOD.md:201-203`
  still say GA4 counts one transaction when both copies arrive.
- **69** — whether `pr-reviewer` is registered is unknown: no review has run
  since 10-11.
- **70** — the report and PR body named above.
- **82** — the social check logs again: CHECK lines on every day from 10-13 to
  10-22. But 10-13, 10-15 and 10-16 logged 3, 2 and 4 of seven scheduled slots.
  I did not find out why.
- **87** — line 106 of the ranking report, in open PR #943.
- **94** — the one-off 45-day sync that repairs Firestore (`HANDOFF.md:1743`).
- **The OAuth outage** — the repair is the operator's (`claude auth login`,
  then `-SmokeTest`).

---

## What I did not verify

- **Live systems.** No Meta, GA4, Stripe, Firestore, Eventbrite, Resend or
  TikTok call was made. I read GitHub run lists, PR bodies, repository
  visibility, the public `nothing-threw` commits and files, the run logs, and
  the engagement log's per-day line counts. Dollar figures and counts come from
  commit messages and reports as quoted.
- **Who noticed 85**, and **whether any of its three women was contacted** on
  the strength of it.
- **How long the repository stayed public after 93's command.** Not recorded.
- **88, 89 and 93 rest on one contemporaneous record each,** a `HANDOFF.md`
  entry the same session wrote the next morning (`d8c3b923`), plus a memory
  note for 93. The fixing commits were folded into the 09-12 history rewrite.
  The same standard admitted incident 74.
- **Whether a decision was made from the low Net Revenue or CAC figures** in
  94's month. Not traced.
- **Why the social check logged fewer runs** on 10-13, 10-15 and 10-16.
- **Incidents 01–48 and 76–94 have not had a claim-by-claim audit.** This
  edition's own claims were re-opened against their anchors, the same standard
  incident 75 shows is not enough.

Looked at and left out, so a later pass need not rediscover them:

- **A scrub helper that stripped every blockquote, the corrections included**
  (09-12). Caught by a post-condition grep before publishing; the public
  September file carries its correction notice. No cost and no anchor outside
  `HANDOFF.md:2384-2388`.
- **The other three 09-17 items** (`HANDOFF.md:2415-2429`): a render check that
  read an "Argument list too long" error as zero tables, a removal policy
  recommended as a fallback that does not apply (corrected in `670f263c`), and a
  handoff that over-gated the essay behind a support queue. All three were
  caught before they cost anything, and the first has no anchor outside
  `HANDOFF.md`.
- **The GitHub Actions free minutes ran out on 10-18 at 21:13 local.** The 10-15
  trim (#913) had cut the tests workflow; the larger waste was `npm ci` taking
  ~75 s of every ~95 s job, billed as two minutes (#931). The operator added a
  card about an hour later. `gh run list` shows no failed run between 10-14 and
  10-22 except two test runs on 10-14, so no cost is established.
- **A guest's complaint about her photo in a paid ad** (10-18, #930). The photo
  was used under the operator's blanket clearance of the folder on 09-29, so
  the agent followed the rule it had. The complaint changed the policy (a door
  opt-out, #927) and is a policy matter, not an agent failure.
- **An Instagram DM truncated at 1,000 characters, which the session's
  whole-text guard refused to send** (10-14). That would be a `GATE`. It is
  anchored only to memory `instagram-dm-composer-traps`, the same reason the
  previous pass left out its 10-01 permission-layer catch, so it is not counted.
- **The `/review-ask` command (#937) said a draft sent by `draftId` keeps its
  links clean.** Its own cited memory says the connector rewrites links in
  drafts too. The 10-21 draft's two review links were wrapped, then restored in
  Gmail before it went to 32 guests (`e69e2b50`, #941). Caught before sending,
  and the cost was nil.
- **The Gmail connector's `reply` re-attached a quoted message's photos**
  (10-18, memory `gmail-connector-wraps-links-in-google-redirects`). The photos
  went back to their own sender. Memory-only, no cost.
- **A merge agent in the 10-21 ranking stalled on the size of its input**
  (ranking report, MECHANISM). The merge was then done in code. It stalled
  visibly and nothing was lost.

Standing, carried forward and updated:

- **One project, one operator, five and a half months.** These incidents come
  from a single small business, not a survey. The frequencies are not a base
  rate for anything.
- **Selection bias runs in the obvious direction, and this pass shows it from
  both ends.** This catalogue holds the failures that were *noticed*. Half of
  this edition was noticed five weeks ago and then waited in a handoff file,
  because the pass that should have taken it in dropped six of its items without
  a word. And the one automated reader of the logs, the nightly analysis,
  could not run for eleven nights. Failures from those nights that nobody
  happened to notice are not here.
- **This document's error rate against the live systems has been measured six
  times and was never zero** (40, 51, 75, 90, 91, 92).
- **"Cost" means what was lost or nearly lost.** Where an incident was caught
  before costing anything (87, 89), that is stated rather than counted as a
  loss.
- **Names, account identifiers and customer records are omitted throughout.**
