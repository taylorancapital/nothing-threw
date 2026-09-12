# Six of Forty-Eight

**On 11 September 2026, at two in the morning, an agent told me my business had stopped selling tickets.**

It had been running unattended for weeks. It pulled the analytics tables, read the ad platform's numbers, wrote its report, opened a pull request, and flagged itself as urgent: no purchase had completed anywhere on the site in ten days. It named the commit it believed had broken the checkout. It said revenue was leaking and that resolving it needed a live check it couldn't perform itself.

Two tickets had sold three days earlier. They were in the payments database, in the server logs, and in the ad platform's own pixel receipts. The agent had missed them because the analytics tool it trusted fires its purchase event only from the buyer's browser — so a buyer whose browser drops the tag is a real sale it can never see. Neither buyer came from an ad, so the ad platform's attribution couldn't see them either. "Confirmed independently by two sources" was two views of the same blind spot.

The supporting evidence was worse. The agent noted that checkout-starts kept climbing while purchases stayed frozen, which sounds like a smoking gun. It was comparing running totals between two snapshots. Running totals only ever go up. That sentence is what the comparison prints every time, whether or not anything is wrong.

Nothing errored. Nothing retried. The run exited zero.

---

## The number

I've been keeping a log of every way the agents running my business have failed, for five months. Not a bug list — a bug list is worthless, everyone has one. Each incident is classified by **how it was actually caught.**

Five categories. A deterministic check refused it (`GATE`). An actual error surfaced at the time (`THREW`). A person distrusted a number (`HUMAN`). Someone reported that data looked missing (`OPERATOR`). Or it turned up months later, during an unrelated dig (`LATER`).

Forty-eight incidents. Here's the distribution:

| Detection | Count |
|---|---:|
| `GATE` — a deterministic check refused it | 1 |
| `THREW` — an actual error surfaced | 5 |
| `HUMAN` — someone distrusted a number | 12 |
| `OPERATOR` — reported as "data went missing" | 1 |
| `LATER` — found by an unrelated dig | 29 |

**Six of forty-eight were caught by anything automated.** Twenty-nine were found by accident, weeks or months on, by someone looking for something else entirely.

The business is small and real: a live ad account, Stripe checkout, a Firestore back end, an events product with actual customers. The agents write code, run the nightly analysis, publish social content, adjust ad budgets. Real money moves through decisions they influence.

## The part I expected to be different

I assumed this ratio would improve. That's the whole reason the log records detection mode instead of just the failure — I wanted a number that would move as the countermeasures accumulated.

It hasn't. Across five months, a 2.7× growth in catalogued incidents, and every gate, test and check built in response, the automated share has stayed between **9% and 15%**. It went 11%, then 9%, then 15%, then 13%. The one apparent improvement was an artifact: a backfill pass that swept older months added two same-day syntax errors from June — exactly the loud, self-announcing kind — and lifted the numerator in a single stroke. Nothing had gotten better. The sample had changed shape.

Countermeasures *do* get built. A stale-link check became a real CI step the day after a published listing served a 404. A pricing bug got a regression test. Swallowed email rejections got a counter. Every one of them works.

None of them caught anything.

They're each named for the single incident that produced them, built afterward, one at a time. And the failures that dominate now aren't the kind any of them look at.

## Why the ratio is stuck

Here is the uncomfortable mechanism, and I think it generalises well past my situation.

Early agent failures are *code* failures. A syntax error, an unescaped apostrophe, a placeholder string where an ID should be, a missing index. These announce themselves. They throw on load. Your existing machinery — tests, CI, error monitoring, the page simply not rendering — is built for exactly this, and catches it in hours.

As the agents get better, they stop making those mistakes. What replaces them is **wrong conclusions drawn from correct data.**

An agent that reads the right tables, runs the right queries, and reaches a false conclusion produces output that is syntactically perfect, internally consistent, confidently worded, and completely wrong. There is nothing for a test to grab. The code is fine. The query is fine. The data is fine. The *inference* is wrong, and inference has no stack trace.

Of my forty-eight incidents, the ones with the largest realised cost are all this shape:

- A report read an ad platform's attributed conversions as if they were sales, concluded sales had stopped nine days earlier, and reached the main branch. Fourteen tickets had actually sold in that window. The platform sees roughly 11% of real sales; a complete join of spend against revenue already existed in the database and nobody had read it.
- A cross-gender ad-spend leak was measured at $241.81 by comparing lifetime spend against each ad set's *current* targeting — invalid for any ad set ever edited, and most had been. Re-measured against each edit's own timestamp: $180.56 of it predated the targeting lock entirely. The real figure was about a quarter of the first one.
- An analysis of alternative ad channels stated that a particular platform's ads had never run on the account. They'd run on every event for four months, at roughly a fifth of the cost-per-ticket of the channel I was being advised to keep.

Every one of those passed every check in the repository, because no check in any repository asks whether a conclusion is true.

## The log got one wrong too

In the September pass, I wrote that a scheduled-task fix was still unbuilt and that nobody had run the elevated shell needed for it.

It had been done three days earlier. I read it live off the task scheduler and found the flag already set. Worse, the evidence was sitting in two places I could have checked at the time: the handoff file recorded who set it and when, and the log directory showed eight consecutive successful nights where there had previously been eight missing out of twenty-three.

That error ran in the *pessimistic* direction. It under-counted a working countermeasure, on the one ratio this whole exercise exists to measure — which is the direction nobody reading a failure catalogue thinks to challenge.

It's incident 40 now. I'm not being noble about this; it's the single most useful entry in the log. It's the only time anyone has audited a claim in the catalogue against the live system, and the audit found an error. Forty-seven entries have not had that treatment. Whatever you conclude from my numbers, discount them accordingly — I do.

## What this doesn't mean

It doesn't mean the agents are bad. They're productive; the business runs on them. In the five days that produced fourteen of these incidents, the repository took more than eighty commits. Density here tracks how hard the system was worked, not how badly it went.

It also isn't a dataset. One business, one operator, five months. The frequencies are not a base rate for anything, and the selection bias runs in the obvious direction: a catalogue built mostly out of accidental discoveries is, by construction, missing everything nobody has tripped over yet. Lean on the mechanisms, never on my percentages.

## What I'd actually do about it

I don't have a tool to sell you. What's worked, in rough order of value:

**Classify by detection, not by failure.** This is the whole trick, and it costs nothing. Bug lists tell you what broke. Detection modes tell you whether you'd find out next time. If you can't fill in the column, that's the finding.

**Be ruthless about what counts as automated.** A failure caught because someone happened to read a diff is not a gate. Inflating that column is the one way this exercise can lie about precisely the thing it exists to measure.

**Ask what it would have taken for the check to fail.** If the answer is "different machine state," it verified nothing. One of my test suites passed for months because the dev machine had an API token that CI didn't.

**Treat "no data" as an unresolved question.** A large fraction of these are an error swallowed somewhere and read downstream as absence. Zero is a claim. Make something prove it.

**Write the correction in the file.** Every incident here that got retracted kept its retraction inline. A catalogue that silently revises itself is worth nothing, and the retractions turn out to be the most-read entries.

The pattern underneath all of it: for five months, my systems have been reliable at telling me when the *machinery* broke and nearly silent on whether the *conclusions* were true. Everything I've built has improved the first. I still have almost nothing for the second, and on current evidence that's where the expensive failures now live.

---

*The full catalogue — all 48 incidents, each anchored to a commit, a log line, a pull request, or an API read taken at the time — is at [github.com/taylorancapital/nothing-threw](https://github.com/taylorancapital/nothing-threw). It's updated monthly.*
