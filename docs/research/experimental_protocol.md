# Experimental Protocol — NR-SpecRAG

Revision: 2026-11-09. Supersedes the initial draft in full. Incorporates the approved working methodology (decision log D-003–D-030), including the owner-approved decisions E-2 (Δ = 5 pp, §9), E-3 (strict adequacy, §5) and E-13 (six-cell version design, §7). Open owner decisions (E-1, E-5, E-6, E-7, E-8, E-10) are marked inline where they bind. A pre-registration document (`docs/research/preregistration.md`) will be created and hash-locked before any test-set run; items marked "pre-registered" below are committed there.

## 1. Systems, data flow and reported conditions

System C is two independent modules:

- **Module S (sufficiency).** Input `(question, retrieved_context)`. **Must not receive any candidate answer.** Emits a continuous score `s ∈ [0,1]` and a rationale. Answer-blindness is a hard requirement: it makes `s` a legitimate gate over any arm's answer and makes the abstention ablation free.
- **Module V (verification).** Input `(question, retrieved_context, answer_B)`. Emits per-claim support verdicts, per-citation validity verdicts, and a revised answer `answer_Cv`. **V takes `answer_B`, never `answer_B+`** — otherwise C-v would embed two revision passes and the C-v vs B+ comparison would be void.

Retrieval and indexing are local (no API). Per question `q`, retrieval runs once; `K(q)` (chunk IDs, text, scores) is logged and shared by every arm that uses retrieved context, so retrieval variance cannot differ across arms.

```
q ──► [Retriever: hybrid BM25+dense, local] ──► K(q) + per-chunk scores ──► LOG

────────────────────── API calls (per answerable item: 6) ──────────────────────
R-A   in: (q)                                      out: answer_A        [closed-book]
R-B   in: (q, K)                                   out: answer_B
R-B+  in: (q, K, answer_B) + GENERIC critique      out: critique_B+, answer_B+
R-S   in: (q, K)              ** answer-blind **   out: s ∈ [0,1], rationale
R-V   in: (q, K, answer_B) + VERIFICATION prompt   out: per-claim verdicts,
                                                        per-citation verdicts, answer_Cv
R-O   in: (q, G(q) gold spans)                     out: answer_O        [answerable only]

──────────────── post-processing only (zero API calls) ────────────────
B-τ  = retrieval_score(K) < τ   ? ABSTAIN(template)               : answer_B
C-a  = s < τ_s                  ? ABSTAIN(template + S rationale) : answer_B
C-v  = answer_Cv                                     (no gate)
C    = s < τ_s                  ? ABSTAIN(template + S rationale) : answer_Cv
```

| ID | Condition | Context | Extra inference | Gate | Isolates / purpose | Tier |
|---|---|---|---|---|---|---|
| A | Closed-book | none | — | none | Parametric-contamination probe; stratifier for correctness analysis; H0c; version profile covariate | 1 (run) |
| B | Standard RAG | retrieved | — | none | Reference point | 1 (run) |
| B+ | Compute-comparable critique | retrieved | generic self-critique, evidence-agnostic | none | Kills the "more inference" confound; denominator for H1/H2 | 1 (run) |
| B-τ | Retrieval-score gate | retrieved | — | top-k score < τ | Kills the "cheap baseline suffices" confound; H5, H0b | 1 (free) |
| C-a | Abstention only | retrieved | Module S | s < τ_s | Isolates abstention from verification | 1 (free) |
| C-v | Verification only | retrieved | Module V | none | Isolates verification from abstention; H1, H2, H3 | 1 (free) |
| C | Full evidence-aware | retrieved | S + V | s < τ_s | The system under test; RQ1 | 1 (free) |
| O | Oracle retrieval | gold spans | — | none | Separates generation faithfulness from retrieval quality; the "correct evidence" intervention | 1 (run) |
| L | Long-context, no retrieval | full corpus | — | none | Optional | 2 |
| Loc | Small local model | retrieved | S + V | s < τ_s | Optional | 2 |

**Compute-comparability of B+ vs V (comparable by construction, verified by measurement — not "matched"):** same call count (1 each); identical input payload `(q, K, answer_B)` with prompt templates trimmed to within ~10% token length of each other; B+ emits a structured critique block before its revised answer, mirroring V's verdicts-then-revision shape, so the manipulated variable is evidence-awareness, not structured self-review. B+ sees K — a context-free B+ would confound evidence-awareness with access to evidence during revision. Residual non-matching (model-controlled internal reasoning tokens) is disclosed; measured input/output tokens per arm are reported from usage fields. Deliberately unmatched comparisons are stated as such: C (2 calls) vs B+ (1 call) — the full system is allowed to cost more and its cost is an RQ1/RQ4 axis; C-a (1 S call) vs B-τ (0 calls) — the asymmetry is the research question.

**Reproducibility audit of the derived (zero-extra-call) conditions:**

