# NR-SpecRAG

**Research project:** evaluating the trustworthiness of retrieval-augmented LLM answers to questions about public 3GPP 5G specifications — evidence faithfulness, citation reliability, and calibrated abstention.

> Research-gap status: **provisional**. No novelty claim is made or permitted until the per-claim literature review in `docs/research/literature_review_protocol.md` is complete.

## Research objective

Determine whether evidence-aware verification and calibrated abstention improve the trustworthiness of LLM answers grounded in a bounded, version-pinned 3GPP corpus — measured against controls that rule out the two cheap explanations (extra test-time compute, and simple retrieval-score thresholding), and reported as separate dimensions on a risk–coverage frontier, never as a composite trust score.

The master's-scale design, metrics, statistics and budget are specified in `docs/research/research_charter.md` and `docs/research/experimental_protocol.md` (revisions of 2026-11-09).

## Trustworthiness dimensions

Measured separately at all times; no composite score at any stage.

**Primary evaluation dimensions**

1. **Factual correctness** — the answer agrees with the gold answer (unit: question).
2. **Evidence faithfulness** — factual claims are supported by a named evidence set, judged per claim against that claim's own resolved citations (unit: atomic claim).
3. **Citation reliability** — each citation resolves to an existing clause and supports the claim it is attached to (unit: claim–citation instance; error taxonomy E1–E5).
4. **Calibrated abstention** — the system declines or qualifies when evidence is insufficient, without declining answerable, well-retrieved questions (unit: question; always reported with coverage).

**Secondary / extension dimensions** (candidate contributions, pending literature validation)

5. **Version-conditional grounding** — the answer follows the specification version the question explicitly targets and the version actually supplied (controlled six-cell sub-experiment).
6. **Normative-modality preservation** — the answer preserves the normative strength (shall/should/may/can) of its supporting clause (MPA metric, M1–M4 error codes, modality gap). Explicitly a secondary contribution.

## Research questions (charter §3)

- **RQ1:** Does evidence-aware verification and abstention move the faithfulness–coverage frontier, relative to standard RAG and a compute-comparable self-critique control, and at what cost in coverage, tokens and latency?
- **RQ2:** How frequently are answers that are factually correct nevertheless unsupported by the clauses they themselves cite?
- **RQ3:** How accurately does the verifier classify claim-level support and citation validity against human labels?
- **RQ4:** What is the exchange rate between reduced claim attribution failures and lost coverage / gold-fact recall?
- **RQ5:** Do systems answer according to the explicitly targeted specification version under aligned, misaligned and both-version contexts, and do they detect cross-version divergence?
- **RQ6:** Do answers preserve the normative strength of their supporting clauses, and does modality-blind evaluation overstate faithfulness?

## Hypotheses (charter §5)

Co-primary, conjoined by an intersection–union test and pre-registered before any test run:

- **H1 (superiority):** the verification condition reduces the claim attribution failure rate versus the compute-comparable control — CAF(C-v) < CAF(B+); two-sided α = 0.05.
- **H2 (non-inferiority):** gold-fact recall is preserved within a fixed margin — GFR(C-v) ≥ GFR(B+) − 5 pp; one-sided α = 0.025, decided by the 97.5% one-sided lower confidence bound. The margin Δ = 5 pp is fixed a priori on substantive grounds (≈ one dropped required fact per 5–7 answered questions) and is never changed to achieve statistical power; pre-registered restricted-conclusion templates govern the inconclusive case.

Secondary (H3–H8, reported with CIs, non-confirmatory): citation support precision and resolution; ungrounded-answering rate; sufficiency gating vs a retrieval-score threshold (risk–coverage AURC); behaviour under evidence interventions; version adherence and substitution (H7); the modality gap (H8). Competing explanations reported regardless of direction: **H0a** (gains are just extra compute), **H0b** (cheap gating suffices), **H0c** (closed-book contamination).

## Experimental design: the S/V architecture

System C is two independent modules on top of standard RAG:

- **Module S (sufficiency):** sees only (question, retrieved context) — never a candidate answer — and emits a continuous sufficiency score with a rationale. Answer-blindness makes the score a legitimate abstention gate over any arm's answer.
- **Module V (verification):** sees (question, retrieved context, baseline answer) and emits per-claim support verdicts, per-citation verdicts, and a revised answer.

