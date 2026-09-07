Nothing Threw
Thirty-four ways autonomous coding agents failed while running a live business — and how each one was actually caught.

Field notes, May–September 2026. One events business: a live Meta ad account, Stripe checkout, a Firestore back end, and a nightly analytics agent that writes reports and opens pull requests unattended.

Twenty-nine of the thirty-four reported success. The interesting question turned out not to be how they failed, but how anyone ever found out — and, five months in, whether that's getting any easier. It isn't yet.

How each of the 34 was detected
Detection	Count	
Gate — a deterministic check refused it	1	
Threw — an actual error surfaced	4	
Human — someone distrusted a number	11	
Operator — reported as "data went missing"	1	
Later — found by an unrelated dig, days to months on	17	
Five of thirty-four were caught by the system itself — 15%. The rest were caught because a person looked at a number and thought that can't be right — or because months later, something else went wrong and led back to it. That share was never really climbing: a small early sample carried two loud, same-day code bugs that happen to announce themselves the moment anyone loads the page. As the incidents get subtler — wrong conclusions from correct data, not broken code — the share any existing check could plausibly catch keeps falling as volume rises.

A. The agent was confidently wrong
01 — Two ID spaces that never match · HUMAN
An analysis compared an ad creative's video_id against the object_ids in a video-engagement audience. The first is the source upload; the second is the delivered rendition — the platform's auto-generated crops. They can never match, even for the same video.

Cost: a false headline that the retargeting audience contained none of the running videos, an entire "the funnel was never wired up" conclusion built on top of it, and a pull request. The audience was correctly scoped the whole time.

02 — A share of 105% · LATER
The insights API returns different conversion counts for the same window with and without a gender breakdown. A report divided a breakdown numerator by an un-broken denominator.

Cost: published "women's share of landing-page views: 105.00%" and nobody noticed. The impossible figure was the visible tip; every other gender-split cost in that report sat on the same mismatch and looked fine.

03 — A debrief that named the wrong cause · LATER
An event post-mortem reasoned from the code alone and blamed a two-for-one ticket offer for a gender imbalance. The flag it depended on was false on every record involved; the offer had played no part at all.

Cost: a confident, circulated, wrong explanation. Rewritten only after someone queried the data directly — and the superseded reasoning was deleted rather than kept, because a wrong cause left lying around is worse than none.

04 — A filtered first query · HUMAN
An investigation into an ad account opened by listing only ACTIVE campaigns. Three dormant campaigns — the ones that had delivered the cheapest clicks the account ever bought — were invisible to every subsequent step.

Cost: three successive proposals, each blocked by a different constraint, each looking like progress. The real fix was flipping a status back. The cost was not the wrong answer; it was four review cycles of someone else's time.

23 — A pricing refactor that touched half of a two-sided contract · HUMAN
A commit removing gendered ticket pricing updated the admin event-creation form and the on-page price display to a single spots/price model. It did not touch the payment path, which still read the old per-gender fields — undefined on any new-model event. Every purchase attempt on a new event was rejected as sold out; had one somehow passed, price would have resolved to $0, charging only the flat service fee.

Cost: none realised. The live payment key wasn't switched in until the day after the fix — no real card could have reached this path. Fixed with a single source of truth for both the admin and payment sides.

24 — Two checkout gaps shipped with the original build, found five days after go-live · HUMAN
A guest paying with a 3-D Secure card got a confirmed ticket but no welcome email, no trial signup, no nurture lead — the entire post-purchase funnel silently skipped anyone who hit that challenge, because the enrollment calls ran before the challenge could return. Separately, a duplicate submit could bump the seat counter twice before the retry was recognised as a repeat.

Cost: the 3DS gap was live with real payments for five days — an unknown number of real guests never got their onboarding funnel. The duplicate-submit gap's worst case, per the fix: "an over-counted seat, never an oversell or double charge."

25 — A placeholder ad-conversion tag broke analytics sitewide for about four hours · THREW
A conversion-tracking install shipped with a literal placeholder string instead of a real tracking ID — not a valid tag, and it broke the site's analytics initialization on every page that carried it.

Cost: the only telemetry the business had, eight days after go-live, was dark sitewide for roughly four hours. Fixed the same day by removing the tag rather than supplying a real ID.

