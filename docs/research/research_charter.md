# Research Charter — NR-SpecRAG

Revision: 2026-11-09. Incorporates the approved working methodology (decision log D-003–D-030). Owner-approved decisions this revision: D-023 (strict adequacy, E-3), D-024 (non-inferiority margin, E-2), D-026 (six-cell version design, E-13). Open owner decisions are listed in `decision_log.md`.

## 1. Working title

Evaluating Trustworthiness of Retrieval-Augmented Language Models for 3GPP Technical Specifications: Evidence Faithfulness, Citation Reliability, and Calibrated Abstention.

This is a working title, not a claim of novelty.

## 2. Problem statement

LLM-based systems can produce fluent technical answers that are incorrect, incompletely supported, or accompanied by citations that do not substantiate the associated claims. Retrieval does not by itself guarantee that generated claims are faithful to retrieved evidence.

The project will empirically investigate these risks in a bounded corpus of version-pinned 3GPP specifications.

## 3. Research questions

Every research question is bound to a metric, a comparison and a unit of analysis before any experiment is run. A question without all three is not a research question.

- RQ1: Does evidence-aware verification and abstention move the faithfulness–coverage frontier for 3GPP specification question answering, relative to standard RAG and to a compute-comparable self-critique control, and at what cost in coverage, tokens and latency? [AURC; risk at matched coverage — C vs B, C vs B+, C-a vs B-τ]
- RQ2: How frequently are RAG answers that are factually correct relative to the gold answer nevertheless not supported by the clauses the answer itself cites? [joint distribution of correctness × claim attribution status, per arm]
- RQ3: How accurately does the evidence verifier classify claim-level support and citation validity against human labels, and does that accuracy bound the end-to-end gain? [per-class precision/recall/F1 against human gold]
- RQ4: What is the exchange rate between reduction in claim attribution failures and loss of answer coverage and gold-fact recall? [ΔCAF per Δcoverage; the (CAF, GFR) frontier]
- RQ5: Do systems answer according to the specification version explicitly targeted by the question when the supplied context is aligned, misaligned, or contains both versions, and do they detect cross-version divergence? [VFR, CVS, VCDR, VAA on the six-cell controlled-context design]
- RQ6: Do answers preserve the normative strength (shall/should/may/can) of their supporting clauses, and does modality-blind evaluation overstate faithfulness? [MPA; the modality gap]

## 4. Operational definitions

Trustworthiness is not represented by a composite score, at any stage of the project.

### 4.1 Dimensions

- Factual correctness: the answer agrees with the gold answer. Unit: question.
- Evidence faithfulness: factual claims are supported by a named evidence set. Unit: atomic claim.
- Citation reliability: each citation resolves to an existing clause and supports the claim it is attached to. Unit: (claim, citation) instance.
- Calibrated abstention: the system declines or qualifies when evidence is insufficient, without declining answerable, well-retrieved questions. Unit: question.
- Version-conditional grounding: the answer and its citations correspond to the specification version the question targets and the version actually supplied. Unit: question and citation.
- Normative-modality preservation: the answer preserves the normative strength of its supporting clause. Unit: atomic claim carrying a normative modal.

### 4.2 Claim-level support and denominators

Support is judged per claim, never answer-wide. For each factual claim, the claim-level verdict is judged against the union of that claim's own resolved citations — a claim cannot borrow support from a span cited for a different claim. Named evidence sets:

- The claim's own resolved citations (PRIMARY): the citation contract a reader relies on.
- E_ctx (secondary): the full retrieved or supplied context; used for the groundability sub-codes of uncited and unresolvable claims, and for USCR_ctx.
- E_corpus (diagnostic only): the whole pinned corpus; used solely to identify claims that are correct and supported somewhere but not cited.

Every claim receives exactly one status from a forced-choice, mutually exclusive set (protocol §4); partially supported claims must be split into supported and unsupported parts wherever atomically possible. An unqualified claim of "faithfulness" is not permitted anywhere in this project.

### 4.3 Answer outcome coding and adequacy

Outcome, assigned on the core proposition — whether the answer asserts the question's required atomic facts:

- ANSWER: at least one required atomic fact is asserted without an evidential caveat.
- QUALIFIED: at least one required atomic fact is asserted together with an explicit evidential caveat (evidence incomplete, version uncertain, outside indexed scope).
- ABSTAIN: no required atomic fact is asserted. Sub-coded ABSTAIN-REFERRED when the response names the correct external specification, ABSTAIN-BARE otherwise.

Generic epistemic hedging ("typically", "generally") is not an evidential caveat; it codes as ANSWER and is captured by the hedging metrics.

Adequacy (strict; approved as decision E-3): an answered question is adequate iff GFR = 1 on required facts, and no required fact is asserted incorrectly, and the answer contains no incorrect extra claim (bounded three-route incorrectness test, protocol §5).

Two coverages, always reported together:

- Response coverage = (ANSWER + QUALIFIED) / all questions — willingness to commit.
- Adequate-answer coverage = adequate answers / all questions — delivered value.

