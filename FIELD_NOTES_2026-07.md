# The nightly's first month, and its first failure nobody escalated

**July 2026.** Nightly automation started 2026-07-24 (`54eae136`) — the first
month there was an unattended agent to fail at all. Part of a five-month
retroactive sweep continuing the numbering from
`reports/NOTHING_THREW_2026-09-04.md` (01–18) and
`reports/FIELD_NOTES_2026-09.md` (19–22).

**The one-line finding:** July's two incidents bracket the whole space this
catalogue cares about — one wrong conclusion, and one correct diagnosis,
repeated accurately for three nights running, that nobody with the ability to
fix it acted on until two days after the third.

| | |
|---|---:|
| Incidents | 2 |
| Caught by anything automated | 1 of 2 |
| Consecutive nights the same blocker was reported before it was fixed | 3 |

---

## EVIDENCE — how they were caught

One `LATER` (a wrong conclusion, found by a more careful subsequent pass
reading actual file content instead of trusting filenames) and one `THREW`
(a credential blocker that surfaced honestly, every single night, and was
reported as such each time). Neither is `HUMAN` or `GATE` — nobody asked a
suspicious question this month, and nothing existed yet to gate anything
automatically. July is the first month the nightly's own output is the
evidence trail, rather than a person's review of it.

## MECHANISM

### A. The agent was confidently wrong — 1 incident

**27 — Filename and mtime were never a valid staleness check.** The
nightly's rotation logic re-uses a prior night's analysis when it decides the
underlying GA4 export is unchanged. On 2026-07-25 and 2026-07-26, and again
in a first pass on 2026-07-27, that decision was made by matching
**filenames and file modification times** against the previous night's export
and concluding "same data, don't repeat the analysis."

That signal was never valid. Browser downloads of GA4 CSVs are always named
`download.csv`, `download (1).csv`, `download (2).csv`… regardless of when
they were pulled — the naming convention is fixed, not derived from pull
date. The user had in fact deleted the old files and re-pulled a fresh 30-day
export on 2026-07-27. Three consecutive nightly cycles treated genuinely new
data as a stale duplicate and skipped analyzing it.

Caught the same night it was written up, by reading each CSV's own
`# 20260628-20260727`-style header line instead of trusting the filename —
the one reliable freshness signal that had been sitting in the files the
whole time.

**Cost:** at minimum two full nightly cycles (07-25, 07-26) where fresh
exported data was never actually analyzed, on the incorrect belief that
nothing had changed. The nightly's entire function — surface what changed —
did not happen for those nights.

**Anchor:** `reports/GA4_ANALYSIS_2026-07-27.md`, "Correction to the prior
nightly log entry" (lines 15–23): *"an earlier run tonight (and the two
before it) concluded this data was unchanged... That was wrong."*

**Detection:** `LATER`.

### F. Correctly diagnosed, not fixed — 1 incident, new class

*Does not fit A–E: the agent was not wrong, nothing was silent, no write
occurred, and this was not a concurrency collision. The failure is that an
accurate, honestly-reported blocker recurred for days because reporting it is
not the same as escalating it, and nothing forced the fix.*

**28 — Three consecutive nights with no way to open a pull request, each one
correctly reported and none of them fixed.** The nightly automation's Cowork
session folder connection covered only `Business Plan\files\Night
Tasks\`, not the sparkdate repo root. That meant `.claude-gh-token.txt` —
expected at the repo root — was unreachable, so no authenticated `git push`
was possible. Per the run's own hard rule ("if missing… STOP and report a
token problem — do not improvise"), no substitute credential was used; each
report was written to the Night Tasks folder as a plain file instead of a
pull request.

This was not silent. `reports/SEO_CTA_HEALTH_CHECK_2026-07-28.md` names the
"same root cause as 2026-07-25 and 2026-07-26" explicitly, states the fix
needed ("confirm the Cowork folder connection... includes the repo root"),
and repeats that exact recommendation — because the two nights before it had
already made it and nothing had changed. At least three consecutive scheduled
runs (07-25, 07-26, 07-28) hit the identical wall and reported it accurately
each time.

**Cost:** three-plus nights of completed analysis stranded outside git,
invisible to anyone not looking directly in the Night Tasks folder. The
reports were eventually recovered and merged 2026-07-30 (`d3208d1d`, PR
#125; `6a6e2d74`, PR #128) — two days after the last of the three. This is
the same *shape* of failure as August's incident 11 (a finished report
stranded with no PR) but a different root cause: incident 11 was a
shell-quoting bug in a script; this is a folder permission scope on the
automation platform itself. The pattern — a report finishes, nothing opens a
PR for it — recurred across two completely different mechanisms and two
completely different automation stacks (Cowork here, the Windows/PowerShell
nightly later) before either was actually fixed.

**Anchor:** `reports/SEO_CTA_HEALTH_CHECK_2026-07-28.md`, "Delivery note —
why there's no PR tonight"; same note in the 07-25 and 07-26 runs per its own
cross-reference; recovery commits 2026-07-30.

**Detection:** `THREW` — the blocker surfaced as an actual, honest stop
every night, which is what puts it here rather than in the silent-failure
class. What makes it worth its own class is what happened *after* the
correct report: nothing, for two more days.

## DECISION — what changed, what didn't

Nothing was built this month against either failure. Incident 27's fix was
local (read the CSV header, not the filename) and did not generalize into a
standing check that later prevented incident 12 (August) — the same
family of "trust the file's own content, not its metadata" mistake recurring
in a different form two months later. Incident 28's underlying folder-scope
problem was worked around by a manual recovery on 07-30, not fixed at the
root; the shape of the failure (a finished report, no PR) recurred again in
August via a different mechanism (incident 11) before anything resembling a
gate — removing push and `gh` access from the nightly agent entirely,
enforced by a separate deterministic script — was built. That countermeasure
doesn't appear until incident 13, in September.

## What I did not verify

- **Whether a first attempt at analysis happened earlier on 07-27 itself**,
  making the true count of wasted cycles three rather than two. The
  correction note says "an earlier run tonight (and the two before it)" —
  read conservatively as at least two full wasted nights, possibly three.
- **The precise date the Cowork folder-connection scope was ever corrected**,
  as opposed to the reports simply being manually pushed around it on 07-30.
  Both `d3208d1d` and `6a6e2d74` are recovery commits for already-written
  reports, not evidence the underlying scope problem itself was fixed.
- **Names, account identifiers and customer records are omitted throughout.**
