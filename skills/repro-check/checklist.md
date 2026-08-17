# repro-check — parameter-disclosure checklist

The Level-1 rubric. For **each results-bearing pipeline** (one per asset group:
a table's training runs, a figure's sweep, a codec's rate curve), walk every
section below, list the parameters an independent implementation needs, and give
each one a disclosure class.

## Disclosure classes

- **(a) stated** — the value appears in the paper (body, caption, or appendix).
- **(b) delegated** — the value is recoverable from a released artifact **and
  the paper explicitly points there** ("hyperparameters as in the released
  configuration", "details in the benchmark repository"). Verify the pointer
  actually resolves: the named file/config exists at the pinned commit and
  contains the value.
- **(c) undisclosed** — neither stated nor explicitly delegated. **Every (c) is
  a finding**, even when the value sits somewhere in public code: "read the
  code" without the paper saying which code encodes the knob is not a methods
  section. Name the recoverable source (file/field at the pinned commit) the
  authors could cite or state.

An explicit delegation that does **not** resolve (dead link, config absent,
value not actually in the artifact) is a **(c) finding of the worst kind** —
the paper claims disclosure it does not deliver.

## Severity calibration

Journal-style papers do not list every constant. Calibrate severity to whether
the value **materially affects the printed numbers**:

- **high** — result cannot be regenerated even approximately without it
  (objective definition, data split, core algorithm choice, stopping rule).
- **med** — result shifts beyond the paper's own printed precision or reported
  spread without it (schedule constants, batch size, seed policy, quantizer
  range convention).
- **low** — cosmetic or robustness-only (plot binning where immaterial, exact
  hardware when precision-insensitive). Prefer PLAUSIBLE + a one-line suggested
  sentence over a hard finding here.

## The eight sections

### 1. Headline-result hyperparameters
Everything the flagship table/figure's runs depended on: optimizer and its
constants, learning-rate peak and schedule shape *with values*, clipping,
batch size, step/epoch counts, objective (including any truncation/retention
fraction), validation split and selection rule (best-checkpoint criterion),
initialization per method variant. Watch for appendices that state constants
for *ablation* runs while the *main* runs' counts appear nowhere.

### 2. Data preparation
Which release/version/split of each dataset; the recipe from raw source to
model input (crop vs. resize, color/luminance conversion, rasterization,
anti-aliasing, category selection, ordering). The classic irreproducibility
hole: the paper states sizes and counts but never *how* images/samples were
produced. Class (b) requires the paper to point at the loader that encodes it.

### 3. Algorithmic disclosure
Every named component must resolve to an algorithm, not a family: "entropy
coding" names a family; the coder, its context model, and its parameters are
the disclosure. Quantizer ranges (per-item min/max vs. fixed), position/index
encoding, how a curve is constructed from a sweep (Pareto? interpolation?),
what a "raw"/"baseline" comparator means in concrete units.

### 4. Randomness and seeds
Every stochastic element (init, batching, augmentation, test-subset draw) has a
seed policy disclosed **where its results appear**. Multi-seed studies: the
seed count and aggregation. Fixed-test-set claims must be auditable.

### 5. Environment
Hardware, numeric precision (float32/64, complex dtype), framework and library
versions for the reported runs. Venue-dependent depth: PLAUSIBLE + suggested
provenance pointer, not a med finding, unless a result plausibly depends on it
(precision often does).

### 6. Uncertainty on aggregates
For every mean/aggregate: is the spread reported (σ, SEM, range — estimator
*named*) or explicitly waived? Are bold/best-marker conventions robust to the
reported spread? Error bars labeled with what they show? A stochastic pipeline
with no reported spread also degrades Level-2 matching (no principled band to
match within) — cross-reference the two findings.

### 7. Availability-statement audit
Every artifact URL in the paper: resolves, public, licensed, and its contents
match what the paper attributes to it ("reproduction scripts" → the entry
points exist and are documented; "the implementation" → the code implements
the paper's method, not a successor). Spot-verify the documented entry point
for at least one flagship asset *exists and is wired* (files present, commands
documented) without running the full pipeline — running is Level 2.

### 8. Determinism caveats
Does the paper claim exact reproducibility anywhere it cannot deliver it
(GPU reduction order, nondeterministic kernels, unpinned dependencies)? The
honest disclosure is a tolerance statement; its absence is a low/med finding.

## Deliberate conventions — do not flag

- Choices the paper *discloses and owns*, however unusual (accounting
  conventions, custom variants) — reproducibility review audits disclosure,
  not taste.
- Values explicitly delegated to an artifact the paper cites, once the
  delegation is verified to resolve.
- Constants immaterial to the printed precision (severity calibration above).
