# Full-Text Review — TelcoAI (ghosh-etal-2025-telcoai)

Review date: 2026-11-10 · Matrix ID: `telcoai-2025` · Screening record: A-11 (Phase 1), FT-01 (this review)
Status: **complete full-text review** (all 13 pages including appendices). First full-text review of the project.

---

## 1. Verified bibliographic details

| Field | Value (verified against ACL Anthology record and PDF) |
|---|---|
| Title | TelcoAI: Advancing 3GPP Technical Specification Search through Agentic Multi-Modal Retrieval-Augmented Generation |
| Authors | Rahul Ghosh¹, Chun-Hao Liu¹, Gaurav Rele¹, Vidya Sagar Ravipati¹, Hazar Aouad² — ¹Generative AI Innovation Center, Amazon Web Services; ²Bouygues Telecom (p. 1305) |
| Venue | Findings volume of IJCNLP-AACL 2025 (Anthology volume `2025.findings-ijcnlp`; PDF footer: "Proceedings of the 14th International Joint Conference on Natural Language Processing and the 4th Conference of the Asia-Pacific Chapter of the Association for Computational Linguistics", December 20–24 2025) |
| Pages | 1305–1317 · DOI 10.18653/v1/2025.findings-ijcnlp.80 · ISBN 979-8-89176-303-6 |
| URL | https://aclanthology.org/2025.findings-ijcnlp.80/ |
| Type | Peer-reviewed conference paper (Findings); **industrial system paper** (AWS + telecom operator) |

## 2. Full-text access and verification status

- Official PDF downloaded 2026-11-10 from the ACL Anthology (418,186 bytes, PDF 1.5, 13 pages).
- Phase 1 had marked the PDF "text layer unreadable"; that was a **fetch-tool limitation, not a damaged PDF**. Local extraction with `pypdf` 7.0.1 recovered all 13 pages (~54 KB of text) including both appendix pages.
- BibTeX fetched from the Anthology and matched against the Phase 1 matrix row: authors, venue, pages, DOI, month all confirmed. No arXiv version was needed.
- Everything in this review is sourced from the extracted full text. Page numbers refer to the proceedings pagination (1305–1317).
- One extraction caveat: Table 1 uses a two-column layout with brace marks rendered as ↑/↓ arrows. All values quoted below are either unambiguous in the extraction or cross-confirmed by the paper's prose.

## 3. Research problem and system architecture

**Problem (§1, pp. 1305–1306).** 3GPP specifications are hierarchical, dense and multi-modal; prior 3GPP QA systems use flat/generic chunking, ignore diagrams and tables, use single-step retrieval, and are evaluated on isolated atomic queries. TelcoAI is an engineering response: an "agentic, multi-modal" RAG system for 3GPP document search/QA, aimed at "reducing the cognitive load on engineers" (p. 1305).

**Scope.** Specifically 3GPP: ingestion takes 3GPP specifications in Microsoft Word (.docx) format (§3.1, p. 1307). Metadata extracted per document: Release (e.g. "R16"), Series (e.g. "23"), Specification number (e.g. "23548-i30"), and three-part Version (e.g. "V18.3.0") (§3.1.1, p. 1307). Evaluation spans Releases 16–19 (Human-QA) and the TSpec-LLM corpus, Releases 8–19 (§4.1, p. 1310).

**Architecture (§3, Figure 1 p. 1308).** Two pipelines:

