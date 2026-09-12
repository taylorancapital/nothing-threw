# The detection-mode method

A way of keeping a failure log for autonomous agents that answers a question a
bug list can't: **would you find out next time?**

This is the method behind [the catalogue in this repo](README.md). It is
deliberately small. There is no tool, no dependency, and nothing to install —
it is a classification discipline and a monthly habit. Copy it, change it,
argue with it.

---

## The one rule that makes it worth doing

**Classify every incident twice: by what went wrong, and by how it was actually
caught.**

Everyone records the first. Almost nobody records the second, and the second is
where the information is. "We had 40 incidents last quarter" tells you the
volume of your mistakes. "We had 40 incidents and 3 were caught by anything
automated" tells you the volume of your *blindness*, which is the number that
should drive what you build next.

If you can't fill in the detection column for an incident, that is itself the
finding.

---

## Detection modes

Exactly one per incident. Resist the urge to add more; the value is in the
coarseness.

| Mode | Means |
|---|---|
| `GATE` | A deterministic check refused it. A test failed, a linter blocked it, a guard rejected the write. |
| `THREW` | An actual error surfaced at the time — a stack trace, a non-zero exit, a failed operation. |
| `HUMAN` | Someone distrusted a number or a claim and went looking. |
| `OPERATOR` | Reported from outside as "the data went missing" or "this looks wrong." |
| `LATER` | Found during an unrelated dig, days to months on. |

**`GATE` is the number that matters, so be ruthless about it.** A failure caught
because somebody happened to read the diff is `HUMAN`, not `GATE`. A failure
caught because a code review noticed it is `HUMAN`. A failure caught because a
test you wrote *after* the incident would now catch it is not `GATE` either —
that check did not exist when it mattered.

Inflating this column is the single way the method can lie about exactly the
thing it exists to measure. When in doubt, classify down.

**`GATE` + `THREW` is your automated share.** Track it over time. It is the only
evidence that anything you build in response is working.

---

## Failure classes

Five, plus room for a sixth when something genuinely doesn't fit.

| Class | Means |
|---|---|
| **A** | The agent was confidently wrong |
| **B** | The check passed for a reason unrelated to correctness |
| **C** | Silent failure — the error was swallowed and read as absence of data |
| **D** | A destructive write the API reported as success |
| **E** | Concurrency — several agents, one working tree |

Class **A** will dominate as your agents improve. That is the finding, not a
problem with the taxonomy: code errors announce themselves and get fixed, and
what remains is wrong conclusions drawn from correct data, which have no stack
trace and nothing for a test to grab.

Add a class only when something truly doesn't fit — this catalogue added one
(**F**, correctly diagnosed and never fixed) exactly once in five months.

---

## The bar for inclusion

An incident goes in only if **all four** hold. This matters more than it looks:
a padded log destroys the value of the honest entries, and the whole credibility
of the exercise is that every claim is checkable.

1. **It is anchored.** A commit SHA, a log line with its date, a file path with
   a line number, or an API read taken at the time. *No anchor, no entry.*
2. **Something was actually wrong.** Not a design decision you disagree with,
   not a TODO, not a known and accepted limitation.
3. **An agent's behaviour is part of the causal chain.** Ordinary product bugs
   go somewhere else. The subject is agents operating live systems.
4. **You can state what it cost** — or state plainly that it was caught before
   it cost anything. "Cost" is what was lost or nearly lost, not what could
   theoretically have happened.

**A short log is a good log.** Three well-anchored incidents beat twelve padded
ones. If a month produced nothing new, record exactly that — it is a real
result, and it is the only way the trend ever means anything.

---

## Incident entry format

```markdown
**<number> — <one-line title, stating the defect not the symptom>**
<date>. <What happened, mechanism first. Name the thing that was actually
true and the thing that was believed instead.>

**Cost:** <what was lost or nearly lost, or "none realised because …">
**Anchor:** <commit SHA / log path and line / PR number / API read + date>
**Detection:** `MODE`
```

Rules that earn their keep:

- **Numbers never restart and are never reassigned.** They become external
  references. Reusing one for a different incident silently corrupts anything
  that cited it.
- **A recurrence is not a new incident.** It is a line in the Recurrences
  section, naming the countermeasure it defeated. Recurrences are the strongest
  evidence you have that a fix didn't work.
- **Corrections go in the file, not over it.** A log that silently revises
  itself is worth nothing. In this catalogue the retracted entries are the
  most-read ones.

---

## The monthly pass

1. **Establish the watermark** — the date of the previous edition. Everything
   before it is already catalogued.
2. **Sweep every source.** For this project: agent memory files, the repo's own
   instructions file (much of which is a failure record wearing a rule's
   clothes), reports written since the watermark, the run logs, git history,
   and the open-threads section of the handoff file. Yours will differ; the
   point is that no single source is sufficient.
3. **Apply the bar.** Most candidates won't clear it.
4. **Classify twice.**
5. **Total the detection column**, this period and cumulative.
6. **Write one honest paragraph on whether the automated share is moving.** If
   it is flat, say so.

Two traps worth inheriting from someone else's expensive afternoon:

- **Check your log files' encoding before grepping them.** This project's
  nightly logs are UTF-16; a plain `grep` for `ERROR` matched nothing in them
  regardless of content, in the one section whose purpose is catching silence.
- **A missing log is invisible by construction.** Count files against a
  calendar. And check that a neighbouring file family isn't answering "is there
  a log for that night" with a misleading yes.

---

## What to expect

If your experience resembles this one:

- The automated share will be lower than you expect, and will not improve on
  its own.
- Class **A** will grow as a proportion as your agents get more capable.
- `LATER` will be your largest bucket, which means your log is a lower bound —
  it holds the failures that were *noticed*.
- Your countermeasures will each be named for the one incident that produced
  them, and none of them will catch the next thing.

The last one is the argument for keeping the log at all. You are not going to
test your way to detecting a false conclusion. What you can do is measure how
often you find out, and refuse to let that number be flattering.

---

*Method and catalogue: [github.com/taylorancapital/nothing-threw](https://github.com/taylorancapital/nothing-threw)*