| Condition | Required logged artifacts | Reproducible from logs? |
|---|---|---|
| B-τ | per-chunk retrieval scores; answer_B | yes — pure function of logs |
| C-a | s; answer_B | yes |
| C-v | answer_Cv | yes (it is a logged artifact) |
| C | s; answer_Cv | yes |
| threshold sweeps | same | yes — the answered text at any threshold is always answer_B or answer_Cv, both labelled once; only the answered/abstained partition moves |

A gated abstention has no model-generated answer text, so the gate abstention message is a fixed template plus Module S's rationale; the ABSTAIN-REFERRED / ABSTAIN-BARE sub-code is coded from that rationale.

**Answer schema.** All answers conform to one structured schema: an array of atomic claims, each with zero or more citations of the form (document, version, clause). Arm A uses the same schema with citations optional; any citation it emits from memory is resolved against the corpus, giving a closed-book hallucinated-citation rate at no extra cost. Schema enforcement is provider-agnostic: structured outputs where available, otherwise schema-by-prompt plus a local validator with one re-prompt on validation failure; items failing twice are logged as format failures (a reportable behaviour, not a crash).

## 2. Benchmark composition (provisional)

Allocation is driven by which cells carry the hypotheses, not by category symmetry.

| Code | Class | n | Construction method |
|---|---|---|---|
| S | Answerable, stable, single-clause | 24 | Clause sampled from the pinned corpus |
| M | Answerable, stable, multi-clause | 16 | Requires ≥2 clauses to answer |
| V | Answerable, version-divergent | 16 | Clause whose content differs between the two stored versions (Gate 1a) |
| R | Referentially deferred | 12 | Corpus cross-references an external TS |
| X | Out of scope | 22 | By construction: content owned by a TS deliberately excluded from the corpus, so non-answerability follows from corpus scope rather than from proving absence over 400 pages |
| | **Test total** | **90** | Answerable (S+M+V) = 56 · abstention-relevant (R+X) = 34 |
| | Dev/pilot | 20 | ~4 per class; codebook freeze, prompt development, threshold selection; excluded from primary analysis |

**n_test = 90 is provisional within [70, 120].** It is committed only when all five evidence artifacts exist, are hashed into the pre-registration document, and jointly support it:

1. `version_divergence_audit.csv` (Gate 1a): ≥16 suitable divergent clauses + 4 spares; shortfall triggers the pre-registered reallocation (V deficit moved to S/M at 60/40) and the C1 downgrade note.
2. Modality density memo (Gate 1b): ≥25 answerable items can carry a modal-bearing required fact, with the 30-clause load-bearing check; else C2 downgrades to taxonomy-only.
3. Measured annotation rates from the fully annotated pilot (min/item per class, claims/answer, labels/hour) → recomputed total-hours forecast, checked against the annotation-hours ceiling **H_max (open decision E-10)**.
4. Pilot variance and simulation outputs: σ̂_d(CAF), σ̂_d(GFR), base rates; simulated H1 power at the pilot effect and at half of it; P(inconclusive H2) at the fixed Δ = 5 pp for n ∈ {70, 90, 110, 120} — reported as a forecast, with Δ immovable (§9).
5. Retrieval-sufficiency distribution over all drafted answerable items (retrieval-only, local, free): sizes the natural RS=0 cell and fixes k before any test generation.

Permissible reasons to move n_test: the hours ceiling (3), gate yields (1, 2), or forecast inconclusiveness (4). Impermissible: observed effect directions on any test-relevant comparison, and anything downstream of test-set generation. Any post-commitment change is a logged protocol deviation.

## 3. Gold annotation

For each question, store: question ID and text; class (S/M/V/R/X); corpus answerability label (CONTAINED / PARTIAL / REFERRED / OUT_OF_SCOPE); the minimal gold clause set (used to compute retrieval sufficiency); expected answer; required atomic facts, each flagged if modal-bearing; gold specification, version, clause and exact evidence span per required fact; acceptable alternative formulations; for V items, the divergent fact pair (f_old, f_new) and divergence type from the Gate 1a audit; for R items, the external specification referred to; for X items, the construction note (which excluded TS owns the content); notes and adjudication history.

Retrieval sufficiency is computed, not judged: RS = 1 iff the retrieved context contains the minimal gold clause set. The (OUT_OF_SCOPE, RS=1) cell is definitionally empty; any occupancy is a gold-label error and triggers re-adjudication (audit check).

