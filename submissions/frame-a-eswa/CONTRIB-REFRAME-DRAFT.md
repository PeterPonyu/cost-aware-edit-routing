# Frame-A contribution reframing — conditional pre-draft (2026-07-31)

**Purpose.** The old PDF is quarantined and the rebuild is mechanical once cells land
(`REBUILD-RUNBOOK.md`). What is *not* mechanical is the paper's claim shape: the pre-registered
verdict is a KILL (the router does not Pareto-dominate `always_grace`), and whether a `Q_ext`
frontier may be published at all depends on G-Q1. This document pre-drafts both branches so no
number-driven pressure reshapes the story after the fact.

**Standing rules for both branches.**
- No `Q_ext` number is computed here. Every `[Q_ext:...]` slot is a placeholder filled from
  `results/frame_a/Q_ext_analysis_20260731.json` after the M6 reviewer recompute.
- `Q` is the **pre-registered primary**. `Q_ext` is **post-data and exploratory**, everywhere,
  including the abstract.
- Truth-first: the KILL is the headline finding, reported plainly. Packaging follows truth; a
  KILL is a finding, not a failure of the wave.
- MIX_C-dependent claims (the routing-majority-on-privacy structural result) are Stage-2 only.
  Both branches below mark them.

---

## Branch 1 — G-Q1 PASSES (a `Q_ext` frontier is publishable)

Meaning: under the extended metric, no policy exceeds the oracle in any of the 27 settings, in
both mixes. The metric is coherent, so `Q_ext` verdicts (G-Q2/G-Q3) are quotable and a frontier
figure may be produced.

Sub-cases the abstract must handle:
- **1a — G-Q2 PASSES**: charging for capacity/latency/staleness flips the comparison; the router
  Pareto-dominates the codebook in at least one mix. This is a *conditional, exploratory* win —
  never the headline, because the pre-registered metric said otherwise.
- **1b — G-Q3 TRIGGERED**: `always_grace` still dominates both mixes across a grid majority. The
  honest negative holds even under a metric built to charge it. The strongest version of the
  paper.

### Abstract skeleton (Branch 1)

> Continual knowledge maintenance of a deployed language model requires choosing, per incoming
> update, among weight editing, adapter-based storage, retrieval augmentation, fine-tuning, and
> refusal. We build a replay harness that measures — rather than assumes — the quality and
> serving cost of each mechanism over streams of [N] updates on a [1B-parameter] model, under
> [two/three] workload mixes and three arrival orderings, and we evaluate eleven admission
> policies including a damage-and-cost router, single-mechanism baselines, and an oracle that
> routes on true measured damage.
>
> Our pre-registered comparison returns a negative result: the router does **not** Pareto-dominate
> the single best fixed mechanism. An adapter-based codebook, whose collateral damage is
> identically zero by construction, matches or beats every routing policy on the pre-registered
> quality composite ([Q]: Δ = [\pDeltaQgraceA] with CI [...] in the first mix, a statistical tie in
> the second). We report this plainly and diagnose why: a composite that scores locality but not
> capacity, serving overhead, or staleness hands a zero-damage mechanism an uncontestable share of
> the score — visible in the fact that the codebook outscored the oracle that defines the ceiling.
>
> We therefore extend the composite, in a pre-registered post-data amendment, to charge each
> mechanism for capacity consumed, serving overhead, and end-of-stream staleness. Under the
> extended (exploratory) score the comparison [1a: reverses in [mix]: the router dominates the
> codebook, Δ = [Q_ext:...] / 1b: **does not** reverse: the codebook dominates in both mixes in
> [Q_ext:k]/27 sensitivity settings], so on this stream [1a: routing pays once serving costs are
> priced / 1b: a zero-damage codebook is the right policy and routing does not pay].
>
> Independent of the routing verdict, two measurements stand. First, retrieval augmentation costs
> [\cratioRag / ~17×] the per-query serving overhead of editing or codebook insertion — an
> ordering that inverts the usual assumption that retrieval is the cheap mechanism. Second, a
> geometric pre-edit damage predictor attains recall@decile of [\discRecall] (CI [...]; lift
> [\discLift]× over chance) at a measured ceiling of [\discCeil], establishing that damaging
> updates are identifiable before the edit is applied. We release the harness, the eleven
> policies, and the per-cell measurements.

*(Bracketed items are macro slots; `[Q_ext:...]` slots wait on the analysis artifact.)*

### Three contributions with honest scope (Branch 1)