Abstention is a post-hoc gate over the logged score, not a separate system. Six generation runs (A closed-book, B standard RAG, B+ compute-comparable generic self-critique, S, V, O oracle gold-spans) yield **eight reported conditions** — A, B, B+, B-τ (retrieval-score gate), C-a (gate only), C-v (verification only), C (gate + verification), O — because the gated conditions are compositions of logged artifacts requiring zero additional calls, which also makes threshold sweeps (full risk–coverage curves) free. B+ makes any claimed gain not attributable to extra inference; B-τ makes it not attributable to trivial gating. Evidence-breach risk and strict attribution risk are reported separately, with gaming monitors.

## Version experiment (controlled extension)

A controlled-context sub-experiment on 16 version-divergent items, separate from main-benchmark retrieval: a balanced six-cell factorial of **target version T ∈ {old, new} × supplied context S ∈ {old-only, new-only, both}**, arms B and C, with an explicit target version in every question. Three orthogonal codes (version adherence, divergence detection, answer/qualify/abstain response) feed behavioural metrics (VFR, CVS, VCDR, VAA). The main retrieval index contains the pinned primary version only; the second version lives in a quarantined store used solely for constructed contexts, and all conflict material is real, provenanced specification text. Behavioural version errors are never claimed to prove parametric-memory use.

## Corpus status

Proposed documents: 3GPP **TS 38.300**, **TS 38.401** (normative prose) and **TR 21.905** (vocabulary, stored and retrieved separately). **Exact versions are not yet selected** (open decision E-5), and no documents have been ingested. Before ingestion, every document's release, version, publication date, source URL, download date and SHA-256 checksum is recorded in a versioned manifest; documents themselves are never committed. The indexed corpus — not all of 3GPP — defines the scope of every answerability claim. No fabricated specification text is created at any point.

## Project status

**Research methodology established and owner-approved (2026-11-09)** — see `docs/research/decision_log.md` (D-001–D-030), including the approved non-inferiority margin (E-2), strict adequacy definition (E-3) and six-cell version design (E-13).

No experiments have run; no benchmark exists yet; there are no results. Remaining before any test run, in order: the per-claim **literature review** (gap validation for candidate contributions C1–C4), the **feasibility gates** (version-divergence audit, modality-density check, ≥98% clause-extraction accuracy, retrieval-sufficiency distribution), the annotated **pilot** with codebook freeze, and **pre-registration**. Open owner decisions: E-1 (model/provider), E-5 (document + version pair), E-6 (second annotator), E-7 (claim extraction mode), E-8 (pilot pivot contingency), E-10 (annotation-hours ceiling).

## Repository layout (actual)

```text
README.md
requirements.txt
configs/                 # placeholder (README only)
data/
  raw/                   # original documents (never committed; .gitkeep)
  processed/             # extracted, normalized text (.gitkeep)
  indexes/               # retrieval indexes (.gitkeep)
  evaluation/            # questions, gold answers, evidence spans (README only)
docs/
  research/
    research_charter.md            # RQs, dimensions, hypotheses, systems, contributions
    experimental_protocol.md       # design, coding, metrics, statistics, gates, budget
    literature_review_protocol.md  # contribution-targeted search + per-claim novelty test
    literature_matrix.csv          # extraction schema (27 columns; no entries yet)
    decision_log.md                # D-001–D-030 + open owner decisions
experiments/             # placeholder (README only)
src/                     # empty Python package; implementation not started
tests/                   # placeholder (README only)
```

## Reproducibility and research-integrity rules

1. No novelty claim before a documented, per-claim literature review.
2. No invented or estimated experimental results.
3. Preserve source document versions and provenance (versioned manifest with checksums).
4. Distinguish factual correctness, evidence faithfulness, citation reliability and calibrated abstention; never aggregate them into a composite score.
5. Report answer coverage alongside every abstention or selective-risk result; compare gated systems at matched coverage, never at their own operating points.
6. Use human review for gold labels and primary outcomes, with a documented adjudication process; blind output annotation to system identity.
7. Keep evaluation data separate from development examples; select thresholds on the dev set only — no tuning on the final test set.
8. Record model identifiers, prompts, retrieval configuration, code revision and run date for each run; where decoding parameters are exposed, pin and record them; where not, archive raw outputs and measure run-to-run variance.
9. Pre-register hypotheses, primary outcomes, the non-inferiority margin and the analysis before any test-set run; archived raw outputs are the artifact of record.
10. Report both selective risks (evidence-breach and strict attribution) separately, with their gaming monitors.

## Research gap

**Provisional and unvalidated.** Four candidate contributions (version-conditional evidence trust; normative-modality faithfulness; a clause-level citation error taxonomy; an attribution-controlled evaluation) are stated in `research_charter.md` §10, each with the evidence required to claim it and the finding that would falsify it. None may be asserted in any writing, presentation or submission until the novelty test has been applied to that specific claim.
