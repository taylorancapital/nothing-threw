# Nothing Threw

**Eighteen ways autonomous coding agents failed while running a live business —
and how each one was actually caught.**

*Field notes, May–September 2026. One events business: a live Meta ad account,
Stripe checkout, a Firestore back end, and a nightly analytics agent that writes
reports and opens pull requests unattended.*

Seventeen of the eighteen reported success. The interesting question turned out
not to be how they failed, but how anyone ever found out.

## How each of the 18 was detected

| Detection | Count | |
|---|---:|---|
| **Gate** — a deterministic check refused it | 1 | |
| **Threw** — an actual error surfaced | 1 | |
| **Human** — someone distrusted a number | 8 | |
| **Operator** — reported as "data went missing" | 1 | |
| **Later** — found by an unrelated dig, days to months on | 7 | |

**Two of eighteen were caught by the system itself.** The rest were caught
because a person looked at a number and thought *that can't be right* — or
because months later, something else went wrong and led back to it.

---

## A. The agent was confidently wrong

### 01 — Two ID spaces that never match · `HUMAN`

An analysis compared an ad creative's `video_id` against the `object_id`s in a
video-engagement audience. The first is the source upload; the second is the
delivered rendition — the platform's auto-generated crops. They can never match,
even for the same video.

**Cost:** a false headline that the retargeting audience contained none of the
running videos, an entire "the funnel was never wired up" conclusion built on
top of it, and a pull request. The audience was correctly scoped the whole time.

### 02 — A share of 105% · `LATER`

The insights API returns different conversion counts for the same window with
and without a gender breakdown. A report divided a breakdown numerator by an
un-broken denominator.

**Cost:** published "women's share of landing-page views: 105.00%" and nobody
noticed. The impossible figure was the visible tip; every other gender-split
cost in that report sat on the same mismatch and looked fine.

### 03 — A debrief that named the wrong cause · `LATER`

An event post-mortem reasoned from the code alone and blamed a two-for-one
ticket offer for a gender imbalance. The flag it depended on was false on every
record involved; the offer had played no part at all.

**Cost:** a confident, circulated, wrong explanation. Rewritten only after
someone queried the data directly — and the superseded reasoning was deleted
rather than kept, because a wrong cause left lying around is worse than none.

### 04 — A filtered first query · `HUMAN`

An investigation into an ad account opened by listing only `ACTIVE` campaigns.
Three dormant campaigns — the ones that had delivered the cheapest clicks the
account ever bought — were invisible to every subsequent step.

**Cost:** three successive proposals, each blocked by a different constraint,
each looking like progress. The real fix was flipping a status back. The cost
was not the wrong answer; it was four review cycles of someone else's time.

---

## B. The check passed for a reason unrelated to correctness

### 05 — Green locally, red in CI, and the difference was a token · `LATER`

A script called `main()` at module scope instead of guarding on
`require.main === module`. CI, with no API token, hit `process.exit(2)`
synchronously on import and the test process died. The dev machine *had* the
token, so the same import went to the network instead of exiting, and the
synchronous tests finished before the exit landed.

**Cost:** the suite passed for a reason that had nothing to do with the code
being right, and was reported as verified.

> When a check passes, ask what it would have taken to fail. If the answer is
> "different machine state," it verified nothing.

### 06 — Six sources agreed, and all six were wrong · `HUMAN`

An audit of the business's public claims found six surfaces describing the event
format identically. One social caption disagreed with all of them — and was the
only correct description anywhere in the codebase.

**Cost:** months of marketing copy describing the product incorrectly. A fact on
six surfaces has six chances to be wrong and one chance to be noticed.

### 07 — The test command that never exits · `HUMAN`

The package's `test` script was bare `vitest` — watch mode. Run unattended it
produces an empty output file and holds until the timeout.

**Cost:** about seven minutes, burned looking like a hang rather than a
misconfiguration. A clean example of a failure whose symptom points nowhere near
its cause.

---

## C. Silent failure — the error was swallowed and read as absence of data

### 08 — Security rules that were never deployed · `LATER`

Nothing in CI deploys the Firestore ruleset; it is a manual command. A
collection shipped without a matching rule, so the dashboard's read was denied —
and the codebase's deliberate fail-soft handlers swallowed the permission error
into a `console.warn`.

**Cost:** the dashboard reported a materially wrong P&L for as long as the rules
sat undeployed, against a collection holding 73 real documents. Deploying moved
cost $260.00 → $527.38, net revenue $628.44 → $434.68, blended CAC
$12.24 → $15.36.

### 09 — A missing composite index inside a live listener · `OPERATOR`

A query combined `where()` with `orderBy()` on a different field, requiring a
composite index that did not exist. Every snapshot threw "the query requires an
index" straight into the listener's own fail-soft catch.

**Cost:** the widget read "No ticket sales yet" from the moment it shipped.
Reported by the operator as data going missing, not as an error — because in the
UI a permission failure and "nobody has entered anything" are indistinguishable.

### 10 — A nightly that simply did not run · `LATER`

Two flags on the scheduled task — start-when-available and
run-only-if-network-available — were both false. A machine asleep at the trigger
time, or online but without a network yet, produced no run and no notice.

**Cost:** 8 of 23 nights lost. One run failed both network-dependent steps
fourteen minutes after a boot. The gap was only visible by counting log files
against a calendar.

### 11 — A finished report with no pull request · `LATER`

The automation pushed an analysis branch, then failed at the step that opens the
PR — a multi-line body passed as a command-line argument, mangled by the shell's
own re-quoting into stray arguments.

**Cost:** a completed report stranded for a day, invisible because a branch with
no PR looks like nothing at all. Fixed by writing the body to a file and passing
the path.

### 12 — A data pull that silently overwrote the one before it · `HUMAN`

