# Frame-A rebuild-of-record runbook (2026-07-31)

**Status of the old artifact.** `submissions/frame-a-eswa/main.pdf` and every table in it are
QUARANTINED by the 2026-07-26 red-team finding (fabricated cells: MIX_B/MIX_C "real" rows were
synthetic relabels; MIX_A itself is 30/33). The rule for this rebuild is absolute:

> **The paper is RE-DERIVED, never patched.** No number enters a table, macro, figure, or
> sentence unless it is emitted by a step below, from a cell that passed `provenance_gate_v2`.
> A number that exists only in the old PDF, the old `macros.tex`, or a memory note is treated
> as absent.

Two-stage structure, because MIX_C does not exist yet:

| Stage | Data basis | What it produces | Publishable? |
|---|---|---|---|
| **Stage 1** (now, local, ¥0) | MIX_A + MIX_B only, after the refill cells land (7 outstanding as of 07-31 — see step a) | **internal draft** — verdict + prose + all MIX_A/B numbers | NO. Internal only. |
| **Stage 2** (needs Batch 2) | + MIX_C 33 cells + 3 quarantined MIX_A cells | **rebuild-of-record** — the submittable paper | YES, after (j). |

**Binding: the rebuild-of-record waits for MIX_C.** The paper's P2 structural claim
(router-edit-majority on privacy updates) lives *only* in MIX_C, and the frontier figure is a
1×3 mix patchwork. A Stage-1 draft may be written, reviewed, and iterated, but it must carry a
visible `DRAFT — MIX_A/B only, MIX_C pending` banner and must never be compiled into an artifact
named as a submission candidate.

---

## Step (a) — 9 refill cells land

**Owner:** the local refill driver (running now).
**Input:** the refill driver's own config; it re-runs the s2 / quarantined cells.
**Output artifacts:** `edit-harness/results/frame_a/cells/cell_llama-3.2-1b_real_MIX_{A,B}_<policy>_s<seed>.json`

**Verified live state, 2026-07-31** (measured, not assumed — commands below):
coverage **59/66**, every gap an `s2` cell, so the refill is on its last seed:
- MIX_A missing `s2` for `cost_only`, `random`, `ft_merge` (3)
- MIX_B missing `s2` for `always_rag`, `always_reject`, `random`, `ft_merge` (4)

That is **7** outstanding, not 9 — 2 of the brief's 9 have already landed. Gate v2 currently
returns exit 2 (`INCOMPLETE`) and `q_ext_analysis --no_write` runs cleanly on the partial set, both
as this runbook specifies.

Completion check — **do not use log markers** (the 07-10 lesson: logs carry stale DONE lines).
Completion = the target cell JSONs on disk with a valid `runner_stamp`:

```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis/edit-harness
python -m experiments.frame_a.q_ext_analysis --cells_dir results/frame_a/cells --no_write
```

The human summary prints `coverage: n_present/n_expected` plus per-mix `missing_seeds`.
`--no_write` means a mid-flight read cannot clobber a good artifact. The authoritative fields are
`coverage.per_mix.<mix>.missing_seeds` and `coverage.complete`; `complete == true` means
`n_present == n_expected` over the analysis module's own scope, which is MIX_A + MIX_B
(11 policies × 3 seeds × 2 mixes = 66). The module skips unreadable mid-write cells by design and
reports them as coverage, not as an error.

**Live-file hazard.** While the refill driver runs, do NOT edit `run_stream.py`,
`real_replay.py`, `config.py`, or any file under `scorer/`. `q_ext_analysis.py` was written to
import policy names by literal, not from `run_stream`, precisely so this step is safe.

---

## Step (b) — `provenance_gate_v2` PASS

**Command** (flags are underscore-style — verified against the argparse block):
```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis/edit-harness
python -m experiments.frame_a.provenance_gate_v2 \
  --cells_dir results/frame_a/cells \
  --report results/frame_a/gate_v2_report_<date>.json
```

**Scope warning, read before interpreting the exit code.** The gate's in-scope universe is
`EXPECTED_MIXES = ("MIX_B", "MIX_C")` — **MIX_A is not gated**, and
`EXPECTED_TOTAL = 11 policies × 3 seeds × 2 mixes = 66`, not 63. The brief's "63 cells" does not
match the code. See spec conflict 9.

**Output:** gate report JSON + exit code. Semantics (read from the code, not assumed):

| exit | status | meaning | may the rebuild proceed? |
|---|---|---|---|
| 0 | `PASS` | no FAIL findings, `n_in_scope == 66`, a `p2_ok` finding present | yes |
| 1 | `FAIL` | any FAIL finding, or in-scope count met but no `p2_ok` | **no** — quarantine, do not repair in place |
| 2 | `INCOMPLETE` | fewer than 66 in-scope cells | Stage 1 only; Stage 2 blocked |