Annotation of system outputs is blinded to system identity; abstention texts are normalised before labelling. Where a second annotator is available (open decision E-6), they independently label a stratified 25% subsample; disagreements are adjudicated and documented; inter-annotator agreement (Krippendorff's α) is reported per label type, including the CITED-PARTIAL materiality split. If no second annotator is available, the fallback is a blinded intra-annotator retest (re-label a 20% subsample ≥2 weeks later), disclosed as a limitation. Claim extraction mode is open decision E-7 (recommended: LLM-assisted extraction with 100% human verification on test items). The codebook is frozen after the pilot; annotating against an unfrozen codebook is not permitted.

## 4. Claim and citation coding

Claims are atomic propositions extracted from the answer by a pre-registered procedure, capped at 6 per answer.

**Split rule.** A partially supported claim was, by definition, not atomic: the annotator must split it; the supported part is re-judged and the unsupported residue becomes its own claim, coded CITED-UNSUPPORTED (entering evidence-breach risk). CITED-PARTIAL survives only for genuinely unsplittable cases.

**Status assignment (forced choice, exhaustive, no skip code).** For each factual claim c:

```
1. No citations attached            → UNCITED
     judged against E_ctx:            UNCITED-GROUNDABLE   (support was in context: lazy attribution)
                                      UNCITED-UNGROUNDABLE (no support in context)
2. Citations attached, none resolve → ALL-INVALID
     judged against E_ctx:            ALL-INVALID-GROUNDABLE / ALL-INVALID-UNGROUNDABLE
3. ≥1 citation resolves             → judge c against the UNION OF c's OWN resolved spans:
                                      CITED-SUPPORTED | CITED-PARTIAL(-MATERIAL|-MINOR) |
                                      CITED-UNSUPPORTED | CITED-CONTRADICTED
4. Too vague to verify              → VAGUE (can never be CITED-SUPPORTED)
```

**Multiple citations on one claim:** the claim-level verdict is judged against the union of that claim's own resolved spans — two individually partial citations that jointly entail the claim yield CITED-SUPPORTED. Separately, every (claim, citation) instance receives its own verdict (supports / partial / none / contradicts), so redundant and freeloading citations are visible at the citation level without distorting the claim level. Every non-supporting instance receives exactly one taxonomy code: **E1** non-existent clause ID · **E2** wrong document · **E3** right document, wrong clause · **E4** right clause, wrong version · **E5** resolves and is topically related but does not support the claim.

**Unsplittable CITED-PARTIAL — auditable materiality rule.** The annotator records two fields: the **supported core** (a rewrite containing only what the cited evidence carries) and the **unsupported delta** (the element present in the claim but absent from the evidence), tagged with one delta type: (a) scope broadening · (b) condition removal or weakening · (c) value/parameter alteration (number, range endpoint, unit) · (d) normative strengthening · (e) entity substitution or addition. The delta is **material** iff it itself contains a checkable technical proposition — i.e., it is of type (a), (b), (c) or (e). CITED-PARTIAL-MATERIAL enters evidence-breach risk; CITED-PARTIAL-MINOR (delta carries no checkable technical content) does not. Both remain failures in strict attribution risk, which is unchanged. **Type (d) carve-out, pre-registered:** a claim whose only delta is normative strength is coded CITED-SUPPORTED under the modality-blind convention and flagged M1–M4; the modality-aware recomputation then flips it. If strength-only deltas entered PARTIAL-MATERIAL they would leak into baseline R_EB and partially erase the modality gap — the quantity the C2 experiment measures. Strength deltas combined with an a/b/c/e delta code on the a/b/c/e ground. The (core, delta, delta-type) triple lets a second annotator re-derive each materiality decision; agreement on the MATERIAL/MINOR split is reported in the IAA subsample.

**Complete status mapping — every status, mutually exclusive, forced choice:**

| Claim status | In R_EB? | In R_strict? | Also surfaces in |
|---|---|---|---|
| CITED-SUPPORTED | no | no | — |
| CITED-PARTIAL-MATERIAL | **yes** | yes | materiality audit fields |
| CITED-PARTIAL-MINOR | no | **yes** | PARTIAL-MINOR-rate monitor |
| CITED-UNSUPPORTED | **yes** | yes | CSP, E2–E5 |
| CITED-CONTRADICTED | **yes** | yes | CSP, E2–E5 |
| UNCITED-GROUNDABLE | no | **yes** | uncited-claim rate |
| UNCITED-UNGROUNDABLE | **yes** | yes | uncited-claim rate |
| ALL-INVALID-GROUNDABLE | no | **yes** | resolution rate, E1 |
| ALL-INVALID-UNGROUNDABLE | **yes** | yes | resolution rate, E1 |
| VAGUE | no | **yes** | VAGUE-rate monitor |

**Anti-disappearance rules** (PARTIAL-MINOR and VAGUE sit outside R_EB and are the only potential laundering routes): per-arm PARTIAL-MINOR and VAGUE rates are reported next to every R_EB figure with a pre-registered red flag — if either rate differs across compared arms by more than 5 pp absolute, the R_EB comparison is flagged as potentially laundered and the R_strict comparison governs. Invalid citations cannot vanish by construction: they are counted in citation resolution rate and E1 regardless of claim-level status, and their claims land in one of the two risks via the groundedness sub-code.

## 5. Outcome coding and adequacy

Outcome (per question × condition), assigned on the core proposition:

- **ANSWER**: ≥1 required atomic fact asserted without an evidential caveat.
- **QUALIFIED**: ≥1 required fact asserted and an explicit evidential caveat attached (evidence incomplete, version uncertain, outside indexed scope). Generic epistemic hedging ("typically", "generally") is not an evidential caveat; it codes as ANSWER.
- **ABSTAIN**: no required fact asserted. Sub-coded ABSTAIN-REFERRED (names the correct external specification — for gate abstentions, coded from the template + S rationale) vs ABSTAIN-BARE.

**Adequacy (strict — approved as decision E-3).** An answered question is adequate iff GFR = 1 on required facts, **and** no required fact is asserted incorrectly, **and** the answer contains no incorrect extra claim. Partial credit lives in mean GFR, not in the adequacy bit.

**Incorrect extra claim — bounded, auditable test.** An extra claim is incorrect iff any of: (i) it is coded CITED-CONTRADICTED (already annotated, zero extra cost); (ii) it contradicts the question's gold answer or a required gold fact (checkable against the gold record); (iii) the annotator can name a specific clause of the pinned corpus that refutes it and records that clause ID (opportunistic; no search obligation). Extra claims failing all three routes are coded TRUTH-UNKNOWN: they do not block adequacy, but they still enter CAF and the risks if unsupported. Rationale for the asymmetry, stated: adequacy measures delivered correct content; attribution hygiene is measured by CAF/R_EB/R_strict; letting every unverifiable extra claim void adequacy would double-count attribution failures inside the completeness metric.

**Extra-claim pipeline** (every claim that is not a required fact): 1. segmentation (split rule applies); 2. gold alignment (a paraphrase of a gold fact counts as that fact and toward GFR; duplicates fold; non-matching claims are "extra"); 3. full status coding (§4 — extra claims are first-class citizens of CAF and both risks); 4. correctness screen (three-route test; outcome ∈ {incorrect, truth-unknown} recorded with the evidence route); 5. modality flag if modal-bearing.

**Two coverages, always reported together:** response coverage = (ANSWER + QUALIFIED) / all questions; adequate-answer coverage = adequate answers / all questions. A 1-of-4-facts answer codes ANSWER, counts in response coverage, scores GFR = 0.25, is inadequate, and is excluded from adequate-answer coverage — visible as a committed, honest, incomplete answer rather than laundered into coverage.

## 6. Metrics

Unit of a "citation" everywhere: the (claim, citation) instance — the same clause cited for two claims counts twice, because it makes two support promises. Aggregation: macro (mean of per-question proportions) primary, micro alongside; uncertainty by cluster bootstrap resampling questions (§9).

| Metric | Numerator | Denominator | Undefined when |
|---|---|---|---|
| **CAF** — claim attribution failure rate (primary) | claims not CITED-SUPPORTED | factual claims in the answer | 0 factual claims (excluded; captured by outcome code) |
| Evidence-breach rate | claims in the R_EB set (§4 table) | factual claims | same |
| Uncited-claim rate | UNCITED claims | factual claims | same |
| Citation resolution rate | resolving citation instances | citation instances emitted | 0 citations (report count) |
| **CSP** — citation support precision | instances supporting their claim | resolving instances | 0 resolving instances |
| USCR_ctx (secondary) | claims unsupported by full supplied context | factual claims | — |
| **GFR** — gold-fact recall | required gold facts correctly asserted | required gold facts for q | — |
| **CY** — claim yield | extracted claims (count) | per answer | — |
| Correctness | answer agrees with gold (binary) | per question | — |

CAF decomposes exactly into the failure families (uncited, invalid citation, non-supporting citation, unsupported/partial/vague claim), so the primary number and its error taxonomy are the same measurement. Guards: hedging-by-omission drops GFR (fails H2); hedging-by-vagueness raises CAF (VAGUE can never be CITED-SUPPORTED); claim-count collapse is monitored by CY with a pre-registered red flag (relative drop > 25% vs B).

**Selective prediction.** Two risks, both computed mechanically from claim labels, both reported as full risk–coverage curves with AURC and risk at matched coverage (pre-registered at coverage 0.60 and 0.80), always together:

- **R_EB — evidence-breach risk.** Error event: the question was answered and the answer contains ≥1 claim in the R_EB set (§4 table) — the "reader misled" risk; designated confirmatory risk for the gating hypotheses.
- **R_strict — strict attribution risk.** Error event: the question was answered and ≥1 factual claim is not CITED-SUPPORTED. Pre-registered caveat: R_strict can saturate near 1.0 if uncited claims are routine; mean CAF among answered questions is co-reported so the curve stays informative at the ceiling.

Also from the answerability × sufficiency cross-tabulation: over-abstention rate (abstain despite answerable + RS=1) and ungrounded-answering rate (answer despite RS=0 or out-of-scope). Cost and latency: tokens in/out per condition from usage fields; wall-clock p50/p95 over repeated runs on an idle machine; reported per condition.

**Version-conditional metrics** (defined only on the controlled-context sub-experiment, §7; each on exactly one cell family, so no behaviour is credited or penalised twice):

| Metric | Definition | Cells |
|---|---|---|
| **VFR** — version-faithful response rate | P(asserts f_T \| answered), T the explicit target | all six |
| **CVS** — cross-version substitution rate | P(asserts f_¬T \| answered), reported per cell and per direction | all six |
| **VCDR** — version-conflict detection rate | P(output explicitly notes that divergent versions exist) | both-context cells (5, 6) only |
| Version-mismatch abstention/qualification rate | P(ABSTAIN or QUALIFIED on version grounds) | misaligned cells (3, 4) only |
| **VAA** — version-attribution accuracy | citations whose version matches the version of the span actually supplied / resolvable citations | all six |

**Normative-modality metrics.** **MPA** (modality preservation accuracy) = claims whose normative strength matches the supporting span's / claims carrying a normative modal and otherwise supported — the conditioning isolates compliance-relevant errors that entailment-style judging passes. Error codes: **M1** strengthening (may/should → shall) · **M2** weakening (shall → should/may) · **M3** modality dropped · **M4** modality invented. **Modality gap** = modality-aware CAF − modality-blind CAF (pp), per arm, where the modality-aware recomputation flips strength-only-delta claims from CITED-SUPPORTED to CITED-UNSUPPORTED (§4 type-d carve-out). The modality gap is the experiment that tests contribution C2, not a side metric.

## 7. Version-conditional sub-experiment (six-cell design — approved as decision E-13)

**Index policy.** The main benchmark retrieval index contains the pinned primary version only. The second version is quarantined in a separate store used exclusively for constructed contexts; standard retrieval can never surface it.

Per V item, gold annotation fixes the divergent fact pair (f_old, f_new) from the Gate 1a audit. Every controlled-context question carries an explicit target version T ("According to [document] [version/release T] …"); question wording is a fixed pre-registered template identical across cells except the version token, so T is the only manipulated element of the question. Contexts are constructed from real clause text only.

**Design — full factorial, T ∈ {old, new} × S ∈ {old-only, new-only, both}, 16 items × 6 cells × arms {B, C}, plus closed-book A per target direction:**

| Cell | T | S | Correct behaviour | What it identifies |
|---|---|---|---|---|
| 1 | new | new-only | assert f_new | adherence baseline, new |
| 2 | old | old-only | assert f_old | adherence baseline, old |
| 3 | old | new-only | flag mismatch; abstain/qualify | mismatch handling; pull and context-following coincide |
| 4 | new | old-only | flag mismatch; abstain/qualify | mismatch handling; pull and context-following dissociate |
| 5 | new | both | assert f_new + note divergence | VCDR; adherence where pull agrees |
| 6 | old | both | assert f_old + note divergence | VCDR; adherence where pull opposes |

The rejected four-cell alternative (one misaligned direction, both-context at T=new only) confounds substitution direction with target direction: wherever adherence and a directional pull toward one version coincide, the mechanisms are unidentified, so it supports only an exploratory, direction-specific claim. The balanced design identifies directional pull as the f_new-assertion asymmetry across the S=old-only vs S=new-only cells and across cells 5 vs 6.

**Three orthogonal codes per output — adherence, detection and response are never conflated:**

1. **Adherence code:** asserts f_T / asserts f_¬T / asserts neither → VFR, CVS per cell. In the both-context cells, asserting f_¬T despite the explicit target is the most serious failure and is visible as CVS-despite-target.
2. **Detection code** (binary): explicitly notes that divergent versions exist → VCDR, defined only on cells 5–6 (the only cells where detection is possible from the supplied evidence).
3. **Response code:** ANSWER / QUALIFIED / ABSTAIN → version-mismatch abstention/qualification rate, defined only on cells 3–4 (the only cells where declining is the correct behaviour).

**Closed-book profile.** Arm A is elicited once per target direction (ask-old, ask-new; no context; 2 × 16 = 32 calls), giving the per-item, per-direction profile P ∈ {matches f_old, matches f_new, other, abstain}. A is never scored on VFR/CVS — those metrics are defined only for arms that receive context; A contributes the profile covariate and is evaluated on correctness only. **Behavioural version errors are never claimed to prove parametric-memory use.** The parametric interpretation enters only through the exploratory conditioning P(CVS | closed-book profile matches f_¬T) vs P(CVS | it does not), explicitly disclaimed: closed-book behaviour is a proxy, not a measurement of internal knowledge; substitution concentrated in one direction and aligned with closed-book profiles is consistent with — not proof of — parametric override, and symmetric substitution is consistent with confabulation.

**Call-count and cost accounting (incremental vs total, kept distinct).** Per cell-item: arm B is 1 call; arm C adds S and V on the shared base answer (3 calls total per cell-item). Totals: 16 × 6 × 3 = **288 generation calls ≈ $5–6** at the assumed token sizes (constructed contexts are 1–2 clauses), plus 32 closed-book profile calls (<$1). Increment of the six-cell over the four-cell design: 16 × 2 × 3 = **96 calls ≈ $1.50–2**. Annotation uses the three-code quick coding only (~1 min/output; full claim labelling remains confined to main-benchmark rows): 16 × 6 × 2 = 192 coded outputs ≈ 3.2 h, plus ~0.5 h for the 32 profile outputs; increment over four-cell: +64 coded outputs ≈ **+1.1 h**. Gold burden: zero (same 16 items, same (f_old, f_new) pairs).

The V items also appear once in the main benchmark via standard retrieval over the primary index, with gold defined against the pinned primary version. Main-run questions are version-unscoped, so those rows are reported descriptively and are **not** adherence measurements.

**Cut ladder position (if hours bind):** cut 2 = drop cells 5–6 (lose VCDR, keep balanced directional identification); cut 2a = drop cell 4 (directional claims degrade to exploratory, as above).

## 8. Evidence intervention conditions

Three conditions on a 24-item subset drawn from the stable-answerable cells, run on B, B+ and C:

1. **Correct supporting evidence** — this is arm O (gold spans). No new runs.
2. **Supporting evidence removed** — gold spans excluded from the retrieved set, same k.
3. **Related non-supporting evidence** — gold spans replaced with the highest-ranked retrieved chunks that annotation confirms support none of the required facts.

Conflicting evidence is NOT synthesised. Conflict is instantiated by the six-cell sub-experiment's both-context cells (§7), using both real versions of the divergent clause. No fabricated specification text is created, stored or published at any point.

The exact context string supplied to the model is recorded and hashed on every call, enforced by the harness.

## 9. Statistical analysis plan

### 9.1 Conventions

- **H1 (superiority, CAF):** paired, two-sided α = 0.05; cluster bootstrap resampling questions; claimed only if the 95% CI on the difference excludes 0 in the favourable direction. Wilcoxon signed-rank as sensitivity analysis.
- **H2 (non-inferiority, GFR):** one-sided α = 0.025. Decision rule: the **97.5% one-sided lower confidence bound** on the GFR difference (equivalently, the lower endpoint of the two-sided 95% CI; implemented as the 2.5th percentile of the cluster-bootstrap distribution) must exceed −Δ. Δ = 5 pp, fixed a priori (approved as decision E-2), justified in interpretable units: with 3–4 required facts per question, 5 pp ≈ one dropped required fact on roughly one answered question in five to seven. The supporting asymmetry argument (justifying a nonzero Δ, not its size): an omitted required fact is partially recoverable — the reader follows the citation and sees the full clause — whereas an evidence-breaching claim is not self-revealing. **Δ is never revised in response to pilot or test data, and is never chosen to achieve statistical power.**
- **Conjunction:** intersection–union test — the conjunctive claim is made only if both components pass, each at its own full level; no Holm correction. Rationale for 0.025 (not 0.05 one-sided): it matches the standard non-inferiority convention and makes ruling out harm exactly as stringent as demonstrating benefit.
- Secondary hypotheses (H3–H8): two-sided 95% CIs, uncorrected, labelled non-confirmatory. Competing hypotheses (H0a–H0c): reported regardless of direction. Category breakdowns: exploratory, descriptive.

### 9.2 Restricted conclusions when non-inferiority is not established (pre-registered verbatim)

| H2 outcome (97.5% one-sided lower confidence bound on the GFR difference vs −Δ) | Permitted claim |
|---|---|
| lower bound > −Δ | "Verification reduced attribution failures **without a meaningful loss of completeness** (non-inferiority at Δ = 5 pp demonstrated)." |
| entire CI below −Δ | "Verification **trades completeness for attribution**"; report the exchange rate as the RQ4 finding. H1 alone may not be presented as overall improvement. |
| bound straddles −Δ | **Inconclusive.** Permitted claim only: "CAF was reduced [if H1 held]; whether completeness was preserved within the pre-specified tolerance could not be determined at this sample size; observed GFR difference X [CI]." Explicitly banned: any "no evidence of harm" phrasing, and any headline conjunction. |

At feasible master's-scale n, the inconclusive outcome is the likely one unless σ_d ≤ ~0.15; this is stated in advance, not discovered later.

### 9.3 Power

Paired NI test, margin Δ, one-sided α = 0.025, power at true difference 0: required n = ((1.9600 + z_power) · σ_d / Δ)² = (2.8016 · σ_d / Δ)² for 80% power. At n = 56 answerable items:

| σ_d (GFR paired differences) | power, Δ = 5 pp | n for 80% power at Δ = 5 pp |
|---|---|---|
| 0.15 | 70% | 71 |
| 0.20 | 46% | 126 |
| 0.25 | 32% | 196 |
| 0.30 | 24% | 283 |

These z-based figures already correspond to α = 0.025 one-sided; the pilot simulation forecasts the same quantities under the exact bootstrap analysis. H1 minimum detectable paired effect at 80% power, two-sided α = 0.05: d_z = 2.8016/√n ≈ 0.374 at n = 56; the percentage-point translation depends on the unknown base rate CAF(B+) and per-question variance and is deferred to the pilot. σ_d values are assumptions until measured (with ~3 required facts, per-question GFR sits on {0, ⅓, ⅔, 1}; if 30–40% of questions differ by ±⅓, σ_d ≈ 0.18–0.21).

Abstention (H4): McNemar on ~34 R+X items is powered for large effects only (≈ ≥35 pp); moderate effects will be inconclusive — pre-registered limitation. The answerable-but-RS=0 cell's size is unknowable until the pilot (§10).

### 9.4 Pilot-based power analysis (forecast only — never moves Δ)

From the 20-item pilot (B, B+, S, V run and fully annotated; k = 3 repeats on the pilot itself), estimate: claims per answer; base rates CAF(B), CAF(B+); per-question paired differences and σ_d for CAF and GFR; within-question claim correlation; gold facts per question. Then: (1) fit a simple generative model (per-question claim count from the empirical distribution; per-claim failure probability per arm with a question-level random effect matching the observed ICC; between-arm correlation as observed); (2) simulate ≥2,000 synthetic benchmarks at each candidate n ∈ {56, 70, 90, 120 answerable}, applying the exact pre-registered analysis; (3) read off H1 power at the pilot effect and at half of it, P(inconclusive H2) at the fixed Δ = 5 pp, and the probability of an inconclusive conjunction; (4) guard against pilot noise by sizing at 1.2 × σ̂ (σ̂ from n = 20 carries a multiplicative 95% CI of roughly ×[0.76, 1.46]); (5) commit n and the simulation code + seed in the pre-registration document before any test item is run.

Pre-registered contingency (open decision E-8 — approve or strike before the pilot): if the pilot base rate CAF(B+) < ~10%, no feasible n detects a realistic absolute reduction, and the primary comparison pivots to the intervention conditions, where failure rates are elevated by construction.

### 9.5 Discipline

Predefine primary outcomes and comparisons before inspecting final test results. No threshold tuning on the final test set: operating thresholds (τ, τ_s) are selected on the dev set; test-set results are reported as full curves plus the pre-registered matched-coverage points. Retrieval depth k is pre-registered before test generation. Nondeterminism: if the k = 3 subsample shows material run variance, primary tests run on per-item means across repeats; otherwise single-draw with the measured variance disclosed.

## 10. Feasibility gates

**Gate 1a — version divergence audit (parser-free).** (1) Select the candidate document and two version-pair options: adjacent releases (cleaner alignment, less divergence) and a wider gap (more divergence, more restructuring) — document and pair are open decision E-5. (2) Use the specification's change-history annex to shortlist changed clauses (verify the annex exists on the downloaded documents; fallback: manual table-of-contents skim plus spot diffs). (3) Manually inspect each shortlisted clause in both versions against four pre-registered criteria: (a) divergence is factual, not editorial/renumbering; (b) expressible as a question whose correct answer differs between versions; (c) each version's answer is self-contained in ≤2 clauses; (d) the pair (f_old, f_new) is objectively checkable. (4) Record everything in `version_divergence_audit.csv`. **Pass: ≥16 suitable + ≥4 spares.** Est. 8–12 h. (5) Fallbacks, pre-committed: other version pair → other document → shrink V to what exists, reallocate to S/M at 60/40, downgrade C1 to a descriptive case study — reported as such, not silently.

**Gate 1b — modality density.** Justified only if (i) ≥25 of the 56 answerable items can carry a modal-bearing required fact and (ii) a 30-clause manual sample confirms the modal verbs in candidate gold spans are load-bearing rather than incidental. Measurement: modal-verb frequency map over extracted text, then the manual check. If the gate fails: add a bounded clause subset from a stage-3 specification (corpus-boundary change, logged), or reduce C2 to the error taxonomy without rate-based claims.

**Extraction gate.** Manually verify clause IDs and span boundaries on a random sample of 50 clauses; gate: ≥98% correct clause attribution. Verify no sampled table is split across chunks in a way that separates a value from its headers.

**Retrieval evaluation.** Reported independently of generation, before any faithfulness result is interpreted: recall@k of the gold clause set, recall of the specific gold clause, nDCG — separately for the lexical and dense legs. As each answerable test question is drafted, run retrieval only (local, free) and log RS, yielding the RS distribution before any generation run. Pre-registered rule: if the natural RS=0 count < 8, the insufficient-evidence analysis relies on the evidence-removed intervention and natural-RS results are reported descriptively only.

## 11. Provider capabilities: verified vs assumed

Documentation-current as of 2026-11-03 for one provider; re-verify at implementation (pricing and capabilities drift). Model IDs/pricing, prompt caching, batch discounts and 1M context affect cost and convenience only. **The only load-bearing capability — schema enforcement — is made provider-agnostic** by the §1 fallback (schema-by-prompt + local validator + one re-prompt). Batch and cache savings are not assumed to stack (unverified); cost bounds use the conservative no-cache batch figure. B+ and V depend on answer_B, so batched execution runs in two waves: {A, B, S, O} then {B+, V}. Native provider citations are incompatible with structured outputs on the documented provider and remain an optional Tier-2 comparison only. After the fallback, no experimental conclusion depends on any unverified capability.

## 12. Cost and annotation budget

API cost (assumes ~3.4k in / 500 out tokens per RAG call, frontier-model pricing as cached 2026-11-03; primary model is open decision E-1):

| Item | Estimate |
|---|---|
| One full 90-item pass, 6 runs, interactive with context caching | ≈ $9–12 |
| Same via batch API (conservative, no cache stacking assumed) | ≈ $5–10 |
| k = 3 repeats on a 30-item subsample | ≈ $7 |
| Interventions (24 × 2 conditions × 3 arms) | ≈ $5 |
| Version sub-experiment, six-cell total (288 calls + 32 closed-book) | ≈ $5–6 (of which ≈ $1.50–2 is the six-vs-four-cell increment) |
| Development iteration (~30 partial passes on the 20-item dev set) | $50–100 |
| **Realistic total** | **$80–150** |

Cache the retrieved context as a shared prefix across the B, B+, S and V calls per item; verify hits via the usage cache fields — zero reads across repeated passes means a silent invalidator.

Annotation (the binding constraint — all rates are estimates, assumption A-6, corrected by the pilot and checked against H_max, open decision E-10):

| Task | Volume | Hours |
|---|---|---|
| Gold creation S / M / V / R / X | 24 / 16 / 16 / 12 / 22 | 8.0 / 8.0 / 6.7 / 2.4 / 2.6 |
| Gold creation, dev set | 20 | 6.0 |
| Outcome coding (answer texts × items; O annotated on the 24-item intervention subset only) | ~430 | 3.6 |
| Claim support labelling | ~900 claims | 7.5 |
| Citation labelling + taxonomy | ~560 instances | 3.7 |
| Modality labelling (modal-bearing claims) | ~360 | 1.5 |
| Extra-claim correctness screen (three-route) | — | 1.0 |
| Correctness + GFR for arm A | 90 | 1.5 |
| Intervention conditions (24 × 2 × 3 arms) | 144 answers | 7.2 |
| Version sub-experiment coding (6 cells × 16 × 2 arms) | 192 outputs | 3.2 |
| Closed-book version profile | 32 outputs | 0.5 |
| Codebook development + pilot adjudication | — | 8.0 |
| Adjudication of second-annotator subsample | — | 4.0 |
| **Total (first annotator)** | | **≈ 75 h** |
| Second annotator, stratified 25% subsample | | ≈ 8 h (theirs) |

**Pre-committed cut ladder if hours overrun** (cut in order; costs stated now so they cannot be rationalised later): 1. drop the distractor intervention (−~3.5 h; loses H6's second clause); 2. drop six-cell cells 5–6 (−~1 h; loses VCDR, keeps balanced directional identification); 2a. drop cell 4 (directional claims degrade to exploratory); 3. shrink test set 90 → 70 (−~13 h; MDE worsens to d_z ≈ 0.42; abstention cell loses even large-effect power — state it); 4. demote H2 to descriptive CI (last resort; the headline weakens accordingly). **Never cut:** B+, B-τ, the answer-blind S design, blinded annotation, pre-registration, or the extraction gate.

## 13. Reproducibility record

Where decoding parameters are exposed by the provider, pin and record them; where not, archive outputs and measure variance. Reproducibility rests on archived outputs and measured run-to-run variance (k = 3 on a 30-item stratified subsample, reported as within-condition variance of every primary metric), not on parameter pinning.

For every run, record: timestamp and git commit; corpus manifest with per-document version and SHA-256; exact model identifier, provider and region; effort/reasoning configuration as sent; prompt version hash and the exact rendered prompt; retrieval configuration and the retrieved chunk IDs with scores; the hash of the exact context string supplied; verification thresholds plus the continuous scores s and retrieval scores (so threshold sweeps can be recomputed without re-running inference); the complete raw provider response including token usage; evaluator labels and annotator ID; analysis script version.

Archived raw outputs are the artifact of record; re-running may not reproduce them.