### 4.4 Corpus answerability and retrieval sufficiency are distinct

- Corpus answerability is a gold, system-independent property of the question and the pinned corpus, labelled CONTAINED / PARTIAL / REFERRED / OUT_OF_SCOPE.
- Retrieval sufficiency (RS) is a per-system, per-run property, computed (not judged) as whether the retrieved context contains the pre-registered minimal gold clause set.

All abstention results are reported as a cross-tabulation of these two axes. The cell (OUT_OF_SCOPE, sufficient) is definitionally empty and is used as a gold-label audit check. The terms "unanswerable" and "insufficient evidence" are never used interchangeably in this project.

## 5. Hypotheses

Pre-registered before the final experiment. H1 and H2 are co-primary; the conjunctive claim ("reduces attribution failures without meaningful loss of completeness") is made by an intersection–union test — each component at its own full level, no Holm correction. Statistical conventions: H1 two-sided α = 0.05; H2 one-sided α = 0.025, decided by the 97.5% one-sided lower confidence bound (protocol §9). The non-inferiority margin Δ = 5 percentage points on GFR is fixed a priori on substantive grounds (≈ one dropped required fact per five to seven answered questions), was approved as decision E-2, and is never changed to achieve statistical power. H3–H8 are secondary, reported with confidence intervals, uncorrected, and labelled non-confirmatory. H0a–H0c are competing explanations and are reported regardless of direction.

- H1 (co-primary, superiority): CAF(C-v) < CAF(B+). Falsified if the cluster-bootstrap two-sided 95% CI on the paired difference includes 0 or the difference is ≥ 0.
- H2 (co-primary, non-inferiority): GFR(C-v) ≥ GFR(B+) − 5 pp. Demonstrated only if the 97.5% one-sided lower confidence bound on the GFR difference exceeds −5 pp; otherwise the pre-registered restricted conclusions apply (protocol §9.2).
- H3: CSP(C-v) > CSP(B+), and citation resolution rate (C-v) > citation resolution rate (B+).
- H4: ungrounded-answering rate (C) < ungrounded-answering rate (B), from the answerability × sufficiency cross-tabulation.
- H5: AURC(C) < AURC(B-τ) on evidence-breach risk — the answer-blind sufficiency score beats a retrieval-score threshold.
- H6 (interventions): with supporting evidence removed, abstention rate (C) − abstention rate (B+) ≥ 25 pp; with a related non-supporting distractor, USCR_ctx(B+) > USCR_ctx(C).
- H7 (version): CVS(B) > 0 in at least one direction; CVS(C) < CVS(B); VAA(C) > VAA(B). Directional asymmetry of CVS is estimated on the balanced six-cell design (approved as E-13) and is identified rather than assumed.
- H8 (modality): the modality gap (modality-aware CAF − modality-blind CAF) > 0 for every arm, and ≥ 50% of modality errors are strengthening (M1).
- H0a (compute): CAF(B+) < CAF(B). If true, part of any naive C-vs-B gain is additional inference, not evidence-awareness; H1's test against B+ controls for this.
- H0b (cheap gating): AURC(B-τ) ≤ AURC(C-a). If true, LLM-based sufficiency assessment adds nothing over a free threshold.
- H0c (contamination): correctness(A) is high on stable answerable items. If so, A-vs-B correctness differences are uninformative and all correctness analysis is stratified by A's per-item outcome.

Behavioural version errors are never claimed to prove parametric-memory use; the parametric interpretation appears only as a disclaimed, exploratory conditioning on the closed-book profile (protocol §7).

## 6. Experimental systems

System C consists of two independent modules:

- Module S (sufficiency): input (question, retrieved_context). MUST NOT receive any candidate answer. Emits a continuous score s ∈ [0,1] and a rationale.
- Module V (verification): input (question, retrieved_context, answer_B). Emits per-claim support verdicts, per-citation validity verdicts, and a revised answer.

Abstention is a post-processing gate over s, not a separate system. Module S's answer-blindness is a hard requirement: it is what makes s applicable as a gate to any arm's answer and what makes the abstention ablation cost no additional inference.

Six generation runs (A closed-book, B standard RAG, B+ compute-comparable generic self-critique, S, V, O oracle gold-spans) yield eight reported conditions: A, B, B+, B-τ (retrieval-score gate), C-a (sufficiency gate only), C-v (verification only), C (gate + verification), O. B-τ, C-a, C-v and C are compositions of logged artifacts and require no additional calls; threshold sweeps are therefore free, and every gated condition is reported as a full risk–coverage curve rather than a single operating point. B+ is compute-comparable to V by construction (same call count, same input payload, matched output shape), verified by measured token accounting — not claimed as exactly compute-matched. Optional Tier-2 arms (long-context L, local model Loc) are strictly after the core experiment. Data flow, module I/O and the reproducibility audit of the derived conditions are specified in `experimental_protocol.md` §1.

## 7. Corpus boundary

Pinned documents: TS 38.300 and TS 38.401 (normative prose) and TR 21.905 (vocabulary). Exact versions are not selected yet (open decision E-5).