1. *Ingestion:* metadata extraction; **section-aware recursive chunking** following heading structure with max size and overlap (Eqs. 1–2, p. 1307); **image/table tagging** — an LLM (Claude 3.5 Sonnet v2) writes natural-language descriptions of images and tables, concatenated into chunks with `[IMG_XXX]` markers (Eqs. 3–4, p. 1308).
2. *QA* (§3.2, pp. 1309–1310), four stages:
   - **Pre-retrieval:** LLM query reformulation/planning into ≤3 standalone sub-queries plus ≤5 follow-up queries (Eq. 5); LLM metadata extraction from the query (release/series/specification) for filtered retrieval. Prompts in Appendix A.1 (p. 1316).
   - **Context retrieval:** hierarchical hybrid search, sim = α·cosine + (1−α)·BM25 with α = 0.5 (Eq. 6, p. 1309); sub-section chunks first, then parent-section expansion.
   - **Post-retrieval:** top-40 → metadata filtering → similarity re-ranking → top-5 (§3.2.3, p. 1309).
   - **Answer generation:** Claude 3.5 Sonnet v2 over fused chunks + referenced images (Eqs. 7–10, p. 1310). The generation prompt (Appendix A.2.1, p. 1317) instructs "explicit references to specifications and sections" and wraps used document names in `<docs></docs>` tags.

**"Agentic" in this paper means LLM query decomposition/planning**, not an autonomous tool-use loop: the Limitations section states the system "is limited to single-turn queries" and names "an agentic planner for action planning and execution monitoring" as *future* work (p. 1314).

**No verification module.** Nothing in the architecture checks generated claims against evidence; claim-level checking appears only in the *evaluation* (RAGChecker). Citations/references are produced by prompt instruction and are never resolved or validated.

**Implementation (§4.4, p. 1311).** Claude 3.5 Sonnet v2 (generation, planning, image/table description), Amazon Titan Text Embeddings v2 (1024-d), Elasticsearch BM25; temperature 0.7; top-5 chunks of 512 tokens, 20% overlap; identical chunk/retrieval settings across all compared systems.

## 4. Datasets and annotation

§4.1 (p. 1310) names "four distinct datasets": Human-QA, Synthetic-QA, Telco-DPR, TSpec-LLM. **No Telco-DPR results appear anywhere in the paper** (Tables 1–6 cover only the other three) — an internal inconsistency.

| Dataset | Size | Origin | Gold |
|---|---|---|---|
| Human-QA | **41 questions**, 12 specifications, Releases 16–19; 60% single-document retrieval, 25% cross-document synthesis, **15% version comparisons** | expert-curated (no further procedure given) | ground-truth answers exist (required by Eq. 11); construction not described |
| Synthetic-QA | **size not reported** | multiple-choice questions generated with Claude 3.5 Sonnet v2 using the exam-generation method of Guinet et al. (2024), "validated using Mistral Large 2 … and human experts" | MCQ-style; validation procedure not described |
| TSpec-LLM | evaluation set of 100 MCQs; corpus 30,137 docs, Releases 8–19, 535M words | external benchmark (Nikbakht et al., 2024) | MCQ keys |

**Not reported:** gold evidence passages or clause-level spans; annotator identity, count or qualifications; annotation instructions; inter-annotator agreement; any train/dev/test separation or leakage control. Footnote 2 (p. 1306): "We are working on releasing the datasets as part of the published paper" — **the evaluation data was not released at publication**.

"Human" in Human-QA refers to question curation. **No human judges any system output**; all output-level judgments are automated (§5 below).

## 5. Evaluation metrics — exact definitions

Framework: **RAGChecker** (Ru et al., 2024) with **Llama 3.3 70B Instruct as both claim extractor and claim checker** (§4.4, p. 1311). Responses and ground-truth answers are decomposed into claims (Eq. 11, p. 1310). All claim-level judgments are therefore LLM-judged; nothing is deterministic matching or human evaluation.

