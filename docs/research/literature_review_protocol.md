# Literature Review Protocol

Revision: 2026-11-09 (decision log D-017, D-030).

## Objective

Establish what has already been demonstrated about trustworthiness, evidence faithfulness, citation support, and abstention in RAG systems, with particular attention to telecom and 3GPP specifications — and apply the novelty test to each candidate contribution (C1–C4, `research_charter.md` §10) individually. The review is sequenced AFTER the contribution claims are sharpened, because the novelty test requires a precise claim to compare against prior work. The research gap remains provisional until this review is complete.

## Search venues

- IEEE Xplore
- ACM Digital Library
- ACL Anthology
- Scopus or Web of Science (if accessible)
- Google Scholar
- arXiv (preprints clearly labelled as such)

## Search strings

These strings test the candidate contributions, not the broad "3GPP + RAG" framing, which this project does not claim.

Candidate contribution C1 — version-conditional evidence trust:
1. `("version" OR "temporal" OR "knowledge conflict" OR "context conflict") AND ("retrieval augmented generation" OR "attribution") AND ("citation" OR "grounding")`
2. `("parametric knowledge" OR "memorisation") AND ("context" OR "retrieved evidence") AND (conflict OR override OR "staleness")`
3. `("versioned documents" OR "document revision" OR "standards revision") AND ("question answering" OR "attribution")`

Candidate contribution C2 — normative-modality faithfulness:
4. `("modal verb" OR deontic OR "normative strength" OR "shall OR should OR may") AND (faithfulness OR entailment OR attribution OR "natural language generation")`
5. `("legal" OR "regulatory" OR "compliance") AND ("obligation" OR "permission") AND (generation OR summarisation OR "question answering") AND (fidelity OR faithfulness)`

Candidate contribution C3 — clause-level citation error taxonomy:
6. `("citation" OR "attribution") AND ("error analysis" OR taxonomy OR "failure modes") AND ("large language model" OR "retrieval augmented")`

Methodological controls (to establish that the design is adequately controlled, not novel):
7. `("self-correction" OR "self-critique" OR "self-verification") AND ("large language model") AND (limitations OR "without external feedback")`
8. `("selective prediction" OR abstention OR "risk-coverage" OR "selective risk") AND ("large language model" OR "question answering")`
9. `("attributable to identified sources" OR "citation support" OR "evidence attribution") AND (evaluation OR benchmark)`

Domain coverage (to establish the state of the art, and to avoid re-deriving it):
10. `("3GPP" OR "telecom" OR "O-RAN" OR "telecommunication standards") AND ("question answering" OR "retrieval augmented generation" OR benchmark OR dataset)`

Candidate anchors to locate and read first, recorded as leads to verify rather than as established citations: telecom QA benchmarks and 3GPP specification datasets for LLMs; telecom-domain LLM surveys and domain-adapted telecom models; O-RAN specification benchmarks; recent 3GPP RAG assistant papers. Confirm each exists before citing it.

## Inclusion criteria

- Peer-reviewed papers and relevant preprints with methods described sufficiently to assess.
- Work on LLM factuality, RAG grounding, evidence attribution, citation verification, uncertainty, or abstention.
- Telecom/standards-specific studies and general methods that can transfer to technical-document QA.
- Foundational work may be included regardless of publication year.

## Exclusion criteria

- Marketing material without an evaluable method.
- Papers with no meaningful connection to the research questions.
- Duplicate versions (retain the most authoritative version and record preprint history).
- Papers whose claims cannot be checked from the available text.

## Screening process

1. Search and export records.
2. Deduplicate.
3. Screen titles and abstracts.
4. Read potentially relevant full texts.
5. Follow backward and forward citations for the closest studies.
6. Record inclusion/exclusion decisions and reasons.
7. Update the gap statement only after full-text review, per candidate contribution.

## Screening record

Maintain `screening_log.csv` with per-stage counts (records retrieved, deduplicated, title/abstract screened, full text assessed, included, excluded with reason) sufficient to draw a PRISMA-style flow diagram. A gap claim without these counts is not auditable.

## Extraction fields

Record, per paper, every column of `literature_matrix.csv`: title, authors, year, venue/status, domain, research question, method, model, dataset/corpus, metrics, main findings, limitations, trust dimensions, evidence intervention, abstention evaluation, relevance to each NR-SpecRAG research question, full-text-reviewed flag, notes — plus the contribution-targeted fields `doi_or_url`, `closest_prior_work`, `delta_vs_nr_specrag`, `claims_vs_demonstrated` (what the paper claims vs what its experiments demonstrate), `version_conditional_evaluation`, `modality_evaluation`, `compute_matched_control`, `abstention_coverage_reported`.

## Novelty test, applied per claim

For each candidate contribution in `research_charter.md` §10, record:

- the single closest prior work;
- a one-paragraph delta against it across task, domain, trust dimension, evidence manipulation, annotation protocol, metrics, baselines and evaluation scope;
- the specific finding that would falsify the claim.

If the delta paragraph cannot be written, the claim is not a contribution and is withdrawn or narrowed. Absence of an exact keyword match is not evidence of novelty, and no novelty claim may appear in any writing, presentation or submission before this test is complete for that specific claim.
