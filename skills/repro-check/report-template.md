# repro-check report template

The report at `docs/repro/<paper-slug>/report.md` must follow this skeleton.
Fill every section; write "none" rather than deleting a section. Evidence
before assertion: every claim names where you looked.

```markdown
# Reproducibility review — <paper title>

- **Paper:** <arXiv ID / DOI / file>, version <vN / date>
- **Artifacts:** <repo URL> @ <commit hash> (+ datasets/deposits, versions)
- **Review level:** Level 1 only | Level 1 + Level 2 (budget: <cap>, spent: <wall-clock>)
- **Environment (Level 2):** <OS, Python, key package versions, hardware, precision>
- **Date:** <YYYY-MM-DD>

## Verdict

<One paragraph: can an independent group regenerate the paper's results from
the paper plus its declared artifacts? Which assets matched, which did not,
what single change would most improve reproducibility.>

## Scorecard

| Asset | Shows | Pipeline | Match policy | Verdict | Evidence |
|-------|-------|----------|--------------|---------|----------|
| Table 2 | <one line> | <pipeline id> | tolerance | REPRODUCED | logs/table2.txt |
| Fig 4b | ... | ... | statistical | NOT-ATTEMPTED (budget) | would run: `<command>` |

Verdicts: REPRODUCED-EXACT / REPRODUCED (within declared policy) / PARTIAL /
MISMATCH / BLOCKED (artifact missing or broken) / NOT-ATTEMPTED (budget or
Level 1 only — always with the command that would attempt it).

## Findings (high → low)

### F1 (high, CONFIRMED) — <pipeline>: <parameter or asset>
- **Class:** (c) undisclosed
- **Evidence:** searched <paper locations> and <artifact paths @ commit>; not found.
- **Impact:** <why the printed numbers depend on it>
- **Suggested fix:** "<one-sentence disclosure in the paper's style>"

<CONFIRMED = verified directly; PLAUSIBLE = likely but not fully verified
within budget — say what would confirm it.>

## Parameter-disclosure matrix

Per pipeline, the full walk of checklist.md sections 1–8: parameter → class
(a)/(b)/(c) → where stated/recoverable. This is the "checks passed" record —
it shows the audit was exhaustive, not just the failures.

## Execution log (Level 2 only)

Per attempted asset: entry point run, commit, wall-clock, key output values
vs. paper values, tolerance applied, raw logs under logs/.

## Out of scope

<One-liners: what this review deliberately did not cover, and where it is
handled (scientific merit, literature fact-check → paper-reviewer, ...).>
```

Final chat message after writing the report: verdict counts by category plus
one line per MISMATCH/BLOCKED/high finding. The user decides what to do next;
do not open issues or contact authors unasked.