| Metric | Definition (as given or as placed by the paper) | Unit | Reference | Where |
|---|---|---|---|---|
| **Recall (R)** | proportion of *ground-truth* claims present in the model response: R = \|{c_gt ∈ m}\| / \|{c_gt}\| (Eq. 12) | gt-claim | human gold answer | §4.2 p. 1310; Tables 1–3 |
| **Claim Recall (CR)** | "evaluates retriever effectiveness" (Table 1 caption, p. 1311); listed under "Retriever Metrics" in Table 6 → RAGChecker's retriever claim recall: proportion of gt claims covered by the **retrieved context** | gt-claim | gold answer vs retrieved chunks | Tables 1–2, 6 |
| Precision / F1 | response claims matching gold / harmonic mean | response claim | gold answer | Table 6 only: **P 0.35, F1 0.46** |
| Context Precision | retriever metric | chunk/claim | — | Table 6: 0.34 |
| **Faithfulness** | RAGChecker generator metric: response claims entailed by retrieved context; the paper itself gives no formula, only the value and gloss "our generated responses accurately represent the technical specifications" (A.3, p. 1317) | response claim | **retrieved context** (not citations, not gold) | Table 6 only: **0.92** |
| Hallucination / Self Knowledge | RAGChecker generator metrics (no formulas in paper) | response claim | retrieved context | Table 6: 0.05 / 0.03 |
| Context Utilization | generator metric | — | — | Table 6: 0.71 |
| Accuracy | MCQ accuracy by difficulty | question | MCQ key | Table 5 |

**The three abstract figures, resolved:**
- "87% recall" = R, TelcoAI + Claude 3.5 Sonnet v2, **Human-QA (41 questions)**, Table 1 p. 1311. It is an *answer-completeness-vs-gold* measure, not retrieval recall and not correctness.
- "83% claim recall" = CR, same row — a **retrieval-side** coverage metric, *not* generation-side attribution (Phase 1 had provisionally read it as generation-side; corrected here).
- "92% faithfulness" = RAGChecker generator faithfulness, **appendix Table 6 only, Human-QA only**, LLM-judged against retrieved context. No baseline faithfulness is reported anywhere, so the "16% improvement" cannot and does not refer to it.
- "16% improvement" = relative recall gain over the best baseline on Human-QA: (0.87−0.75)/0.75.

**Rationale the authors give for recall as primary** (§4.2, p. 1310): human gold answers are concise while model responses add context, so recall "measures whether the model captures all information from the ground truth, without penalizing … supplementary details." The flip side surfaces only in the appendix: **Precision 0.35** — about two-thirds of response claims are not in the gold answer — which A.3 (p. 1317) frames as "a choice to prioritize comprehensive information retrieval … rather than indicating inaccuracy."

**Stated metric limitations by the authors:** none beyond the precision gloss. No variance, confidence intervals, significance tests, or repeat runs are reported, at temperature 0.7.

## 6. Verified results

| Result | Value | Dataset | Baseline | Config | Source |
|---|---|---|---|---|---|
| Recall | **0.87** | Human-QA | Chat3GPP+Claude 0.75 (+16% rel.) | TelcoAI + Claude 3.5 Sonnet v2 | Table 1, p. 1311 |
| Claim Recall | **0.83** | Human-QA | Chat3GPP 0.73 (+13.7% rel.) | retriever-level | Table 1, p. 1311 |
| Recall | 0.85 | Synthetic-QA | Chat3GPP+Claude 0.73 (+16.4% rel.) | same | Table 1, p. 1311 |
| Claim Recall | 0.83 | Synthetic-QA | Chat3GPP 0.72 (+15.3% rel.) | same | Table 1; **prose on p. 1312 misprints this as "(0.87 vs 0.72)" — 0.83 matches both Table 1 and the stated 15.3%** |
| Ablation | 0.63/0.60 → 0.87/0.83 (R/CR); largest steps: section chunking (→0.72/0.73) and multi-modal fusion (0.82/0.81→0.87/0.83) | Human-QA | Naive RAG | cumulative components | Table 2, p. 1312 |
| Generator robustness | R 0.87 / 0.85 / 0.84 / 0.83 (Claude 3.5 Sonnet v2 / Claude 3 Opus / Llama 3.3 70B / Mistral Large 2) | Human-QA | — | TelcoAI, R only | Table 3, p. 1312 |
| Latency | 11.81 s total vs Chat3GPP 8.07 s, Telco-RAG 9.91 s, Naive 6.76 s | — | — | Claude 3.5 Sonnet v2 | Table 4, p. 1312 |
| MCQ accuracy | **93** overall (Easy 93 / Inter. 93 / Hard 92) vs Chat3GPP+Claude 92, Telco-RAG+Claude 91 | TSpec-LLM (100 MCQs) | — | accuracy % | Table 5, p. 1313 |
| RAGChecker panel | P 0.35, R 0.87, F1 0.46; CR 0.83, CtxP 0.34; CtxUtil 0.71, Halluc. 0.05, SelfKnow. 0.03, **Faithfulness 0.92** | Human-QA only | none (TelcoAI only) | Llama 3.3 70B judge | Table 6, p. 1317 |