26 — A fatal syntax error shipped with a new set of pages, same day as the pages themselves · THREW
Two strings used a quote style that broke on an apostrophe inside them — a fatal syntax error that broke the entire script on two new city pages. No events loaded; the pages fell back to static, hardcoded copy for the wrong city. A second, independent defect in the same file tagged fallback content with the wrong city in the page's own structured data, handed directly to search engines.

Cost: near zero — fixed the same day it shipped, consistent with a syntax error that fails immediately and visibly on load.

27 — Filename and modification time were never a valid staleness check · LATER
A nightly analysis pipeline's rotation logic skips re-analyzing an export it believes is unchanged from the night before — decided, for three consecutive cycles, by matching filenames and file modification times. Both are always identical across pulls by construction; the underlying data had in fact been deleted and freshly re-pulled. Three consecutive nightly cycles treated genuinely new data as a stale duplicate and skipped analyzing it.

Cost: at least two full nightly cycles where fresh exported data was never actually analyzed. Caught the same night by reading each file's own embedded date range instead of trusting its name.

19 — 245 GA4 fields declared empty without probing them · LATER
An audit of the analytics property's coverage concluded that 245 fields were "structurally empty because every ad we run is on Meta," written from category names rather than from probing the fields. Wrong three separate ways: one field family turned out to be real and populated; two others returned constant placeholder values on every row, which looked like data but weren't; and most of the true remainder returned a different, unrelated "not set" value with a different fix.

Cost: weeks of live ad spend on another platform stayed unread because a neighboring probe (incident 20) concluded it was unmeasurable. The durable cost is the method: triaging an API's fields by category name produces confident, checkable, wrong claims.

20 — A capability probe that tested metrics in isolation · LATER
A metrics family errored when queried alone, asking for a companion dimension to be added. Read in isolation, that error means "unavailable." It means "needs a pairing."

Cost: weeks of real ad spend invisible to every report written before the fix — some of which bought real clicks and zero attributed sessions, and landed in neither of the business's two cost-tracking paths. Every efficiency figure in every prior report was understated by that amount.

21 — Correlation presented as causation · HUMAN
A live read of every campaign found one objective holding all of one week's purchases on similar spend to another objective holding zero — nine times the visitors, no sales — and it was presented as proof that the second objective does not sell.

Tested per dollar, it was not statistically significant. With six lifetime conversions on the account, no causal claim in either direction was supportable.

Cost: none realised. Someone asked whether it was actually causal; the finding was corrected the same day.

29 — A wrong headline reached the branch that runs the business, and had to be retracted the same evening · LATER
A delivery diagnosis read the ad platform's attributed conversions as sales and concluded the business's sales had stopped for over a week. They had not — in the exact window the report called dead, more than a dozen tickets actually sold. The platform sees roughly one in nine real ticket sales; a complete revenue join already existed in the business's own database and nobody had read it. Read correctly, gross return on ad spend for the period was better than break-even, not the fraction of a dollar the retracted report implied.

Cost: a wrong "the business is dying" headline reached the branch running the business, retracted the same evening. No live change was made on the strength of the wrong number before the retraction — the near-miss is real regardless.

30 — Two more attribution bugs, found in the same evening's dig · LATER
Investigating the wrong headline above turned up two independent, previously undetected defects in the same attribution pipeline. First: a cookie-reading pattern was written as a plain string rather than a regular expression, so a single stray character silently made the whole pattern unable to match unless the target cookie happened to come first — meaning the platform's own click-tracking cookies went unread for most buyers, and an earlier "only one buyer's cookie was ever set" claim was this bug, not a real signal. Second: first-touch attribution never expired, so a visitor's oldest recorded touch — months old, from an entirely different channel — could keep "winning" credit over any number of genuinely recent ad clicks.

Cost: an unknown span of degraded ad-platform match quality, and an unknown number of real ad-driven purchases mis-credited to the wrong channel — both found only because the investigation into incident 29 went adversarial on its own pipeline instead of stopping at the first correction.

31 — A leak recalculated at a quarter of its first estimate · LATER
A cross-gender ad-delivery leak was first measured by comparing lifetime spend against each ad set's current targeting settings. That comparison is invalid for any ad set that has ever been edited — most had. Re-measured against each edit's own timestamp, the great majority of the apparent leak turned out to predate the targeting restriction even existing. The real leak was about a quarter of the first estimate.

Cost: a headline dollar figure overstated roughly four times over, caught before any live change was made on it.

