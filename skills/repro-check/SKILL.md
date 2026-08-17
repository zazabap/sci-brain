---
name: repro-check
description: Use when independently checking whether a scientific paper's results are reproducible — "is this paper reproducible", "check reproducibility of arXiv:2401.12345", "do the tables/figures match the released code", "third-party review of this paper's artifacts", "try to reproduce this PDF's results". Takes an arXiv link/ID, DOI, local PDF, or local source, plus whatever artifacts the paper declares (GitHub repo, data deposits). Not for prose/reference review of a manuscript (use paper-reviewer), figure aesthetics (use figure-taste), or building a permanent validation harness (use autoresearch-validator).
---

# Repro Check

Run a **third-party reproducibility review** of a scientific paper: determine
whether an independent group could regenerate every results-bearing table,
figure, and headline number from the paper **plus the artifacts the paper
itself declares**, and — when the user authorizes execution — actually
regenerate what the budget allows and compare numbers.

**Scope note.** This reviews *whether the results regenerate from the released
artifacts*. It is **not** `paper-reviewer`, which reviews prose, references,
and claim–text consistency of a manuscript. It is **not** `figure-taste`
(visual quality) and **not** `autoresearch-validator` (builds a permanent
gated validator for one's own project). A paper with no code release is still
in scope: the review then documents exactly what is missing.

Rubric: `skills/repro-check/checklist.md`. Report skeleton (mandatory):
`skills/repro-check/report-template.md`.

---

## Operating principle

**Paper-first, report-only, evidence before assertion.**

- **Paper-first** — the ground truth for "reproducible by a third party" is
  the paper text plus the artifacts it *explicitly points to*, resolved at
  their declared or recorded versions. A value that lives in public code the
  paper never points to is still an undisclosed value. A local checkout you
  happen to have is **not** the artifact; clone from the declared URL and pin
  the commit (a working tree can be ahead of or behind what readers get).
- **Report-only** — **never edit** the paper or the artifact repositories.
  The only outputs are the report directory and scratch clones/environments.
- **Evidence before assertion** — every verdict names where you looked, on
  both the paper side and the artifact side, at which commit.

## The two levels

- **Level 1 — disclosure audit** (default; never executes artifact code):
  static classification of every parameter each results pipeline needs.
- **Level 2 — execution audit** (opt-in, budgeted): regenerate assets from the
  artifacts and compare numbers, climbing the cheapest-first ladder below.

Ask once at intake (with a default): *"Level 1 only, or Level 1 + Level 2
execution? If Level 2: what compute budget?"* **Default if unspecified:**
Level 1 plus the free rungs of Level 2 (R0–R1, ≤ 10 min wall-clock each,
no training, downloads ≤ 100 MB). Anything beyond the default budget —
long jobs, large dataset downloads, any training — needs explicit
per-step confirmation.

---

## Phase 0 — Intake & acquisition

1. **Resolve the paper.** Accept an arXiv link/ID, DOI, local PDF, or local
   source (`.tex`/`.typ`/`.md`). For arXiv/DOI inputs, acquire via
   `download-ref` when a knowledge base is in play (it fetches metadata, the
   PDF, and renders markdown); otherwise fetch the PDF directly and render it
   to text in a scratch directory. Record the paper **version** (arXiv vN,
   journal DOI, or file date + hash) — reproducibility verdicts are
   version-specific.
2. **Pick the review level and budget** (question above; apply the default in
   non-interactive runs and say so in the report).
3. **Create the report directory** `docs/repro/<paper-slug>/` with `logs/`.
   All evidence lands here as you go, not reconstructed at the end.

## Phase 1 — Asset inventory

Enumerate every **results-bearing asset**: numbered tables, figures/plots with
quantitative content, and headline numbers stated in prose or abstract. For
each, record: asset id (`Table 2`, `Fig 4b`, `abstract:+5.4dB`), what it
claims, the pipeline that produced it, and the key printed values with their
printed precision. Assets the user explicitly scoped out are listed as
out-of-scope, not silently dropped. This inventory keys every later phase and
the final scorecard.

## Phase 2 — Artifact resolution

1. Collect every artifact the paper declares: code URLs, data-availability
   statements, deposits (Zenodo/OSF), supplementary files.
2. Clone/download each from the **declared** location, at the version the
   paper names (tag, commit, "as of" date); if none is named, use the current
   default branch and record that choice as a finding candidate (checklist §7).
   **Record the commit hash** of everything you use.
3. Audit the availability statements themselves (checklist §7): URLs resolve,
   licenses present, contents match what the paper attributes to them, and the
   documented entry point for at least one flagship asset exists and is wired
   (files present, commands documented) — *without running it yet*.
4. Map each asset from Phase 1 to its claimed regeneration path (README,
   reproduction guide, Makefile target), or mark it unmapped.

## Phase 3 — Level 1: disclosure audit

For each pipeline, walk all eight sections of `checklist.md` and classify every
parameter an independent implementation needs:

- **(a) stated** in the paper;
- **(b) delegated** — recoverable from a declared artifact **and** the paper
  explicitly says so (verify the pointer resolves at the pinned commit);
- **(c) undisclosed** — a finding, *even when the value sits in public code*.
  "Read the code" is not a methods section; the finding names the exact
  file/field the authors could cite.

Output: the per-pipeline **parameter-disclosure matrix** plus severity-ranked
findings (severity calibration in `checklist.md`). Also audit uncertainty
reporting (checklist §6): every mean has a named spread or an explicit waiver.

## Phase 4 — Level 2: execution audit (gated)

**Before anything runs:** per asset, declare the **match policy** — you must
write it into the report *before* seeing any output, and never loosen it
afterwards (a downgrade after a mismatch is itself a finding):

- **exact** — deterministic pipeline; printed precision is the tolerance
  (printed `31.42` matches any output that rounds to `31.42`);
- **tolerance** — deterministic modulo floating-point/backend noise; match
  within the printed last digit, deviation reported;
- **statistical** — stochastic pipeline; match within the paper's own reported
  spread. No reported spread → that's already a Level-1 finding; state an
  explicit ad-hoc band and mark it as such.

**Environment:** isolated only — the repo's declared Docker image or lockfile
if it has one, else a fresh venv in the scratch area. Never install into the
user's environment, never elevate privileges, no network beyond the declared
artifacts and the datasets their scripts fetch. Record OS, Python, package
versions, hardware, and precision in the report.

**The ladder** — climb cheapest-first, per asset; stop at the budget:

| Rung | What runs | Typical cost |
|------|-----------|--------------|
| R0 | Recompute printed values from **committed result data** (JSON/CSV in the repo) — no pipeline code at all | seconds |
| R1 | Re-render figures/tables via the repo's entry points from committed data | seconds–minutes |
| R2 | Re-evaluate metrics from committed checkpoints/models on obtainable data | minutes–hours; confirm |
| R3 | Full re-training / re-simulation from scratch | hours+; explicit go-ahead only |

R0 is almost always available and already decisive: it separates "the numbers
follow from the released data" from "the release doesn't even contain them".
Independent assets may be attempted by parallel subagents, each writing its
log under `logs/`; the comparison against the paper values stays in the main
context.

**Verdicts** (fixed vocabulary — no free-form "looks reproducible"):
`REPRODUCED-EXACT` / `REPRODUCED` (within the declared policy) / `PARTIAL`
(some values match, some don't — list both) / `MISMATCH` / `BLOCKED`
(artifact missing, broken, or unobtainable) / `NOT-ATTEMPTED` (budget or
Level 1 only — always with the exact command that would attempt it).
GPU-reduction-order noise on an exact policy → report as `REPRODUCED` with
the cause named, not `MISMATCH`.

## Phase 5 — Deliver

Write `docs/repro/<paper-slug>/report.md` following `report-template.md`
exactly: verdict paragraph → scorecard → findings (high → low, each with
disclosure class, evidence trail, CONFIRMED/PLAUSIBLE, and a suggested
one-sentence disclosure fix) → parameter-disclosure matrix → execution log →
out-of-scope. Then give a short terminal summary: verdict counts plus one line
per MISMATCH/BLOCKED/high finding. Do not contact authors or open issues
unasked.

---

## Reused vs. new

**Reused:** `download-ref` for paper acquisition and KB layout; severity
conventions and the non-destructive operating principle from `paper-reviewer`;
the report-only discipline from `figure-taste`.

**New here:** the asset inventory as the unit of review, the (a)/(b)/(c)
disclosure classification, the declare-before-compare match policy, the
R0–R3 execution ladder, and the fixed verdict vocabulary.

## Common mistakes

| Mistake | Instead |
|---|---|
| "The value is in the repo, so it's disclosed" | Class (b) requires the *paper* to point there; otherwise it's (c) — a finding naming the file the authors should cite. |
| Deciding the tolerance after seeing the output | Match policy is written into the report before anything runs; loosening it afterwards is a finding, not a fix. |
| Auditing a local checkout or submodule you happen to have | Clone from the declared URL at the declared/recorded ref; diff against any local copy only as a *finding* (e.g., pointer drift). |
| Free-form verdicts ("strong", "seems fine") | Use the six-verdict vocabulary; prose goes in the evidence, not the verdict. |
| Delivering findings only in chat | The report file with the evidence trail *is* the deliverable; chat gets the summary. |
| "Nothing is runnable here, so nothing can be verified" | R0 needs no pipeline environment — recompute from committed result data first. |
| Skipping spread checks because means matched | A mean without a named spread estimator is a checklist §6 finding even when R0 matches. |
| Editing the paper or artifacts to "make it work" | Report-only. A required patch is a `BLOCKED`/`PARTIAL` finding quoting the patch. |

## Integrations

- **Paper acquisition / rendering:** `download-ref` (arXiv/DOI metadata, PDF →
  markdown, shallow repo clones under `.raw/repos/` when a KB is in use).
- **Rubric:** `skills/repro-check/checklist.md`.
- **Report skeleton:** `skills/repro-check/report-template.md`.
- **Manuscript prose/claims/references:** `paper-reviewer` (different concern).
- **Permanent validation harness for your own project:** `autoresearch-validator`.