**C1 — A measured operating-point frontier for knowledge-maintenance mechanisms, with a
pre-registered negative routing result.**
*Scope, honestly stated:* one 1B model, one editor family per mechanism, streams of [N] updates,
[two/three] mixes × three orderings, single hardware profile. The frontier is a measurement of
*these* mechanisms under *this* harness, not a general claim about routing. The negative result is
what the pre-registered gate returned; we do not re-scope the gate to convert it. Serving-latency
terms are modelled from a frozen clock, not wall-clock measured — disclosed at every point of
use, and bounded to at most [\qextLamLat] of the extended composite.

**C2 — A pre-edit damage predictor that identifies damaging updates before they are applied.**
*Scope, honestly stated:* recall@decile [\discRecall] with CI [...]; the point claim is
**suppressed** because n = [\discN] damaging ground-truth updates falls below our pre-registered
power floor of 50 — we report the interval only. The measured ceiling is [\discCeil]. **The
predictor's evidence base is shared with a separate submission by the same author** (see the
disclosure paragraph below); it is not independently re-derived on this paper's stream, and we
flag that as a limitation rather than presenting the two results as mutual replication.

**C3 — A cost inversion: retrieval augmentation is the expensive mechanism, not the cheap one.**
*Scope, honestly stated:* measured per-arm serving-cost ratios ([\cratioRag] for retrieval vs
[\cratioEdit] / [\cratioGrace] for editing and codebook insertion), over [\nqRag] / [\nqEdit] /
[\nqGrace] measured queries per arm. This is a property of our retrieval implementation and index
size, not a universal ordering; we state the configuration and the query volumes so the ratio can
be re-derived or contradicted.

**(Stage 2 only, C1 rider)** In the privacy-weighted mix the router assigns the editing arm to a
[\routerMajorityOnPrivacy] fraction of privacy-tagged updates, below the [\routerMajorityThreshold]
structural threshold — a descriptive routing-behaviour result, not a performance claim.

---

## Branch 2 — G-Q1 FAILS (no `Q_ext` frontier may be published)

Meaning: under the extended metric, some policy *still* exceeds the oracle in at least one of the
27 settings. Then `Q_ext` is also mis-specified. M6 is explicit: report that, do **not** proceed to
quote G-Q2/G-Q3, and do **not** publish a frontier under `Q_ext`. The gate driver stamps the
suppressed verdicts `verdict_suppressed_by_G_Q1`; they exist for the record only.

This branch is *narrower but not weaker*. The paper becomes a measurement-and-metric-critique
paper: the routing question is answered negatively on the pre-registered metric, and the attempt
to rescue the comparison with a better metric is reported as a documented failure with its worst
violator named. That is a more useful contribution than a frontier nobody should trust.

### Abstract skeleton (Branch 2)

> Continual knowledge maintenance of a deployed language model requires choosing, per incoming
> update, among weight editing, adapter-based storage, retrieval augmentation, fine-tuning, and
> refusal. We build a replay harness that measures the quality and serving cost of each mechanism
> over streams of [N] updates on a [1B-parameter] model, under [two/three] workload mixes and
> three arrival orderings, and evaluate eleven admission policies including a damage-and-cost
> router, single-mechanism baselines, and an oracle that routes on true measured damage.
>
> The pre-registered comparison returns a negative result: the router does **not** Pareto-dominate
> the single best fixed mechanism. An adapter-based codebook, whose collateral damage is zero by
> construction, matches or beats every routing policy ([Q]: Δ = [\pDeltaQgraceA], CI [...] in the
> first mix; a tie in the second). We further show this negative is partly an artifact of how such
> composites are built: the codebook outscored the damage-and-cost oracle that defines the ceiling
> ([\qGrace] vs [\qOracle]), which is self-refuting — a policy cannot exceed a bound it is
> measured against.
>
> We attempted to repair the composite by charging every mechanism for capacity consumed, serving
> overhead, and end-of-stream staleness, with weights frozen in a pre-registered amendment before
> any number was computed. **The repair fails its own sanity gate**: [Q_ext: policy] still exceeds
> the oracle by [Q_ext: margin] at [Q_ext: setting]. We therefore report no frontier under the
> extended score and quote no verdict from it. The negative routing result on the pre-registered
> metric stands; the diagnosis is that scoring mechanisms whose failure modes are structurally
> different requires a cost model that our composite — and, we argue, composites of this family —
> does not supply.
>
> Two measurements are unaffected by the metric question. Retrieval augmentation costs
> [\cratioRag / ~17×] the per-query serving overhead of editing or codebook insertion. And a
> geometric pre-edit damage predictor attains recall@decile [\discRecall] (CI [...]; lift
> [\discLift]×) at ceiling [\discCeil], so damaging updates are identifiable before application.
> We release the harness, the policies, and the per-cell measurements.