Note the FAIL ordering: `any_fail` is checked **first**, so a FAIL finding on a partial wave
reports exit 1, not exit 2. A partial wave that trips `synthetic_anchor_v2` is a fabrication
signal, not an incompleteness signal — treat exit 1 as such regardless of coverage.

`p2_ok` requires the namespaced artifact `p2_llama-3.2-1b_real_MIX_C.json` — **a MIX_C artifact**.
So a `PASS` (exit 0) is structurally unreachable until the Batch-2 wave lands. At Stage 1 the gate
returns exit 2 `INCOMPLETE`; that is correct behaviour, not a defect to work around.

v2 adds four check groups over v1: `synthetic_anchor_v2` (the 07-21 relabel signature),
`primary_metric_variance_v2` (a metric that is identical across seeds is a relabel tell),
`same_second_writes_v2`, and `runner_stamp_v2` (cells written after
`DEFAULT_RUNNER_STAMP_CUTOFF = 2026-07-27T00:00:00Z` **must** carry a valid `runner_stamp`;
older cells are downgraded to `legacy_runner_stamp_v2`).

**Acceptance artifact for H14:** write `engine/FRAME_A_GATE_V2_PASS.ok` containing the gate
JSON's `status`, `counts.in_scope_cells`, and the ISO timestamp — only when exit code is 0.
The `.ok` file is written **by the gate run**, never by hand.

Stage-1 note: at Stage 1 the gate reports `INCOMPLETE` (MIX_C absent, so both the 66-cell count and
`p2_ok` are unreachable). That is expected and is the reason Stage 1 is not a rebuild-of-record.
Record the exit-2 report; **do not lower `EXPECTED_TOTAL` or `EXPECTED_MIXES` to make it green.**

**MIX_A gap.** Because MIX_A is outside `EXPECTED_MIXES`, gate v2 does not vet the 3 requarantined
MIX_A cells at all, yet the paper's primary tables are MIX_A. Closing this needs a decision —
either extend `EXPECTED_MIXES` to include MIX_A (changes `EXPECTED_TOTAL` to 99) or run a
separate MIX_A-scoped verification pass. See spec conflict 9; **do not build a MIX_A table on
ungated cells.**

---

## Step (c) — `q_ext_analysis` run (M6)

**Command** (underscore flags, verified against argparse):
```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis/edit-harness
python -m experiments.frame_a.q_ext_analysis --selftest        # run first: must PASS
python -m experiments.frame_a.q_ext_analysis \
  --cells_dir results/frame_a/cells \
  --expect_model llama-3.2-1b --expect_provenance real
```

`--no_write` prints the human summary without emitting the artifact — use it for the coverage read
in step (a) so a partial wave never overwrites a good artifact.

**Input:** `results/frame_a/cells/cell_*.json` (read-only).
**Output (exactly one artifact):** `edit-harness/results/frame_a/Q_ext_analysis_20260731.json`