Baselines (§4.3, p. 1311): 5 base LLMs, Naive RAG, Telco-RAG, Chat3GPP — all re-run by the authors with five generation models and identical chunk/retrieval parameters (a genuine fair-comparison effort at the retrieval/prompt level). Note: TelcoAI's own Table 1 row is Claude-only; the margin over Chat3GPP+Claude (0.87 vs 0.75) mixes architecture and is not compute-matched (TelcoAI adds planning + image-description LLM calls; latency table shows +3.7 s).

## 7. Limitations and failure analysis

**Author-stated limitations (p. 1314):** (1) .docx-only ingestion (no scanned PDFs, Markdown, slides); (2) single-turn only — no multi-turn state, no agentic planner (future work); (3) not optimized for efficiency/latency/deployment.

**Checklist against NR-SpecRAG's trust dimensions** (classification: explicitly evaluated / mentioned but not evaluated / not reported):

| # | Dimension | Status in TelcoAI |
|---|---|---|
| 1 | Behaviour under insufficient evidence | **Not reported** |
| 2 | Correct abstention when evidence insufficient | **Not reported** |
| 3 | False abstention despite sufficient evidence | **Not reported** |
| 4 | Unsupported extra claims | **Explicitly evaluated, automated aggregate only** — Hallucination 0.05 / Self-Knowledge 0.03 (Table 6, LLM-judged vs retrieved context); Precision 0.35 counts beyond-gold claims but is framed as deliberate comprehensiveness; no human check, no error taxonomy |
| 5 | Claim-level evidence attribution | **Explicitly evaluated, automated aggregate only** — Faithfulness 0.92 vs *retrieved context*; never against the answer's own cited references; no per-claim human labels |
| 6 | Citation resolution to actual clauses | **Mentioned but not evaluated** — prompt instructs "explicit references to specifications and sections" + `<docs>` tags (p. 1317); no resolution or correctness metric |
| 7 | Incorrect/incomplete citations | **Not reported** |
| 8 | Specification-version mismatches | **Mentioned but not evaluated** — release-metadata filtering (§3.1.1, §3.2.1); no mismatch metric |
| 9 | Cross-version conflicts | **Mentioned but not evaluated** — 15% of Human-QA are version comparisons (p. 1310), example queries compare R16–R19 (p. 1309, 1316); no per-category results, no conflict analysis; "sophisticated cross-version comparison capabilities" named as **future work** (p. 1313) |
| 10 | Normative modality (SHALL/SHOULD/MAY) preservation | **Not reported** (no occurrence in the paper) |
| 11 | Conflicting evidence across versions | **Not reported** (no conflict condition constructed or observed) |
| 12 | Confidence calibration / selective prediction | **Not reported** |

Careful language: items marked "not reported" are absent from the paper; this does not establish the deployed system lacks related behaviour — only that it was not evaluated or claimed.

**Additional observations:** no error analysis section exists; no per-category (single-doc / cross-doc / version) result breakdown despite the categorisation being introduced; n = 41 for every headline number; Synthetic-QA size unreported; evaluation data unreleased at publication; single runs at temperature 0.7 with no variance or significance reporting.