### Three contributions with honest scope (Branch 2)

**C1 — A measured mechanism frontier plus a documented failure of composite scoring for
heterogeneous update mechanisms.**
*Scope, honestly stated:* the pre-registered negative result is the verdict. The metric repair is
reported as an attempted fix that failed its pre-registered sanity gate, with the violating
policy, the margin, and the setting named. We do not report the repaired metric's rankings, and we
do not extend the metric further — the amendment forbids iterating until a router win appears, and
we abide by that in print. The value here is a negative result about *evaluation design*, backed
by a self-refuting-ceiling diagnostic any future benchmark can run.

**C2 — pre-edit damage predictor.** Identical scope and disclosure as Branch 1 C2. The predictor
result does not depend on the composite at all — it is a ranking claim against measured damage —
so it survives the metric failure intact. State that explicitly, so a referee does not assume the
metric problem contaminates it.

**C3 — retrieval cost inversion.** Identical scope as Branch 1 C3. Also composite-independent:
these are measured per-arm serving costs and query volumes, not composite scores.

**(Stage 2 only, C1 rider)** Same routing-behaviour descriptive as Branch 1.

---

## The M6 post-data disclosure paragraph (shared by both branches, verbatim in the paper)

Place at the head of the metric section, **not** in an appendix, and reference it from the
abstract's metric sentence.

> **Disclosure: the quality composite was amended after the data were collected.** Our
> pre-registered composite scores each policy on cumulative accuracy, ripple locality, and
> collateral-damage locality, with locality weighted [0.30]. After all [63/99] measurement cells
> were complete, we found two defects. First, the adapter-based codebook does not modify weights:
> it stores key–value entries at inference time, so its collateral damage is *identically* zero by
> construction and its locality term is exactly [1.0000] — not approximately. The [0.30] locality
> weight is therefore a fixed allotment no weight-modifying mechanism can contest, which is the
> definition of a method leaking into the score rather than a measurement of its merit. Second,
> and decisively, the always-codebook policy scored [\qGrace] against the damage-and-cost oracle's
> [\qOracle]. The oracle routes on true measured damage and true measured cost and is by
> construction the ceiling; a policy above the ceiling proves the composite is not measuring what
> the oracle optimises. Any Pareto claim resting on that frontier is an artifact, and we withdraw
> the corresponding claim from the earlier version of this work.
>
> We therefore pre-registered an amendment, **before computing any value under it**, extending the
> composite with three penalties that charge each mechanism for what it actually costs in
> deployment: capacity consumed (codebook entries against a fixed budget), serving overhead per
> query, and end-of-stream staleness. The penalty weights ([\qextLamCap], [\qextLamLat],
> [\qextLamStale]) were frozen in the amendment together with a [\qextGridN]-setting sensitivity
> grid and four gates: a sanity gate (no policy may exceed the oracle under the extended score, in
> any setting, in both mixes), a primary gate (does the router dominate the codebook in at least
> one mix), an honest-negative gate (if the codebook still dominates across a grid majority, that
> is the finding and the metric will not be extended further), and a no-op gate (both scores
> reported side by side for all eleven policies, with any rank shift beyond two positions
> explained in text).
>
> **The amendment is post-data and we treat it as such throughout.** The original composite
> remains the pre-registered primary and is reported beside the extended score in every table; all
> extended-score values are labelled exploratory. One further caveat binds the serving-overhead
> term specifically: it is derived from a frozen clock model rather than wall-clock measurement on
> the replay path, so we describe it as modelled, never measured. It contributes at most
> [\qextLamLat] of the composite; the capacity and staleness terms are unaffected. Two of our
> three penalty terms are additionally conservative in the codebook's *disfavour* — staleness is
> read at end of stream, so fine-tune forgetting is charged twice and outright-rejected updates
> read as fully stale — and we report that direction rather than tuning it away. The three penalty
> fractions were each recomputed from the persisted per-cell numerators and denominators by a pass
> that did not write the scorer, before any value entered this manuscript.
>
> [Branch 1 closing:] Under the extended score the sanity gate passes, so we report the extended
> frontier as an exploratory companion to the pre-registered result. [Branch 1a: the primary gate
> passes in [mix] — routing pays once serving costs are priced, on this stream, under an
> exploratory metric. / Branch 1b: the honest-negative gate triggers in [k]/[\qextGridN] settings —
> on this stream a zero-damage codebook is the right policy and routing does not pay, and we
> report that as the finding.]
>
> [Branch 2 closing:] Under the extended score **the sanity gate fails**: [policy] still exceeds
> the oracle by [margin] at [setting]. The extended composite is therefore also mis-specified. Per
> the amendment we publish no frontier under it and quote none of its rankings as verdicts; the
> values are retained in the released artifacts for the record only. The pre-registered negative
> routing result stands on its own metric, and the metric failure is itself reported as a finding
> about composite scoring of heterogeneous mechanisms.