Contents the rebuild consumes: `coverage`, `central` (per-mix per-policy `Q`, `Q_ext`, cost,
`n_seeds`), `gates.{G_Q1,G_Q2,G_Q3,G_Q4}`, `sensitivity_grid` (27 settings),
`per_cell` (every frac's numerator + denominator), `latency_provenance`, `integrity_note`.

Frozen central setting: `λ_cap=0.15, λ_lat=0.10, λ_stale=0.15`. Do not vary it outside the
27-setting grid the amendment fixed.

**Two provenance facts that must travel with every Q_ext number into the paper:**
1. `Q_ext` is **post-data and exploratory**; `Q` is the pre-registered primary. Every table
   reports them side by side (that is G-Q4's purpose).
2. The latency term is tagged `synthetic-frozen-clock`, not measured-real-replay
   (`Arm.serve_overhead` reads `SyntheticClock`; no real backend overrides it). This caveat
   binds `λ_lat` only — at most 0.10 of the composite. Capacity and staleness are unaffected.

**M6 §Integrity, non-negotiable:** `capacity_frac`, `latency_frac`, `stale_frac` must each be
recomputed once from the persisted aggregates by a **reviewer pass that did not write the
scorer**, before any paper use. `per_cell` carries every numerator/denominator so the recompute
needs this JSON but not this code. Log the recompute as its own artifact
(`results/frame_a/Q_ext_frac_recompute_<date>.json`) with a PASS/FAIL per frac.

---

## Step (d) — G-Q1..G-Q4 verdict evaluation

Read from `gates` in the step-(c) JSON. Decision logic, exactly as frozen:

- **G-Q1 (sanity, hard stop).** No policy may exceed the oracle's `Q_ext` in **any** of the 27
  settings, in **both** mixes.
  - PASS → G-Q2/G-Q3 verdicts are quotable; a frontier under `Q_ext` may be published.
  - FAIL → `Q_ext` is still mis-specified. G-Q2/G-Q3 are still *computed* (for the record) but
    the driver stamps them `verdict_suppressed_by_G_Q1`; those numbers **must not be quoted**
    and **no Q_ext frontier may be published**. Read `worst_violator` (policy, margin, setting)
    and report it as the finding.
- **G-Q2 (primary).** At the central setting, does `both` Pareto-dominate `always_grace` in at
  least one mix, under the *unchanged* MINOR-2 CI-level predicate (seed-level bootstrap over
  orderings 0/1/2)? Field: `gates.G_Q2.passes`.
  - The JSON also carries `always_grace_dominates_both` (the complementary reading) and
    `mutual_non_dominance` per mix. Report all three; a mutual non-dominance with 3 seeds may be
    a power floor rather than a genuine tie, and the Q-only comparison is printed alongside so a
    reader can tell.
- **G-Q3 (honest-negative).** Triggered iff `always_grace` dominates in **both** mixes in a
  **majority** of the 27-setting grid. If triggered, the finding to publish is the gate's own
  `statement` string: on this stream a zero-damage codebook is the right policy and routing does
  not pay. **Extending the metric further to manufacture a router win is forbidden by M6.**
- **G-Q4 (no-op check).** `Q` vs `Q_ext` side by side, all 11 policies × 2 mixes, with
  `rank_shift`. Any policy shifting more than 2 rank positions is auto-`flagged` with an
  `explanation` field — every flagged row needs a sentence in the text. Do not bury the
  extension's effect in a frontier plot.

**Expected outcome (stated in advance so it cannot be re-narrated later):** KILL on the
pre-registered rule. On `Q`, `both − always_grace = −0.016` in MIX_A (CI `[−0.041, −0.003]`,
FAIL) and ties in MIX_B/MIX_C. The paper's headline is the measurement, not a router win.

---

## Step (e) — verdict JSON

**Command:**
```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis/edit-harness
python -m experiments.frame_a.scorer.analyze_frame_a --cells-dir results/frame_a/cells
```

**Output:** the pre-registered verdict artifact (`results/frame_a/frame_a_verdict.json` for the
rebuild; the old `frame_a_verdict_ftfix.json` is **quarantined** — the 07-26 finding states it is
invalid for all three mixes and it must not be read by any rebuild step).

The verdict JSON supplies: `per_mix.<mix>.P1_detail` (the CI-level pairwise deltas),
`per_mix.<mix>.discovery`, the P1–P4 sub-findings, and
`measured_vs_synthetic_cost_ratio_check.per_arm.<arm>`.

**Pairing rule.** `frame_a_verdict.json` (pre-registered `Q`) and
`Q_ext_analysis_20260731.json` (exploratory `Q_ext`) are both required. Every table that shows a
`Q_ext` column must show the `Q` column from the verdict JSON beside it.

---

## Step (f) — macros regeneration

**Delete `macros.tex` and regenerate.** The existing file is quarantine-tainted: its header
claims a "full 99-cell wave-1 grid on disk", its `% SOURCE` lines point at
`frame_a_verdict_ftfix.json`, and its MIX_B/MIX_C blocks are the fabricated rows. Do not diff
against it; do not "keep the ones that look right".

Rule for every macro: a `% SOURCE:` comment naming the artifact and the JSON key path, and a
machine-checked assertion that the macro's value equals that key's value at the stated rounding.
No macro without a passing assertion.

### Macro ledger — what the paper needs, and from where

**Group 1 — MIX_A Q operating points.** SOURCE: `frame_a_verdict.json::per_mix.MIX_A.<policy>.Q`
(means over seeds 0/1/2).

| macro | policy |
|---|---|
| `\qBoth` | `both` (router) |
| `\qGrace` | `always_grace` |
| `\qEdit` | `always_edit` |
| `\qRag` | `always_rag` |
| `\qCostOnly` | `cost_only` |
| `\qDamageOnly` | `damage_only` |
| `\qOracle` | `oracle` |
| `\qRandom` | `random` |
| `\qFloor` | `always_ft` / `always_reject` / `ft_merge` (state the range, or split into three macros — a single "floor" macro hides three policies) |

**Group 2 — MIX_A total GPU-seconds (install + serve).** SOURCE:
`frame_a_verdict.json::per_mix.MIX_A.<policy>.cost_total_gpu_s`.
`\cBoth`, `\cGrace`, `\cEdit`, `\cRag`, `\cCostOnly`, `\cRandom`, `\cFloor`.

**Group 3 — P1 pairwise CI detail.** SOURCE: `frame_a_verdict.json::per_mix.MIX_A.P1_detail`.
`\pDeltaQedit`, `\pDeltaQgrace`, `\pDeltaCostRag` — each needs its CI interval carried into the
text, not just the point.

**Group 4 — router behaviour.** SOURCE:
`cell_llama-3.2-1b_real_MIX_A_both_s0.json::routing.arm_counts` and the cell's `lambda_cost`.
`\routerGraceShare`, `\routerEditShare`, `\lambdaCost`.
*Spec note:* the old file called these "stable across seeds" on a seed-0 read. Either state
seed-0 explicitly in the macro comment or emit the 3-seed mean plus spread. Do not assert
stability without the cross-seed numbers.

**Group 5 — discovery / predictor.** SOURCE:
`frame_a_verdict.json::per_mix.MIX_A.discovery`.
`\discRecall`, `\discLift`, `\discCeil`, `\discN`.
*Binding:* `\discN` is below the pre-registered point-claim floor (50) → the point claim stays
suppressed, CI-only, per prereg rev.5. And `\discCeil` is the FA-3 number — see step (i) and the
disclosure paragraph in `CONTRIB-REFRAME-DRAFT.md`.

**Group 6 — MIX_B operating points.** SOURCE:
`frame_a_verdict.json::per_mix.MIX_B.<policy>.Q`. `\qBothB`, `\qGraceB`, `\qEditB`, `\qRagB`
(+ cost twins if the paper tabulates MIX_B cost).

**Group 7 — MIX_C operating points. STAGE 2 ONLY.** SOURCE:
`frame_a_verdict.json::per_mix.MIX_C.<policy>.Q` after the Batch-2 cells land and gate v2
passes. `\qBothC`, `\qGraceC`, `\qEditC`, `\qRagC`. **Every MIX_C macro in the current file is
fabricated — none may be carried forward.**

**Group 8 — P1..P4 sub-findings.** SOURCE: `frame_a_verdict.json` (the rebuilt one).
`\pDeltaQeditA`, `\pDeltaQgraceA`, `\pDeltaQgraceB`, `\pDeltaQgraceC` (Stage 2),
`\routerMajorityOnPrivacy` (**MIX_C-only → Stage 2**), `\routerMajorityThreshold` (structural
constant 0.5, pre-defined, no artifact needed but say so).

**Group 9 — measured-vs-synthetic per-arm cost ratios.** SOURCE:
`frame_a_verdict.json::measured_vs_synthetic_cost_ratio_check.per_arm.<arm>`.
`\cratioEdit`, `\cratioFt`, `\cratioGrace`, `\cratioRag`, `\cratioReject`,
`\nqEdit`, `\nqFt`, `\nqGrace`, `\nqRag`, `\nqReject`.
This group is the RAG cost surprise (rag ≈ 17× edit/grace) — a contribution in its own right.

**Group 10 — NEW, Q_ext (M6).** SOURCE: `Q_ext_analysis_20260731.json`.
- `\qextBoth{A,B,C}`, `\qextGrace{A,B,C}`, `\qextOracle{A,B,C}` ← `central.per_mix.<mix>.<policy>.Q_ext`
- `\qextLamCap` `0.15`, `\qextLamLat` `0.10`, `\qextLamStale` `0.15` ← `lambda_central_frozen`
- `\qextGridN` `27`, `\qextGQoneStatus`, `\qextGQtwoStatus`, `\qextGQthreeFrac` ← `gates.*`
- `\qextCapFracGrace`, `\qextLatFracGrace`, `\qextStaleFracGrace` ← `central.per_mix.*.always_grace`
  penalty components (the three fracs that pay for the zero-damage advantage)
- `\qextRankShiftMax` ← largest `|rank_shift|` in `gates.G_Q4`
Every Group-10 macro's comment must contain the word `exploratory` and, for any
latency-derived value, `synthetic-frozen-clock`.

**Group 11 — DELETE, do not regenerate.** `\ftFlushesExpected`, `\ftFlushesObserved` — FT-defect
forensics, marked "REMOVE from camera-ready" in the old file. They belong in working notes, not
the manuscript.

**Machine check — use the existing cross-paper auditor, do not write a new one.**
`submissions/audit_macro_sources.py` already does exactly this job for all four manuscripts:
parses every `\newcommand{\X}{value}` plus its trailing `% ...` provenance comment, resolves the
named artifact and field path, recomputes, and compares **at the macro's displayed precision**,
classifying MATCH / MISMATCH / CANNOT-VERIFY with a reason. It is read-only with respect to every
manuscript.

```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis
python3 submissions/audit_macro_sources.py --json /tmp/fa_macro_audit.json
python3 submissions/audit_macro_sources.py --macro qBoth        # single-macro trace
```

H14's acceptance condition ("every macro machine-checked SOURCE→value") is: **zero MISMATCH and
zero CANNOT-VERIFY** among Frame-A macros. A CANNOT-VERIFY is not a pass — it means the `% SOURCE`
comment does not name a resolvable artifact + field path, which is the exact gap the fabricated
rows hid in. Fix the comment or drop the macro.

This audit runs before every PDF build (step h precondition 2).

---

## Step (g) — R figures

**Generator:** `submissions/frame-a-eswa/figures-r/make_figures_frame_a.R`
(cloned from `submissions/ieee/figures/make_figures_ieee.R`; R/ggplot2 → tikzDevice; every
plotted number carries a provenance string; **`\resizebox` around tikzpictures is BANNED**).

```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis/submissions/frame-a-eswa
Rscript figures-r/make_figures_frame_a.R --preflight   # print gate, exit
Rscript figures-r/make_figures_frame_a.R --preview     # MIX_A-only, writes to figures-qa/ watermarked
Rscript figures-r/make_figures_frame_a.R               # final, fail-closed
Rscript figures-r/tests/test_make_figures_frame_a.R    # generator unit tests
```

The generator is **fail-closed**: any preflight failure → nonzero exit and it writes **no**
`fig02_pareto.tex` / `fig03_router_discovery.tex` / `fig04_gate_evidence.tex`. Do not bypass it.

### Which figures exist

| figure | file | status | notes |
|---|---|---|---|
| Fig 1 — routing framework schematic | `figures/fig01_routing_framework.pdf` | **EXISTS**, direct-TikZ, not data-driven | Re-render + eyeball. Diagram only, no fabricated numbers, so it survives quarantine — but confirm no arm label contradicts the rebuilt verdict. |
| Fig 2 — Q vs cost Pareto, 1×3 mix patchwork | `fig02_pareto.tex` | generator EXISTS; **needs MIX_C data** | Stage 1 renders 1×2 (or the MIX_A watermarked preview) only. |
| Fig 3 — router arm counts (a) + discovery recall@decile CI (b) | `fig03_router_discovery.tex` | generator EXISTS | Panel (b) must show CI-only (`\discN` < 50 floor). |
| Fig 4 — gate status (a) + MIX_B Pareto drilldown (b) + MIX_C structural evidence (c) | `fig04_gate_evidence.tex` | generator EXISTS; **panel (c) needs MIX_C** | Truth-first: the generator treats PASS / GREY / KILL as all valid renderable states. |

### Which figures need new specs

- **[F4] frontier under `Q_ext`** — the H15 acceptance artifact. Does not exist. Spec needed:
  same Q-vs-cost geometry as fig02 but with the `Q_ext` series, oracle marked as the ceiling, and
  the `Q` frontier overplotted faint so the reader sees the metric's effect. **Gated on G-Q1
  PASS** — if G-Q1 fails, this figure must not be produced at all (M6: no frontier under
  `Q_ext`); substitute a G-Q4 rank-shift table.
- **G-Q4 rank-shift panel** — new spec: 11 policies × 2 (Stage 2: 3) mixes, `rank_Q` →
  `rank_Q_ext` slope chart, flagged rows (|shift| > 2) emphasised. Cheap, and it is the honest
  visual for a KILL verdict.
- **Sensitivity-grid strip** — new spec (optional): across the 27 λ settings, a small-multiples
  strip of "does `always_grace` dominate both mixes?" — makes the G-Q3 majority visible instead
  of asserted.

**Rendered-page QA is mandatory.** The 07-05 lesson: agents' "visually clean" claims were wrong;
trust only rendered-page inspection. Every figure gets re-rendered and eyeballed at final size;
any overlap or clip is a defect, listed and fixed before step (h).

---

## Step (h) — PDF rebuild

```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis
python3 submissions/audit_macro_sources.py       # step (f) gate — must be clean first
cd submissions/frame-a-eswa && latexmk -pdf main.tex   # or the repo's existing build target
```

Preconditions, all hard:
1. `engine/FRAME_A_GATE_V2_PASS.ok` exists (step b, exit 0).
2. `audit_macro_sources.py` reports zero MISMATCH and zero CANNOT-VERIFY for Frame-A (step f).
3. Figure generator exited 0 in final mode (step g).
4. Stage 2 only: MIX_C macros present and sourced.

Build gates: 0 errors, 0 overfull boxes, 0 undefined references. Wide tabulars never emit
overfull warnings — measure them with `\savebox`, because the log is blind to them.

Stage-1 builds output to `main-draft.pdf` with the DRAFT banner. `main.pdf` is written only at
Stage 2, after step (j).

---

## Step (i) — leak sweep

```bash
cd /home/zeyufu/Desktop/idea-feasibility-analysis/submissions/frame-a-eswa
pdftotext main.pdf - > /tmp/fa_rendered.txt
grep -nE 'B6|D2|D3|E6|H1[0-9]|FA-[123]|MIX_[ABC]|G-Q[1-4]|M6|Frame-A|run_u[0-9]|run_mixc|\.json|edit-harness|results/|cell_llama|q_ext_analysis|provenance_gate|VALIDATE|prereg' /tmp/fa_rendered.txt
```

Target: **0 hits**. Categories to strip (each was a real leak in the 07-05 sweep):
- **Internal codenames**: `B6`, `D2`, `D3`, `E6`, `FA-1/2/3`, `H14`, `H15`, hole IDs, `Frame-A`
  itself as a project name.
- **JSON / file paths**: any `% SOURCE` value leaking into rendered text, `results/...`,
  `edit-harness/...`, `cell_llama-3.2-1b_...`, `frame_a_verdict.json`,
  `Q_ext_analysis_20260731.json`. Replace with one anonymity-safe reproducibility sentence.
- **CLI flag syntax**: `--expect-provenance`, `--cells-dir` → prose descriptions.
- **Gate codenames**: `G-Q1..G-Q4`, `P1..P4`, `MINOR-2`, `DOF-2` → describe the gate in words
  ("the pre-registered sanity check that no policy may exceed the oracle").
- **Mix codenames**: `MIX_A/B/C` → named workload mixes in the paper's own vocabulary.
- **Amendment codename**: `M6` → "a post-data pre-registered amendment", with the amendment
  document cited by its public identifier, not its internal name.

Also sweep the figure `.tex` files — provenance headers are `%`-comments in source (correct) but
must not surface in captions. And keep the FA-3 disclosure sentence itself: it names the *other
paper's* evidence base, which is a required disclosure, not a leak. Write it in venue-neutral,
anonymity-safe form (see `CONTRIB-REFRAME-DRAFT.md`).

Artifact: `submissions/frame-a-eswa/LEAK-SWEEP-<date>.txt` with the grep output (0 hits) and the
list of patterns swept.

---

## Step (j) — hostile review

Separate pass, separate context, author does not review. Reviewer's mandate:

1. **Recompute ~30 numbers from raw cells/JSON**, independent of `audit_macro_sources.py` (the
   auditor checks macro-vs-artifact; the reviewer checks artifact-vs-raw-cells, which is the layer
   the fabricated rows lived in). Any irreproducible number = FAIL, quarantine, do not repair in
   place.
2. **The three M6 fracs** (`capacity_frac`, `latency_frac`, `stale_frac`) recomputed from
   `per_cell` numerators/denominators by someone who did not write the scorer — this is the
   explicit M6 §Integrity requirement and it gates *any* paper use of `Q_ext`.
3. **Verdict wording audit**: does the text state the KILL plainly? Is `Q` labelled the
   pre-registered primary and `Q_ext` labelled exploratory *everywhere*? Is the latency term's
   `synthetic-frozen-clock` provenance disclosed at the point of use, not only in an appendix?
4. **Fabrication regression**: does any table row trace to a cell without a `runner_stamp`?
   Does any MIX_C number exist without a Batch-2 cell behind it?
5. **Rendered-page inspection** of every figure at final size.
6. **G-Q1-FAIL discipline**: if G-Q1 failed, confirm no `Q_ext` frontier appears anywhere and no
   G-Q2/G-Q3 verdict is quoted.
7. **Placeholder sweep**: no TODO, no `\textcolor{red}`, no "TBD", no stub table.

Reviewer verdict is written to `submissions/frame-a-eswa/REVIEW-<date>.md`. Only an
APPROVE (or APPROVE-WITH-FIXES with all fixes applied and re-verified) clears the rebuild-of-record.

---

## Stage 2 — MIX_C 33-cell box wave (REQUIRED)

**This is not optional prep. The rebuild-of-record does not exist without it.**

| field | value |
|---|---|
| Batch | 2 (per `PLAN-GAP-CLOSURE-MASTER-2026-07-31.md` §4) |
| Contents | MIX_C 33 cells (11 policies × 3 seeds) + the 3 quarantined MIX_A cells |
| Hardware | 2× 4090D, ~16 h wall |
| GPU-h | ~32 (≈29 MIX_C + ≈2.7 the 3 MIX_A) |
| Cost | ¥65–75 — **needs a per-batch user go, price restated at launch** |
| Unblocks | the Frame-A honesty gate (H14) |

Required before launch:
1. **Runner-stamp patch active** — every new cell must be written with a valid `runner_stamp`,
   or gate v2's `runner_stamp_v2` group FAILs them (they are all after the 07-27 cutoff).
2. **setsid/trap fix applied to `engine/run_mixc.sh`** (Phase 0-L item 1). The 07-29 Frame-A
   chain died on its 4th SIGTERM because `run_mixc.sh` runs python inside a `tee` pipeline with
   no `setsid`. MIX_C is currently 0/33 for exactly this reason. Apply the fix; do not rewrite
   the driver.
3. **Wave manifest + 2-card shard driver** — exists (Phase 0-L item 9); apply the setsid fix only.
4. **Smoke-gate covers every policy arm** (the 07-13 lesson: a smoke gate that skips a model or
   arm tests nothing).
5. **Model string check** — the gate and the analysis both pin `llama-3.2-1b`; a cell on any other
   checkpoint falls silently out of scope rather than failing. Verify before launch (spec
   conflict 11).
6. **Pull by manifest**, then re-run step (b) — gate v2 must go `INCOMPLETE` → `PASS`, and the
   `p2_llama-3.2-1b_real_MIX_C.json` artifact must be present (it is what makes exit 0 reachable
   at all — spec conflict 10).

What Stage 2 unlocks that Stage 1 cannot:
- Macro Group 7 (all MIX_C operating points).
- `\routerMajorityOnPrivacy` and the whole P2 structural claim.
- fig02's third patchwork panel and fig04 panel (c).
- `gates.*` per-mix coverage over all three mixes, hence the G-Q3 majority statement in the form
  the amendment expects.

**Stage-1 permission, stated once:** an internal MIX_A/B-only draft may be authored, reviewed,
and iterated in parallel with the Batch-2 wait. It carries the DRAFT banner, builds to
`main-draft.pdf`, and is never presented as a submission candidate.

---

## Spec conflicts found during runbook authoring

Recorded here rather than resolved unilaterally — several need a decision.

1. **MIX_C is out of the Q-frontier scope in code, but the paper needs it.**
   `q_ext_analysis.py:60` sets `MIXES = ("MIX_A", "MIX_B")` with the comment *"MIX_C hosts P2
   only (no Q frontier)"*, yet `macros.tex` carries `\qBothC/\qGraceC/\qEditC/\qRagC` and
   `make_figures_frame_a.R` builds a **1×3** Pareto patchwork including MIX_C.
   → Either MIX_C enters the Q_ext scope (an M6 amendment question, since the gate wording says
   "both mixes") or fig02 drops to 1×2 and the MIX_C Q macros are deleted. **Needs a decision
   before step (f).** The M6 gates as written are two-mix propositions; silently making them
   three-mix would change what G-Q1/G-Q3 assert.

2. **`\qGraceC/\pDeltaQgraceC` etc. currently have no valid source.** They are 07-21 fabricated
   rows. Any Stage-1 document that references a MIX_C macro is quoting a fabricated number even
   if the surrounding text is honest. → Stage 1 must not `\input` the MIX_C macro block at all.

3. **Capacity-budget ambiguity is unresolved in the M6 artifact itself.** The analysis JSON's
   own integrity notes flag it: the capacity budget is frozen at 200 while wave-1 streams are 500
   updates, so `capacity_frac` and `capacity_frac_alt_stream` disagree (the note cites 1.000 vs
   ~0.972 as a share-vs-utilisation distinction). The JSON says resolving it *"needs a USER
   decision on the budget (200 as frozen, or 500 = one per update)"*. It drives no verdict today,
   but the paper will have to state which denominator it reports. **User decision.**

4. **Latency provenance vs "real harness" framing.** Cells are named `..._real_...` and the paper
   describes a real replay harness, but `serve_overhead` is read from `SyntheticClock` — no
   backend overrides it, and the closed-form check reproduces every cell exactly
   (grace 325.50, rag 300.00). → The word "measured" must not attach to the latency term
   anywhere. Consider renaming the paper's description of that term to "modelled serving
   overhead" to avoid an overclaim a referee will find in one grep.

5. **`\routerGraceShare`/`\routerEditShare` are seed-0 reads asserted as seed-stable.** The old
   comment says "stable across seeds" without cross-seed numbers on the line. → Emit 3-seed
   mean + spread, or state seed-0 explicitly. Cheap to fix, and it is exactly the kind of
   unbacked stability claim the 07-26 red-team flagged elsewhere.

6. **`\qFloor` collapses three distinct policies** (`always_ft`, `always_reject`, `ft_merge`)
   into one number with a parenthetical range. Three policies with different mechanisms sharing a
   macro invites a wrong sentence. → Split, or report the range explicitly in the table.

7. **G-Q2's two readings are genuinely different propositions** and the code says so: the
   amendment's headline ("router dominates `always_grace` in ≥1 mix") and the operational phrasing
   ("`always_grace` must NOT dominate in both mixes") differ — a mutual non-dominance tie
   satisfies the second and fails the first. The code computes both (`passes` follows the
   amendment; `always_grace_dominates_both` carries the other). → The paper must quote **one**
   as the verdict and report the other as a companion. Pick the amendment's wording; do not let
   the looser reading become the headline.

8. **Old verdict artifact still on disk.** `results/frame_a/frame_a_verdict_ftfix.json` is
   quarantined (07-26: invalid for all three mixes) but is still the `% SOURCE` target of the
   current `macros.tex` Group-8/9 blocks. → Rename it `*.QUARANTINED-20260726` so no rebuild step
   can read it by accident.

9. **The gate does not cover MIX_A, and the cell count is 66, not 63.** `provenance_gate.py:38`
   sets `EXPECTED_MIXES = ("MIX_B", "MIX_C")`, so `EXPECTED_TOTAL = 11 × 3 × 2 = 66` and **MIX_A
   is out of scope entirely**. Two consequences:
   (i) The "gate v2 PASS on 63 cells" framing does not correspond to any quantity in the code —
   63 matches neither 66 (gate scope) nor 99 (full three-mix grid).
   (ii) The paper's primary tables are MIX_A, and MIX_A is the mix with 3 requarantined cells, yet
   nothing in the gate inspects them. A green gate would therefore certify the two mixes the paper
   leans on *least*.
   → **Decision needed:** extend `EXPECTED_MIXES` to all three (`EXPECTED_TOTAL` → 99) so the
   honesty gate actually covers the primary mix, or run a documented MIX_A-scoped pass. Do not
   publish a MIX_A table on cells no gate has vetted. This is the single most load-bearing conflict
   in this list.

10. **`PASS` is unreachable without MIX_C, by construction.** The exit-0 branch requires a
    `p2_ok` finding, and `NAMESPACED_P2_NAME = p2_llama-3.2-1b_real_MIX_C.json` is a MIX_C
    artifact. So `FRAME_A_GATE_V2_PASS.ok` cannot exist before Batch 2 completes — which is
    consistent with "the rebuild-of-record waits for MIX_C", but worth stating because it means
    there is no partial-credit gate state between `INCOMPLETE` and full Stage 2.

11. **Ungated `EXPECTED_MODEL`.** Both the gate and the analysis pin `llama-3.2-1b`. If any
    Batch-2 cell is produced on a different checkpoint it will fall out of scope silently rather
    than FAIL. → Confirm the Batch-2 launch config's model string matches exactly before the wave
    starts; a mismatch costs the full ¥65–75.

---

## Step (fig) — figure regeneration + review-src sync (2026-08-08)

The honest-review manuscript `\input`s figures from **`figures-review-src/`**, but the
generators write to **`figures-src/`** with different names. The sync is a rename copy;
it was previously undocumented (drift hazard). Canonical procedure:

1. Regenerate (run from the package root `submissions/frame-a-eswa/`, NOT from `figures-r/`;
   the generators assume `HARNESS="../../edit-harness"` and `OUT_DIR="figures-src"`):
   ```bash
   Rscript figures-r/make_figures_framea_full.R      # 99-cell: F2 F3 F4 F5 F7 F8 F9 F10
   Rscript figures-r/make_figures_framea_partial.R   # F1 routing, F6 spread (single-mix panels)
   ```
2. Sync into `figures-review-src/` with the rename map (prose stream names are baked into
   the plots as of 2026-08-08: MIX_A=steady, MIX_B=higher-churn, MIX_C=privacy-tagged):
   ```
   figF2_pareto        -> figF2_pareto
   figF3_predictor     -> figF3_predictor
   figF4_gates         -> figF4_gates
   figF5_ragcost       -> figF5_ragcost
   figF7_mixc          -> figF7_privacy_tagged
   figF8_failure_modes -> figF8_failure_modes
   figF9_mixb          -> figF9_higher_churn
   figF10_governance   -> figF10_governance
   figF1_routing_problem -> figF1_routing_problem   (from partial)
   figF6_mixa_spread   -> figF6_steady_spread        (from partial)
   ```
3. `pdflatex main-honest-review.tex` and eyeball the rendered figure pages.

Note: `figures-r/build.sh` invokes the legacy `make_figures_frame_a.R` (dead fig02-04 path),
NOT the partial/full generators above — do not rely on it for the review-src figures.