32 — A parity audit assumed the wrong side was the reference · LATER
Three near-identical checkouts exist because nothing shares a code path between them. An audit treated one of the three as automatically correct and framed every finding as "bring the others in line with it." In closeout, four confirmed defects turned out to be in the assumed-correct file — including a guard against a customer being charged twice — and the file nobody treated as canonical had every one of them right.

Cost: a near-miss. Applied in the direction the audit assumed, real safety guards — including the double-charge guard — would have been stripped from the file that had them working, in the name of "fixing" it to match a file that did not. Caught during the closeout itself, before anything shipped in the wrong direction.

B. The check passed for a reason unrelated to correctness
05 — Green locally, red in CI, and the difference was a token · LATER
A script called main() at module scope instead of guarding on require.main === module. CI, with no API token, hit process.exit(2) synchronously on import and the test process died. The dev machine had the token, so the same import went to the network instead of exiting, and the synchronous tests finished before the exit landed.

Cost: the suite passed for a reason that had nothing to do with the code being right, and was reported as verified.

When a check passes, ask what it would have taken to fail. If the answer is "different machine state," it verified nothing.

06 — Six sources agreed, and all six were wrong · HUMAN
An audit of the business's public claims found six surfaces describing the event format identically. One social caption disagreed with all of them — and was the only correct description anywhere in the codebase.

Cost: months of marketing copy describing the product incorrectly. A fact on six surfaces has six chances to be wrong and one chance to be noticed.

07 — The test command that never exits · HUMAN
The package's test script was bare vitest — watch mode. Run unattended it produces an empty output file and holds until the timeout.

Cost: about seven minutes, burned looking like a hang rather than a misconfiguration. A clean example of a failure whose symptom points nowhere near its cause.

34 — A verification pass that never ran, read as a pass · LATER
A multi-agent workflow ran three proposals and a set of agents tasked with verifying each one. Most of the verifiers hit a resource limit mid-run and never executed — including every mechanics check and the final synthesis step. Two of the three proposals came back from the workflow marked "no objections, survives review." Not because they passed. Because nobody checked. One quiet field in the workflow's own usage report was the only thing that said so; nothing else distinguished "reviewed and clean" from "never reviewed."

Cost: none realised — caught by reading that field before trusting the "survives" verdict, and the surviving proposal was checked by hand afterward. The pattern is the one this whole document is about: an empty list of objections reads exactly like a pass, and here it was a pass with no examiner in the room.

C. Silent failure — the error was swallowed and read as absence of data
08 — Security rules that were never deployed · LATER
Nothing in CI deploys the Firestore ruleset; it is a manual command. A collection shipped without a matching rule, so the dashboard's read was denied — and the codebase's deliberate fail-soft handlers swallowed the permission error into a console.warn.

Cost: the dashboard reported a materially wrong P&L for as long as the rules sat undeployed, against a collection holding 73 real documents. Deploying moved cost $260.00 → $527.38, net revenue $628.44 → $434.68, blended CAC $12.24 → $15.36.

09 — A missing composite index inside a live listener · OPERATOR
A query combined where() with orderBy() on a different field, requiring a composite index that did not exist. Every snapshot threw "the query requires an index" straight into the listener's own fail-soft catch.

Cost: the widget read "No ticket sales yet" from the moment it shipped. Reported by the operator as data going missing, not as an error — because in the UI a permission failure and "nobody has entered anything" are indistinguishable.

10 — A nightly that simply did not run · LATER
Two flags on the scheduled task — start-when-available and run-only-if-network-available — were both false. A machine asleep at the trigger time, or online but without a network yet, produced no run and no notice.

Cost: 8 of 23 nights lost. One run failed both network-dependent steps fourteen minutes after a boot. The gap was only visible by counting log files against a calendar. Recurred the following month — the exact same gap, on a night that a naive log check would have missed because an unrelated task's log existed for the same date. Neither flag has been fixed; both need an elevated shell nobody has run yet.

11 — A finished report with no pull request · LATER
The automation pushed an analysis branch, then failed at the step that opens the PR — a multi-line body passed as a command-line argument, mangled by the shell's own re-quoting into stray arguments.

Cost: a completed report stranded for a day, invisible because a branch with no PR looks like nothing at all. Fixed by writing the body to a file and passing the path.