---

## FA-3 shared-evidence-base cross-reference (both branches)

Already present as **Limitations item 3b**; this is the canonical wording to keep it consistent
with the abstract's C2 scope sentence. Written anonymity-safe (no venue, no title, no author
identity) so it survives double-anonymous review and the leak sweep.

> **Shared evidence base for the damage predictor (Limitations 3b).** The geometric pre-edit
> damage predictor evaluated here is not developed in this paper. Its ranking coefficient and the
> layer at which it is measured are taken from a separate manuscript by the same author, currently
> under review, and both papers read the *same* set of pre-edit key matrices and post-edit damage
> measurements for that quantity. The two results are therefore **not independent replications**,
> and we do not present them as mutual confirmation. What this paper contributes on that axis is
> narrower and stated as such: that a predictor with this ranking behaviour, applied as an
> admission filter inside a maintenance stream, recovers damaging updates at recall@decile
> [\discRecall] (interval only; n = [\discN] is below our pre-registered point-claim floor of 50).
> An independent re-derivation on this paper's own stream would strengthen the claim and is left to
> future work. We disclose the overlap here rather than in a data-availability note because it
> bears directly on how much of the routing story the predictor can be said to support.

Cross-reference wiring (three places, all required so the disclosure cannot be missed):
1. Abstract / C2 scope sentence — one clause: "whose evidence base is shared with a companion
   submission (Limitations 3b)".
2. The predictor's results subsection — a forward pointer at the first appearance of
   [\discRecall].
3. Limitations 3b — the paragraph above, in full.

Leak-sweep note: this paragraph deliberately names *another manuscript*. That is a required
disclosure, not a leak. It must not name the venue, the paper's internal codename, or any file
path. Keep "a separate manuscript by the same author, currently under review".

---

## Branch-selection procedure (so this is not a judgement call later)

1. Run step (c) of the runbook → `Q_ext_analysis_20260731.json`.
2. Read `gates.G_Q1.passes` — **that single boolean selects the branch.** Nothing else.
3. If `false`: use Branch 2, record `gates.G_Q1.per_mix.<mix>.worst_violator` (policy, margin,
   setting) into the Branch-2 abstract slots, and confirm no `Q_ext` frontier figure is generated
   (runbook step g gates [F4] on exactly this).
4. If `true`: use Branch 1, then read `gates.G_Q3.triggered` to pick 1a vs 1b, and
   `gates.G_Q2.passes` (amendment wording) as the primary verdict — **not**
   `always_grace_dominates_both`, which is the looser companion reading and must be reported
   beside it, never in its place.
5. Either way, `gates.G_Q4` supplies the side-by-side table and the flagged rank shifts; every
   flagged row gets a sentence.

## Open items this draft cannot settle

- **MIX_C's scope.** The analysis code restricts the frontier to two mixes (MIX_A + MIX_B); the
  figure generator and the macro set assume three. Both abstract skeletons say "[two/three]"
  because the answer changes what "in both mixes" means in G-Q1 and G-Q3. Decision needed before
  the metric section is written (runbook spec conflict 1).
- **The honesty gate does not cover MIX_A.** The provenance gate's scope is MIX_B + MIX_C only, so
  the mix carrying every primary table — and the 3 requarantined cells — is unvetted. Both
  branches' C1 rests on MIX_A numbers, so this must be closed before either abstract is written
  against real values (runbook spec conflict 9).
- **Capacity denominator.** 200 as frozen, or 500 (one per update)? The paper must name one; the
  analysis artifact flags it as a user decision (runbook spec conflict 3).
- **The word "real".** Cells are named `..._real_...` and the harness is described as a real
  replay, while the serving-overhead term is clock-modelled. Both branches already say "modelled",
  but the harness's own name invites the overclaim. Consider a terminology pass (runbook spec
  conflict 4).