**Side-finding for the matrix:** TelcoAI's reference list (p. 1315) cites RAGChecker as *Advances in Neural Information Processing Systems, 37:21999–22027* — secondary confirmation of the NeurIPS 2024 publication our matrix had marked unverified (primary record still unchecked).

## 8. Comparison with NR-SpecRAG (protocol = source of truth)

| Research dimension | TelcoAI (verified) | NR-SpecRAG proposed design (experimental_protocol.md) | Difference / overlap |
|---|---|---|---|
| 3GPP source material | .docx specs; Human-QA: 12 specs R16–19; TSpec-LLM corpus R8–19 | Pinned document/version pair from TS 38.300 / TS 38.401 / TR 21.905 family (owner decision E-5 open); quarantined second version | Overlap in domain; genuine difference: we pin and hash exact versions as experimental objects, TelcoAI treats the corpus as a store |
| Specification version control | Release/series/spec/version metadata for *retrieval filtering*; version-comparison queries in data | §7 six-cell factorial: target version T × supplied context S ∈ {old, new, both}; VFR/CVS/VCDR/VAA metrics | **Genuine methodological difference**: TelcoAI's version handling is an implementation choice in retrieval; no version-conditioned evaluation exists there |
| Gold answers | Exist for Human-QA (41), construction undocumented | §3 gold record per item: answer, required facts, answerability × RS label, evidence spans; five-artifact commitment protocol | Genuine difference in documentation and auditability; TelcoAI's gold is not released |
| Gold evidence spans | **Not reported** | Clause-level gold spans per required fact | Genuine difference |
| Human verification of outputs | **None** (all output judgments by Llama 3.3 70B via RAGChecker) | Human claim-level annotation, blinded to system; LLM judge auxiliary only | **Genuine difference — the central one** |
| Claim-level support evaluation | Automated entailment vs retrieved context (Faithfulness 0.92, aggregate) | §4 per-claim status tree (CITED-SUPPORTED/-PARTIAL(MATERIAL/MINOR)/-CONTRADICTED/…), CAF primary metric, materiality rule | Partial overlap in unit (claims); difference in judge (human vs LLM), reference (resolved citations vs whole context), and granularity (taxonomy vs single rate) |
| Citation reliability | Citations generated by prompt; **never evaluated** | §4 per-(claim, citation) verdicts; E1–E5 resolution taxonomy; citation resolution rate | Genuine difference |
| Evidence sufficiency | Not evaluated | Module S (answer-blind sufficiency), USCR_ctx, §8 interventions | Genuine difference vs TelcoAI (but crowded generically — Phase 1: RINSE, Sufficient Context, SURE-RAG) |
| Abstention | Not evaluated | Calibrated abstention: CY, coverage/risk at matched coverage, B-τ sweep | Genuine difference vs TelcoAI (same generic-crowding caveat) |
| Normative modality preservation | Not reported | §6 MPA, M1–M4 codes, modality gap = modality-aware − modality-blind CAF | Genuine difference |
| Version mismatch experiments | None (future work per authors) | §7 six-cell with real divergent clause pairs; no synthetic conflict (§8) | Genuine difference |
| Error analysis | None | E1–E5 + claim statuses + outcome/adequacy codes designed for failure analysis | Genuine difference |
| Statistical treatment | None (point estimates, single runs) | §9: H1/H2 co-primary, fixed Δ = 5 pp NI, 97.5% one-sided bound, power table, restricted conclusions | Genuine difference |
| Compute framing | Latency table; no compute-matched generation baseline | B+ compute-comparable control; zero-extra-call derived conditions | Genuine difference (TelcoAI latency analysis is an operational, not inferential, control) |

