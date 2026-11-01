# Six new incidents, three caught by machines, and the one that mattered most was caught by nothing

**Window: 2026-10-22 (`ab368425`) through 2026-11-01 (`3f404192`).** The
previous edition, `FIELD_NOTES_2026-10-21.md` (#971), closed at incident 94,
and its anchors run to `ab368425`. That commit is the watermark. This
edition continues the numbering: **95–100**. It is the eighth pass. It is filed
under its end date, like the seventh.

The window held 113 commits on `origin/main` (#943 to #1055), 11 nightly runs
and 11 report-review runs (10-22 to 11-01), and 14 memory files written after
the watermark. All six sources were swept: the memory files, `CLAUDE.md` (three
changes), the reports and commit messages, the run logs (decoded from UTF-16),
git history, and `HANDOFF.md`'s unowned threads. Every anchor below was
re-opened against git, GitHub or the working tree on 2026-11-01.

**The one-line finding:** half of this pass was caught by something automated,
the highest share of any pass. But all three were CI failures on the code
agents write for themselves. The three failures that reached the business were
caught by people, eleven days to ten weeks late, and one was a photo a guest
had asked to be kept out of.

| | |
|---|---:|
| New incidents, numbered 95–100 | 6 |
| Caught by anything automated (`GATE` + `THREW`) | 3 of 6 |
| Of those, failures outside the repository's own tests | 0 of 3 |
| Nightly analysis runs that ended `analysis FAILED` | 2 of 11 (10-22, 10-23) |
| Recurrences of catalogued incidents | 2, plus one open question answered |

---

## EVIDENCE — how they were caught

| Detection | This edition (95–100) | Previous cumulative (94) | Cumulative (100) |
|---|---:|---:|---:|
| `GATE` — a deterministic check refused it | 1 | 2 | 3 |
| `THREW` — an actual error surfaced where someone reads it | 2 | 8 | 10 |
| `HUMAN` — someone distrusted a number or claim, within about a day | 0 | 33 | 33 |
| `OPERATOR` — reported as "data went missing" / "this looks wrong" | 1 | 2 | 3 |
| `LATER` — found by an unrelated dig, or days on | 2 | 49 | 51 |
| **Total** | **6** | **94** | **100** |

**Rules for the modes,** unchanged. `HUMAN` includes a session that distrusted a
specific claim and checked it within about a day. A catch made by an LLM step is
`HUMAN`, never automated; none this pass. A CI failure that surfaced **after**
merge is `THREW`, not `GATE`, because nothing was refused (100).

**Is the automated share moving?** Up, for the first time in three passes. Per
pass, cumulative: 11% (2/18) → 9% (2/22) → 15% (5/34) → 13% (6/48) → 12% (9/74)
→ 12% (10/84) → 11% (10/94) → **13% (13/100)**. This pass alone is 3 of 6.
Read it narrowly. All three automated catches are the test suite catching an
agent's change to code or data in this repository: 96 on a PR branch, 98 on
every branch, 100 on `main`. None of the three caught anything that reached a
customer, a public page or a scheduled post. Those three (95, 97, 99) were
caught as before: by the operator, or by someone looking at something else.
After eight passes, the countermeasures that work are the ones that
live inside CI.

**The nightly ledger.** `<date>.log` and `review-<date>.log` exist for every
date from 10-22 to 11-01, so no night is missing. The OAuth outage that ran
through the previous edition ended here. The nightly failed with `OAuth session
expired` at 02:03:02 on 10-22 and 02:03:07 on 10-23, and the review failed at
09:01:02 on 10-22. A hand re-run at 08:36 on 10-23 completed (`-AnalysisOnly`),
and that morning's review completed. Every log from 10-24 on ends
`=== Run complete ===`. `ed4e9961` (#986) calls the 10-24 run "the first
unattended nightly since 10-11." The standing ruling holds: not numbered (see
DECISION).

| Nights | What the log says |
|---|---|
| 10-22, 10-23 nightly | `OAuth session expired`, `analysis FAILED`, ~02:03 |
| 10-22 review | `OAuth session expired`, `review run failed (exit code 1)` |
| 10-24 → 11-01 nightly and review (18) | `=== Run complete ===` |

---

## MECHANISM

### A. The agent was confidently wrong — 2 incidents

**95 — The careers page an agent wrote put a four-hour shift on a two-hour job,
and the business costed hosts at double for ten weeks.** 2026-08-23; found
10-30. The operator gave the $22 rate. The four hours were the agent's: the
page said "$22 an hour, which on a four-hour evening is about $88," the terms
grid read "$22/hour · about $88 a night," and the JobPosting structured data
said "Roughly four hours" (`71fac1b9`, #278, `public/careers.html:304`, `:319`,
`:324`, `:450`). The commit body gives the reasoning: "The per-night figure
matters more than the hourly one for work that is four hours at a time." No
source is cited for the four hours. The figure then went into strategy work: "A
night costs about $90–100 in host pay at the careers-page rate"
(`reports/EXPANSION_STRATEGY_REVIEW_2026-10-12.md:141`). On 10-30, while a
candidate follow-up was being drafted, the operator said how hosts are
actually paid: "I typically pay my ppl for 2 hours 6:30 to 8:30 20-25 an hour"
(`407dade3`, #1021).

**Cost:** for 69 days the public careers page and its structured data
advertised a night of about twice the real hours. The expansion review's host
cost per night was about double, in an argument about which cities could
afford a host. A candidate never answered the pay question (memory
`host-pay-is-two-hours`); whether the $88 figure had anything to do with that
is not known. **Anchor:** `71fac1b9` (2026-08-23, #278);
`reports/EXPANSION_STRATEGY_REVIEW_2026-10-12.md:141`; `407dade3` (#1021,
merged 2026-10-31 16:29Z). **Detection:** `LATER` — found ten weeks on, during
hiring work.

---

**96 — An agent tidied an agreed partner round into history, and the queue lint
stopped reading the agreement.** 2026-10-30. Recording the TL3 partner asks
(99), the session moved one café's agreed MC2 round into `previous_rounds` in
`content/women-surfaces.json`. `featuredHandles()` reads only the current
round, so that café's tag on MC2-22 (Nov 4) became "not an agreed partner".
That is a lint error, and the commit that fixed it says it "would stop all
queued posting once merged." The PR's `tests` run failed on `19b8a2c1` at
2026-10-30 17:15Z. The fix (`44c9b803`, 10-31 16:56Z) taught the lint to count
an agreement in `previous_rounds`, and that run passed.

**Cost:** none. The PR could not merge red, so nothing reached `main`. The
record of ten sent asks waited about 24 hours. **Anchor:** `44c9b803` and
`19b8a2c1` in #1029 (squashed as `a9083af7`); `gh run list` for the branch.
**Detection:** `GATE` — CI refused the PR.

### B. The check passed for a reason unrelated to correctness — 2 incidents

**97 — "Every scheduled copy replaced," except the one whose file name did not
say which photo it held.** Pulled 2026-10-18; found 10-29/30. A guest pictured
in IMG_9547 asked to be kept out of future ads and posts, and the operator
agreed. The sweep that night (`fc899431`, #930, subject: "every scheduled copy
replaced") grepped `queue.csv` and `motion-ads.json` for the IMG number. The
MC2-25 question card, approved to post Sun Nov 1 at 12:30 on Instagram,
Facebook and X, had been rendered on 10-11 from IMG_9547. Its file is named
`MC2-25.jpg`, and its photo is recorded only in a `render.js` outside git, so
the grep returned nothing and the sweep read that as clean. It was found while
building the next batch of question cards; PR #1024 was opened 10-30 04:03Z.
The card was re-rendered from another photo, X's scheduled post was edited in
place (`b6f3d992`, 10-31 17:08Z), and the Facebook scheduled post was deleted
and re-sent (`769bc8b6`, 19:41Z).

**Cost:** nothing published, but only just. Facebook's copy of the pulled photo
was removed about 21 hours before its slot. For 12 days, two platforms held a
scheduled post the business had promised a guest would not run. **Anchor:**
`fc899431` (#930); `b9d43841` (#1024) and its PR body; `b6f3d992` (#1033);
`769bc8b6` (#1034). **Detection:** `LATER` — found eleven days on, while working
on other cards.

---

**98 — A test's clock was frozen on 10-27 in the half that was not reading
it, and every branch went red the next night.** 2026-10-27/28. On 10-27 a
fixture's early-bird deadline passed and `tests/checkout-remind.test.js` failed.
The fix froze `Date` inside the test's vm (`d6b1d800`, #1010: "The vm now gets
a Date frozen at the file's NOW"), and the suite went green. But `lib/` is
loaded in the host realm, and `normalizeEvent()` there still defaulted to
`Date.now()`. At 2026-10-28 04:00Z the fixture's second deadline passed, and the
test failed again "on every branch, main included" (`a45123ca`, #1013). The
same dig found that `sendCheckoutReminders` and `sendCheckoutRecovery` took a
`nowMs` argument and then judged the early bird against the wall clock.

**Cost:** about 21½ hours of red CI on every branch, until #1013 merged at
10-29 01:34Z. It held #1007, the PR that created the morning due list, which
was due to run first at 08:33 that day
(`reports/ROUTINES_HEALTH_2026-10-28.md` §6). It merged a minute after #1013,
so the morning list was not lost. No real send changed, because production
calls the two functions with the real time. **Anchor:** `d6b1d800` (#1010);
`a45123ca` (#1013). **Detection:** `THREW` — the test failed.

### C. Silent failure — the error was swallowed and read as absence of data — 1 incident

**99 — Ten partner asks were planned for 10-27 in a report, and nothing reads
reports.** 2026-10-27; found 10-28. `reports/OUTREACH_STRATEGY_2026-10-25.md`
§7 (#997) put the TL3 partner asks on Tue Oct 27 with status "drafts", meaning
Claude builds them and Taylor sends. No session built them. The tool that tracks
outreach, `build-outreach-pack.js --status`, printed "No nudges due," because
it reads nudges from `women-surfaces.json` and the plan lived in a markdown
table (`reports/ROUTINES_HEALTH_2026-10-28.md` §4). The same shape held for
venues: "the venue log has not been touched since and lists 21 follow-ups due
or overdue, while three venues that replied this week do not appear in it"
(same report, standfirst). The first version of the due list, built that same
day (`cbfae276`, #1007), printed only today and later, so it would have dropped
the missed asks the next morning. The operator's request that evening ("find
all the routines that are broken") is what found it. The next day: *"I can't
keep silently failing like this"* (`eac2e115`, #1023).

**Cost:** one ask went on 10-29 (#1020) and nine on 10-30 (`a9083af7`,
#1029), two and three days late, and up to two days into the Oct 28 → Nov 4 window they were asking partners to
share in. No partner is recorded as declining because of the timing. **Anchor:**
`reports/OUTREACH_STRATEGY_2026-10-25.md` §7 (line 259 on);
`reports/ROUTINES_HEALTH_2026-10-28.md` §4; `cbfae276` (#1007); `eac2e115`
(#1023); `a9083af7` (#1029). **Detection:** `OPERATOR` — "find all the routines
that are broken."

### D. Destructive, or unreported, writes that report as fine — 0 new incidents

None this window.

### E. Concurrency — several agents, one working tree — 1 incident

**100 — Four PRs from the fleet merged in 98 seconds, each green alone, and
`main` went red.** 2026-10-23. The batch of the 10-21 compute ranking landed
together. #953 (tests only), #959 (the fact audit becomes a CI gate), #970 (the
webhook fix) and #976 (the TL3 organic calendar) merged between 04:35:17Z and
04:36:55Z. On `main`, four tests failed. Three asserted that the event lock is
written exactly once, and #970 now writes it twice (`processing`, then `done`).
One was an `it.fails` documenting the bug #970 fixed, so it now passed, which
`it.fails` reports as a failure. Separately, #976's TL3-08 caption said "before
you've finished your first drink". That is in the free-drink pattern #959 had
just made a CI gate, so the gate failed on `main` (`4bb87f4f`, #979). Each PR's
CI ran against a `main` without the others.

**Cost:** `main` was red for 11 minutes, until #979 merged at 04:48:06Z. TL3-08
was `pending`, so the caption never posted. **Anchor:** `4bb87f4f` (#979);
merge times of #953, #959, #970 and #976 from `gh pr view`. **Detection:**
`THREW` — CI failed on `main`. It is not a `GATE`, because the gate ran after
the merge and refused nothing.

---

## DECISION — recurrences, what changed, what's still open

**Recurrences.** Each one names the countermeasure it defeated.

- **Incident 70 — customer names in tracked text, twice more.** The HANDOFF
  audit applied on 10-26 removed guest first names from three entries
  (`9c681e85`, #1002). On 10-23, `5dbd0749` (#982) put a guest's full name, her
  +1's name and the +1's email address in a commit message on `origin/main`,
  where no later scrub can reach without rewriting history. **Defeated:** #788's
  scrub, which checks files and not commit messages, and has no check on new
  writes.
- **Incident 69's open question answered.** The previous edition could not say
  whether `pr-reviewer` was registered. It is not: the 10-24 review "skipped PR
  #986 because REVIEW_PROMPT.md asked for a subagent type that has never loaded
  in headless --print" (`7baa7012`, #989). The review now uses the Explore agent
  with `pr-reviewer.md` pasted in. **Defeated:** nothing checked that the
  review's own reviewer existed. A skipped review printed `Run complete`.

**Not numbered, again: the OAuth outage**, which ended 10-23 after the
operator's `claude auth login`. The ruling stays: an expired platform login is
infrastructure with no agent in the chain. The same ruling covers the scheduled
tasks' phone alerts, which did not reach the phone ("Mobile push not sent
(Remote Control inactive)", run transcripts 10-27 11:57 and 10-28 20:04,
`ROUTINES_HEALTH_2026-10-28.md` §5). The tasks reported it in their transcripts
honestly. The cause was an app setting, turned on 10-28.

**Open item 82, explained.** The previous edition did not know why the social
check logged 3, 2 and 4 of seven slots on 10-13, 10-15 and 10-16. The laptop is
a Modern Standby machine, and its 10-minute screen-off *is* the standby. Claude
desktop tasks have no wake (memory `claude-desktop-tasks-skip-in-modern-standby`,
`ROUTINES_HEALTH_2026-10-28.md` §5). It is infrastructure, not an incident.

**Built in response, during the window.** As in every previous edition, each
was built after its failure.

- **Auth pre-flight, STATUS.json, one issue per failure and a brief block**
  (#955, `6dcc17d0`). This closes the gap the previous edition named, where the
  launcher matched only `Not logged in` and no session read the logs.
- **`check-nightly-report.js`**, a deterministic claim gate on each nightly
  report (#960).
- **The fact audit as a CI regression gate** (#959). Its first catch is in 100.
- **`npm run due` with OVERDUE and FOLLOW-UPS OWED** (#1007, #1023), emailed
  every morning, plus the CLAUDE.md rule that a line is cleared only with
  evidence. Closes 99's mechanism for dated items. A plan written only into a
  report is still read by nothing.
- **`render.js` refuses any `pulled_images` photo** (#1024, outside git), and
  the motion-ads rulebook refuses a frame showing one (#930). Closes 97 for
  that renderer. A sweep that greps file names would still miss a card whose
  name carries no IMG number.
- **The nightly fast-forwards the main checkout** (#1025). It is a fix for
  memory `nightly-pulls-from-stale-main-checkout`, not for anything numbered
  here.
- **The HANDOFF removal gate** (#962).

**Still open, as of `origin/main` on 2026-11-01:**

- **49** — `lib/ga4-mp.js:71` still says the shared id "lets GA4 count one
  transaction when both copies arrive."
- **70** — `5dbd0749`'s commit message, above.
- **87** — merged with the ranking (#943): `reports/COMPUTE_PRIORITIES_2026-10-21.md:106`
  still says "a live worktree holds the edition."
- **94** — the one-off 45-day spend sync that repairs Firestore has not been
  recorded as run (`HANDOFF.md:1038-1050`).
- **100's shape** — no PR's CI runs against the other PRs of its batch. A merge
  queue, or merging in sequence with a re-run between merges, would have made
  100 a `GATE`.

---

## What I did not verify

- **Live systems.** No Meta, GA4, Stripe, Firestore, Eventbrite, Gmail, X or
  Facebook call was made. I read git, PR bodies, GitHub run lists, the run logs
  and the reports. Times and counts come from commits, PR metadata and reports
  as quoted.
- **Whether a host candidate saw the $88 figure, or decided on it** (95).
- **That neither scheduled copy of MC2-25 published before it was replaced**
  (97). The commits say X was edited in place and Facebook deleted before the
  Nov 1 12:30 slot. I did not read either platform.
- **Who found 97, and how.** The PR body states the mechanism, not the trigger.
- **Whether any TL3 partner shared less because the asks came late** (99).
- **Incidents 01–48 and 76–100 have not had a claim-by-claim audit.**

Looked at and left out, so a later pass need not rediscover them:

- **MC2's Eventbrite early bird was set to end at the start of Oct 27, a day
  early** (`9bd05279`, #948). Caught before the date. Nothing records who chose
  the date, so I could not show an agent was involved.
- **A Gmail reply sent through the draft route re-attached the quoted message's
  photos** (10-24, memory `gmail-connector-wraps-links-in-google-redirects`).
  This is the route the 10-18 note had called clean. The photos went back to
  their own sender. It is anchored only in memory and cost nothing, the same
  reason the previous pass left out the 10-18 case.
- **MC2's first Meta-era sale arrived with no attribution on its ticket**
  (`ed4e9961`, fixed `03eeaeb6`). A browser storage read came back empty. It is
  a product bug with no agent in the chain.
- **The HANDOFF audit removed 55 entries and corrected 28** (`9c681e85`).
  That is a lot of stale state, but no single entry in it is shown to be an
  agent's failure with a cost.
- **TikTok support's reply called a rejected promotion "approved, delivered,
  and completed"** (memory `tiktok-reach-collapsed-after-september`). It was
  wrong, but no agent is in the chain.

Standing, carried forward and updated:

- **One project, one operator, six months.** These incidents come from a single
  small business, not a survey. The frequencies are not a base rate for
  anything.
- **Selection bias runs in the obvious direction.** This catalogue holds the
  failures that were *noticed*. This pass's best case for it is 97: the sweep
  reported success, and the miss was found only because someone happened to be
  rendering the next batch of cards. Had it been found a day later, it would
  have been a published post, not a near miss. The month's automated share
  went up because the failures that CI can see increased, not because the
  business-facing ones got easier to see.
- **This document's error rate against the live systems has been measured six
  times and was never zero** (40, 51, 75, 90, 91, 92). No new error was found
  this pass. Nobody looked for one.
- **"Cost" means what was lost or nearly lost.** Where an incident was caught
  before costing anything (96, 97, 100), that is stated rather than counted as a
  loss.
- **Names, account identifiers and customer records are omitted throughout.**