Nightly export files are named by table and date, not by pull time. Two pulls on
the same day collide and the second wins.

**Cost:** nearly destroyed the exact dataset a published report cited. The loss
would have been invisible — the report keeps rendering perfectly, against
different numbers.

---

## D. Destructive writes that the API reports as success

### 13 — Replacing an array that should have been appended to · `GATE`

Attaching a tracking pixel to two live ads. The natural implementation copies
the shape used at ad creation, which sets the tracking array to the pixel entry
alone — correct at birth, when there is nothing to preserve. Those live ads
already carried **seven** entries, including the one every landing-page-view
number came from.

**Cost:** none, because it was caught first — but the platform returns success
either way, and the damage would have shown up as metrics quietly going to zero.
The tool now dry-runs by default, appends, reads the field back rather than
trusting the write, and warns if the count shrank.

### 14 — A config loader that clobbered a working credential · `HUMAN`

The checked-in env file holds two-character placeholders; the real 200-character
tokens live in the shell environment. A one-off script loading that file the
usual way overwrote a good token with an empty one.

**Cost:** a failure that presents as `Provide valid app ID` — which reads like a
scopes or app-registration problem and sends you to the token debugger for
nothing. The fix is one conditional: only set a variable if it is not already
set.

---

## E. Concurrency — several agents, one working tree

### 15 — A pushed commit, overwritten · `LATER`

Two agent sessions shared one checkout. One pushed work to a branch; the other
reused the same branch name and force-moved it.

**Cost:** the commit survived unreferenced and had to be recovered from the
reflog. Recovery, not prevention — and only because someone went looking.

### 16 — A commit that landed on someone else's branch · `HUMAN`

Work intended for one branch was committed onto another session's, which then
committed on top of it.

**Cost:** untangling took a rebase that dropped a commit from the middle. One
recovery created the next problem.

### 17 — A branch that could not be created · `THREW`

`git checkout -b` failed outright: the shared tree was mid-merge, with a
conflict in a file belonging to nobody in that session.

**Cost:** minutes. Listed because it is the **only incident in this document
that produced an immediate, honest error** — one of eighteen.

### 18 — A pull request carrying a stale copy of someone else's file · `HUMAN`

A session swept an unrelated in-progress file out of the shared working tree
into its own commit.

**Cost:** caught in review. Had it merged second, it would have **silently
reverted a fix already verified against the live API** — a regression with a
green diff, introduced by a PR about something else entirely.

---

## What was built in response

**1. Put the constraint in a gate, not in the prompt.** The nightly agent is
launched with push and the GitHub CLI *removed from its tool set*, so it cannot
reach GitHub at all. It writes a report and commits. A deterministic script then
inspects what it produced — exactly one commit ahead of main, exactly one
changed file, matching a path regex, working tree clean — and refuses to push
anything else, whatever the prompt said.

```
# Report-only is enforced here, not in the prompt.
```

**2. Give every session its own working tree.** All four concurrency failures
came from agents sharing one checkout. Each session now gets an isolated
worktree branched fresh from the remote main, so no session can inherit
another's half-finished state or reuse its branch name.

**3. Read back after every write; never trust the success response.** The tool
that attaches a pixel is dry-run by default, appends rather than replaces,
re-reads the field from the API instead of believing the POST, and warns if the
number of entries went down.

**4. Build one check that reads more than one source.** Every existing check
read exactly one surface. One audit now cross-checks every public factual claim
across 26 surfaces against a named canonical value. It is deliberately *not* in
CI: several of its checks report things that are correct-but-worth-knowing, so
it would sit permanently red and train everyone to ignore it.

**5. Make reports declare what they did not verify.** Analyses label each
section by epistemic status — measured, inferred, or not verified — and every
one ends with a mandatory "what I did not verify" section. A second scheduled
agent, with edit, commit and merge removed from its tools, fact-checks the first
one's output and comments on the pull request.

---

## Six rules that would have caught most of this

1. **Put the constraint where the model cannot reach it.** An instruction in a
   prompt is a request. A check that runs after the model finishes, on the
   artifact it produced, is a guarantee. Only one of the two survives a model
   that is confidently wrong.
2. **When a check passes, ask what it would have taken to fail.** If the answer
   is "different machine state," it verified nothing.
3. **Agreement between sources is not evidence.** It proves the sources agree.
   Six surfaces agreed here and all six were wrong; the lone dissenting one was
   correct.
4. **Read back after every write to a live system.** Success responses are
   cheap. Two of these incidents were destructive operations the API cheerfully
   confirmed.
5. **Treat "it went empty with no error" as a first-class symptom.** Fail-soft
   error handling is right for resilience and catastrophic for diagnosis. A
   denied read and an empty collection are indistinguishable downstream.
6. **Enumerate the whole state before forming a hypothesis.** One filtered
   opening query produced three wrong proposals in sequence, each of which
   looked like progress. Paused and dormant objects are part of the picture.

---

## What this is and is not

- **One project, one operator, four months.** These are incidents from a single
  small business, not a survey. The frequencies here are not a base rate for
  anything.
- **Selection bias runs in the obvious direction.** This catalogue contains the
  failures that were eventually noticed. Failures still undetected are, by
  construction, absent — and given that 16 of the 18 were found by something
  other than an automated check, assuming the list is complete would be the same
  error it documents.
- **"Cost" means what was lost or nearly lost**, established from logs, commits
  and API reads at the time. Where an incident was caught before doing damage,
  that is stated rather than counted as a loss.
- **Not a claim that the agents were unusually bad.** Most of these are ordinary
  distributed-systems and API-integration failures. What is specific to agents
  is the *rate* at which confidently-wrong output gets produced and the ease
  with which it reaches a pull request.
- **Names, account identifiers and customer records are omitted throughout.**