Implementation-choice differences (not methodological): choice of generator/embedder, hybrid-search weighting, chunking parameters, .docx parsing. Differences not yet supported by evidence on our side: **all of ours** — NR-SpecRAG has run no experiments; every row above compares TelcoAI's *demonstrated* work against our *proposed* design. Areas where TelcoAI already performs essentially the same evaluation: aggregate automated claim-vs-context entailment (RAGChecker faithfulness/hallucination) — our LLM-judge auxiliary track must treat this as the established default, and our human gold must demonstrate value beyond it.

## 9. C1–C4 research-gap assessment (vs TelcoAI specifically)

- **C1 — version-aware evidence attribution: Partially overlapping.** TelcoAI *engineers* version awareness (metadata-filtered retrieval; 15% version-comparison questions; cross-release example queries, pp. 1307–1310, 1316) but *evaluates* none of it: no version-conditioned metric, no per-category breakdown, no claim-to-release attribution, no conflict cells; the authors explicitly defer "sophisticated cross-version comparison capabilities" to future work (p. 1313) — which affirms the problem is recognised and open. Evaluation-side C1 survives TelcoAI. Remaining external threat: the unread Research Square preprint (rs-8854256, P-01).
- **C2 — normative modality preservation: Potentially distinct.** Zero occurrence of modality/SHALL-strength anywhere in the paper. Consistent with Phase 1 (LexDeMod = detection; Deontological Keyword Bias = judgment).
- **C3 — claim-level citation reliability: Potentially distinct** (vs TelcoAI), with a caveat. TelcoAI never checks its generated references; its claim-level checking is whole-context entailment. The caveat: automated claim-vs-context verification is standard practice (RAGChecker — the very framework TelcoAI uses), so C3's delta must rest on resolved clause-ID citations, per-(claim, citation) verdicts, the E1–E5 taxonomy, and human gold — not on "claim-level evaluation" per se.
- **C4 — evidence sufficiency + calibrated abstention: Potentially distinct vs TelcoAI** (nothing there), **but unchanged from Phase 1 globally**: generic sufficiency/abstention is crowded (RINSE, SURE-RAG, ESB, Sufficient Context, GRAB-RAG); our claim stays confined to the attribution-based risk definitions, compute-matched controls and human gold on a normative corpus.

**Other directions that remain valuable after this review:** human-annotated 3GPP evaluation gold with documented procedure and IAA (none exists in TelcoAI; Roychowdhury et al. validated *automatic metrics* against experts but built no claim-level attribution gold); statistical inference for 3GPP RAG comparisons (absent in TelcoAI); precision/verbosity-controlled reporting (TelcoAI's P = 0.35 shows the recall-only risk); a released, auditable benchmark (TelcoAI's datasets are unreleased).

## 10. Unresolved questions about TelcoAI

1. Synthetic-QA size, validation protocol and agreement — not reported.
2. Why Telco-DPR is named as an evaluation dataset but absent from results.
3. Per-category (single-doc / cross-doc / version-comparison) performance — not reported; the 15% version slice is where our C1 would bite.
4. Whether `<docs>` references in outputs are ever correct — unevaluated.
5. Reproducibility: prompts are in the appendix, but data is unreleased and runs are single-sample at temperature 0.7.

## 11. Reviewer confidence and remaining verification work

**Confidence: high** on everything quoted above — complete text extracted from the official PDF; all headline numbers located in their tables with page numbers; abstract claims reconciled against tables (one prose misprint documented in §6). **Residual risks:** Table 1's two-column layout (values cross-checked against prose where possible); RAGChecker metric semantics beyond the paper (CR as retriever-side claim coverage; faithfulness/hallucination formulas) rest on the RAGChecker framework's definitions — the paper names the metrics and placement ("Retriever Metrics"/"Generator Metrics", Table 6) but restates no formulas, so **reading RAGChecker (ragchecker-2024, matrix) in full is now elevated in priority** to lock those definitions. Figures (Figure 1) were not visually inspected (text-only extraction); nothing in the analysis depends on them.