For the version-conditional sub-experiment (RQ5), one of TS 38.300 or TS 38.401 is additionally stored at a second version from a different release. The main benchmark retrieval index contains the pinned primary version only; the second version is quarantined in a separate store used exclusively for constructed contexts, so that standard retrieval can never surface it. Which document and which version pair is open decision E-5, settled by the Gate 1a divergence audit. All cross-version conflict material is real, provenanced specification text; no fabricated specification text is created, stored or published at any point in this project.

The glossary (TR 21.905) is stored and retrieved separately from normative prose, so that short vocabulary entries cannot crowd out substantive clauses in a single index.

Every document records release, version, publication date, source URL, download date and SHA-256 checksum in a versioned manifest. Documents are never committed; the manifest and a fetch script are.

The indexed corpus, not all of 3GPP, defines the scope of every answerability claim.

## 8. Evaluation principles

- Create gold answers and evidence spans before evaluating system outputs.
- Use independent human annotation where feasible.
- Define claim segmentation and support labels before annotation.
- Report each metric separately, with uncertainty intervals.
- Include answer coverage whenever reporting abstention or selective risk.
- Preserve all prompts, configurations, model identifiers, code revisions, and raw outputs.
- Do not tune thresholds on the final test set.
- Report metrics as a frontier, never as a composite. Where a single pre-registered outcome is required, use a superiority test paired with a non-inferiority test on the counter-metric, not a weighted sum.
- Report evidence-breach risk (R_EB) and strict attribution risk (R_strict) separately, always together, with their gaming monitors (protocol §6).
- Bootstrap claim-level and citation-level metrics by resampling QUESTIONS, not claims or citations; observations within one answer are correlated.
- Never compare abstaining and non-abstaining systems at their own operating points. Report risk at matched coverage and the area under the risk–coverage curve.
- Report every claim-level faithfulness metric alongside gold-fact recall and claim yield. A reduction in attribution failures accompanied by a fall in either is degenerate hedging, not improved trustworthiness.
- Human labels are the primary instrument for all primary outcomes. An LLM judge may be used only as an auxiliary scalability check, must be from a different model family than the generator and the verifier, and must be reported with per-class agreement against human labels.
- Annotation of system outputs is blinded to system identity. Abstention phrasing can leak system identity; abstention cases are normalised before labelling.

## 9. Research-gap status

**Provisional / unvalidated.** Four candidate contributions are stated in §10, each with the literature evidence required to claim it and the finding that would falsify it. None may be asserted in any writing, presentation or submission until the full-text review in `literature_review_protocol.md` is complete and the novelty test has been applied to that specific claim.

The literature review is sequenced AFTER the contribution claims are sharpened, because the novelty test requires a precise claim to compare against prior work. Searching for "3GPP AND RAG AND faithfulness" tests a gap this project does not claim.

## 10. Candidate contributions (provisional, pending literature validation)

> **C1 — Version-conditional evidence trust.** A controlled-context benchmark and metric set (VFR, CVS, VCDR, VAA) measuring whether a RAG system answers according to the specification version explicitly targeted and supplied, cites the correct version, and detects real cross-version divergence — instantiated on two versions of the same 3GPP specification, so every conflict is authentic and provenanced rather than synthesised. The metrics are behavioural; no claim about internal parametric memory is made.
> *Claimable only if* no prior work measures version-conditional attribution on versioned normative sources. *Falsified by* prior work in temporal/knowledge-conflict QA already reporting a version-attribution metric over authoritative versioned documents. *Precondition:* ≥16 genuinely divergent clauses exist between two versions (assumption A-1, Gate 1a).

> **C2 — Normative-modality faithfulness.** A metric (MPA) and error taxonomy (M1–M4) for whether generated answers preserve the normative strength of their supporting clauses, plus the **modality gap** — quantified evidence that modality-blind faithfulness evaluation overstates faithfulness on normative documents by scoring modality-shifted claims as supported. Secondary contribution.
> *Claimable only if* no prior work evaluates modal/deontic strength preservation as an attribution dimension. *Falsified by* existing work on deontic fidelity in legal or regulatory NLG that already reports such a metric. *Precondition:* sufficient modal density in the chosen corpus (assumption A-2, Gate 1b).

> **C3 — Clause-level citation error taxonomy.** E1–E5 as a reusable error model for attribution in normative documents, exploiting the fact that citation targets are objectively verifiable here. *Claimable only as* a domain instantiation, not as a new method. Likely the weakest on novelty grounds.

> **C4 — An attribution-controlled evaluation of evidence-aware RAG.** A compute-comparable and cheap-baseline-controlled comparison, where any reported gain from "evidence awareness" is demonstrably not attributable to additional inference (B+) or to retrieval-score thresholding (B-τ). *Claimable as* methodological rigour, not novelty; it makes the negative result publishable too.

What is explicitly **not** claimed: RAG for 3GPP; a new retrieval method; a new verification architecture; any composite trustworthiness score; any inference about internal parametric knowledge from behavioural version errors.