12 — A data pull that silently overwrote the one before it · HUMAN
Nightly export files are named by table and date, not by pull time. Two pulls on the same day collide and the second wins.

Cost: nearly destroyed the exact dataset a published report cited. The loss would have been invisible — the report keeps rendering perfectly, against different numbers.

22 — The nightly's two halves run different versions of the code · LATER
A scheduled pipeline pulls fresh data in one stage and runs its analysis, from a separately checked-out copy of the code, in a later stage. The two stages can silently disagree: a merged change that widened what the pipeline pulls did not change what a run minutes later actually pulled, because the stage that fetches data was still running from a copy of the repository that hadn't been updated that session. The resulting report was written by current analysis code against stale data. The run exited clean and opened a pull request; nothing in the log said the inputs were old, because there was no line written for it to say.

Cost: one full nightly report built on data a merged change should have replaced. Not fixed — recurs after every merged change to the part of the pipeline that pulls data, and has already resurfaced once through a second, unrelated mechanism with the identical root cause: an interactive tool serving a stale, months-old version of one of its own instruction files, for the exact same reason (a checked-out copy of the code that hadn't been refreshed).

D. Destructive, or unreported, writes that report as fine
13 — Replacing an array that should have been appended to · GATE
Attaching a tracking pixel to two live ads. The natural implementation copies the shape used at ad creation, which sets the tracking array to the pixel entry alone — correct at birth, when there is nothing to preserve. Those live ads already carried seven entries, including the one every landing-page-view number came from.

Cost: none, because it was caught first — but the platform returns success either way, and the damage would have shown up as metrics quietly going to zero. The tool now dry-runs by default, appends, reads the field back rather than trusting the write, and warns if the count shrank. The only incident in this whole document caught by an actual deterministic gate.

14 — A config loader that clobbered a working credential · HUMAN
The checked-in env file holds two-character placeholders; the real 200-character tokens live in the shell environment. A one-off script loading that file the usual way overwrote a good token with an empty one.

Cost: a failure that presents as Provide valid app ID — which reads like a scopes or app-registration problem and sends you to the token debugger for nothing. The fix is one conditional: only set a variable if it is not already set.

33 — A workflow wrote to a shared file when it was only asked to return a proposal · LATER
Three parallel agents were asked to design and return a competing configuration proposal as structured data. One of them additionally wrote its proposal directly into a live shared file — uncommitted, unmentioned in the workflow's own result. It surfaced only because a later, completely unrelated edit failed on an anchor point that had quietly moved underneath it.

Cost: none realised — the unrequested write turned out to be good work and was kept after review, and nothing else was touched. That it caused no damage this time was luck, not design: nothing distinguishes a helpful unrequested write from a harmful one except checking the working tree immediately after every run, before trusting it.

E. Concurrency — several agents, one working tree
All four of the following happened on a single day, from two agent sessions sharing one checkout.

15 — A pushed commit, overwritten · LATER
Two agent sessions shared one checkout. One pushed work to a branch; the other reused the same branch name and force-moved it.

Cost: the commit survived unreferenced and had to be recovered from the reflog. Recovery, not prevention — and only because someone went looking.

16 — A commit that landed on someone else's branch · HUMAN
Work intended for one branch was committed onto another session's, which then committed on top of it.

Cost: untangling took a rebase that dropped a commit from the middle. One recovery created the next problem.

17 — A branch that could not be created · THREW
git checkout -b failed outright: the shared tree was mid-merge, with a conflict in a file belonging to nobody in that session.

Cost: minutes. Listed because it is one of only four incidents in this whole document that produced an immediate, honest error.

18 — A pull request carrying a stale copy of someone else's file · HUMAN
A session swept an unrelated in-progress file out of the shared working tree into its own commit.

Cost: caught in review. Had it merged second, it would have silently reverted a fix already verified against the live API — a regression with a green diff, introduced by a PR about something else entirely.

F. Correctly diagnosed, not fixed
New this edition. Doesn't fit A–E: the agent was not wrong, nothing was silent, no write occurred, and this was not a concurrency collision. The failure is that an accurate, honestly-reported blocker recurred for days because reporting it once is not the same as escalating it.

28 — Three consecutive nights with no way to open a pull request, each one correctly reported and none of them fixed · THREW
An earlier version of the nightly automation lost its access to push code for three consecutive scheduled runs, because its own environment stopped exposing the credential it needed. Per its own rule, it did not improvise a workaround — it wrote each night's finished report to disk instead of opening a pull request, and stated the exact cause and the exact fix needed, correctly, every single time. The two nights before it had already said the same thing.

Cost: three-plus nights of finished analysis stranded outside version control, invisible to anyone not looking in the right folder directly. Recovered two days after the third occurrence. The same shape of failure — a finished report, no pull request — recurred later via a completely different mechanism, on a completely different automation stack, before either root cause was actually addressed.

What was built in response
1. Put the constraint in a gate, not in the prompt. The nightly agent is launched with push and the GitHub CLI removed from its tool set, so it cannot reach GitHub at all. It writes a report and commits. A deterministic script then inspects what it produced — exactly one commit ahead of main, exactly one changed file, matching a path regex, working tree clean — and refuses to push anything else, whatever the prompt said.

# Report-only is enforced here, not in the prompt.
2. Give every session its own working tree. All four concurrency failures came from agents sharing one checkout. Each session now gets an isolated worktree branched fresh from the remote main, so no session can inherit another's half-finished state or reuse its branch name.

3. Read back after every write; never trust the success response. The tool that attaches a pixel is dry-run by default, appends rather than replaces, re-reads the field from the API instead of believing the POST, and warns if the number of entries went down.

4. Build one check that reads more than one source. Every existing check read exactly one surface. One audit now cross-checks every public factual claim across 26 surfaces against a named canonical value. It is deliberately not in CI: several of its checks report things that are correct-but-worth-knowing, so it would sit permanently red and train everyone to ignore it.

5. Make reports declare what they did not verify. Analyses label each section by epistemic status — measured, inferred, or not verified — and every one ends with a mandatory "what I did not verify" section. A second scheduled agent, with edit, commit and merge removed from its tools, fact-checks the first one's output and comments on the pull request.

What hasn't been built. Nothing yet catches a wrong conclusion drawn from correct data — which is now the dominant failure mode (incidents 01, 02, 19, 20, 21, 29, 30, 31, 32). Nothing yet turns a three-times-repeated, honestly reported blocker (incident 23) into an escalation instead of a fourth identical report. And the nightly's two-stage staleness problem (incident 22) has recurred once already through a second mechanism since being found. The gate checks the shape of an agent's output, not whether it's true — that gap is now the largest one left.

Seven rules that would have caught most of this
Put the constraint where the model cannot reach it. An instruction in a prompt is a request. A check that runs after the model finishes, on the artifact it produced, is a guarantee. Only one of the two survives a model that is confidently wrong.
When a check passes, ask what it would have taken to fail. If the answer is "different machine state," it verified nothing.
Agreement between sources is not evidence. It proves the sources agree. Six surfaces agreed here and all six were wrong; the lone dissenting one was correct.
Read back after every write to a live system. Success responses are cheap. Several of these incidents were destructive, or silently unreported, operations that reported back as fine.
Treat "it went empty with no error" as a first-class symptom. Fail-soft error handling is right for resilience and catastrophic for diagnosis. A denied read and an empty collection are indistinguishable downstream.
Enumerate the whole state before forming a hypothesis. One filtered opening query produced three wrong proposals in sequence, each of which looked like progress. Paused and dormant objects are part of the picture.
Reporting a blocker once is not the same as escalating it. An automation that hits the identical wall three nights running and says so accurately every time has still failed, if nobody with the power to fix it ever hears about it as anything other than last night's log line.
What this is and is not
One project, one operator, five months. These are incidents from a single small business, not a survey. The frequencies here are not a base rate for anything.
Selection bias runs in the obvious direction, and it gets sharper the further back the record goes. This catalogue contains the failures that were eventually noticed. The month a live dashboard, daily automation, and concurrent agent sessions all existed at once produced eleven incidents; the three months before it — pre-launch, first-week-live, and the automation's own first month — produced six between them, not because those months were cleaner, but because less was live yet to leave a mistake worth noticing.
"Cost" means what was lost or nearly lost, established from logs, commits and API reads at the time. Where an incident was caught before doing damage, that is stated rather than counted as a loss.
Not a claim that the agents were unusually bad. Most of these are ordinary distributed-systems and API-integration failures. What is specific to agents is the rate at which confidently-wrong output gets produced and the ease with which it reaches a pull request — or, in one case, the branch that runs the business itself.
Names, account identifiers and customer records are omitted throughout.
