# Full-Text Review — VersionRAG (arXiv:2510.08109)

**Reviewed:** 2026-11-10 · **Reviewer:** NR-SpecRAG literature review (Phase 2.5) · **Screening origin:** S-12 (owner seed item 12), superseded by FT-05
**Status caution:** arXiv preprint, v1 only (2025-10-09, cs.IR), **not peer-reviewed**; no DOI or journal reference on the abstract page and no venue acceptance found by search as of 2026-11-10. ACL-style template with Limitations and Ethics sections suggests a conference submission; Appendix B.1 states the results come "from the thesis evaluation", indicating derivation from a (student) thesis. Weight accordingly.

---

## 1. Verified bibliography

| Field | Verified value | Source |
|---|---|---|
| Title | VersionRAG: Version-Aware Retrieval-Augmented Generation for Evolving Documents | PDF p.1; abs-page `citation_title` |
| Authors | Daniel Huwiler, Kurt Stockinger, Jonathan Fürst — Zurich University of Applied Sciences (ZHAW), Switzerland | PDF p.1; abs-page `citation_author` (matrix previously used ASCII "Fuerst"; canonical spelling has the umlaut) |
| Posted | 2025-10-09 (v1; only version as of 2026-11-10) | abs-page version history |
| Identifier | arXiv:2510.08109 [cs.IR]; no DOI, no journal reference | abs page |
| Venue status | Preprint; web search (2026-11-10) finds only aggregator mirrors, no acceptance | WebSearch |
| Code / dataset | github.com/danielhuwiler/versionrag — **existence verified** (created 2025-10-03, 27 stars, **no license file**); contents NOT inspected | GitHub API, 2026-11-10 |
| AI-use declaration | "We have used GPT-5, Gemini 2.5 pro and Claude Opus 4.1 to help us refine our writing. All final text in the paper has been verified by the authors." | p.10 |

## 2. Full-text access and verification

- Official arXiv PDF downloaded from the abs page's `citation_pdf_url` (1,863,178 bytes); pypdf extracted **25/25 pages** (`file(1)` misreports 14 — known harmless artifact). **The complete paper was read: main body pp.1–9, Ethics pp.9–10, references pp.10–12, and Appendices A–F pp.13–25** (extended related work; Tables 7–21; qualitative examples; implementation details; dataset examples). This is a complete full-text review.
- Bibliographic identity cross-checked against the abs page's `citation_*` meta tags; abstract text on the abs page matches the PDF verbatim.
- Structure: §1 intro pp.1–2 (Figs. 1–3); §2 related work pp.2–3 (Table 1); §3 problem/graph pp.3 (Fig. 4); §4 system pp.3–4; §5 dataset/setup pp.4–6 (Tables 2–4, Fig. 5); §6 results pp.6–7 (Tables 5–6, Fig. 6); §7 discussion pp.7–8; §8 conclusion p.8; Limitations p.9; Ethics pp.9–10; App. A related work pp.13–14; App. B extended results pp.14–16 (Tables 7–14); App. C ablations pp.17–18 (Tables 15–19); App. D qualitative/failure analysis pp.18–19 (Table 20); App. E implementation pp.20–23 (Figs. 7–9); App. F dataset examples pp.24–25 (Table 21).

## 3. Research problem and work type

Two failure modes motivate the work (§1, pp.1–2): **version conflation** — standard RAG retrieves semantically similar content from multiple versions (Fig. 1: `assert.CallTracker` stability differs across Node.js 14.21.3/15.14.0/16.20.2) — and **no tracking of implicit changes** — systems cannot answer "in which version was X removed?" when the removal is undocumented (Figs. 2–3). Formalisation (§3.1, p.3): documents exist in multiple versions; standard RAG optimises p(c|q) while versioned QA requires **p(c|q, v)**. Three query types are defined: content retrieval, version retrieval, change retrieval.

**Work type:** system + benchmark paper — a retrieval architecture (VersionRAG), a benchmark (VersionQA), and an evaluation. Contributions claimed (p.2): task formalisation, the system, the dataset, and the evaluation. Domain is **software documentation and changelogs** (Node.js, Apache Spark, Bootstrap) — not 3GPP, not normative text.

## 4. System architecture

- **Graph** (§3.2, p.3; Fig. 4): five node levels — category → document → version (edges encode temporal sequence) → content (references to vector chunks) → change nodes (explicit from changelogs; implicit from diff analysis).
- **Indexing** (§4.1, pp.3–4; App. E.3, p.20): LLM attribute extraction (title, summary, version from first pages; first ten pages classify changelog vs documentation); LLM-based document clustering; graph construction; content indexing (512-token chunks, 50-token overlap, version metadata, Milvus Lite); change extraction — explicit via LLM over changelogs, implicit via **DeepDiff line-level comparison between consecutive versions** with LLM interpretation.
- **Retrieval** (§4.2, p.4; App. E.5, p.21): LLM query classification into the three intents plus parameter extraction (category, document, version) — 92% classification accuracy with DeepSeek-R1 70B, 91% GPT-4o Mini, 76% Llama 3 8B (Tables 8/18); mode selection (graph traversal for version/change queries, vector search for content); **version-aware filtering constrains vector search to the target version's chunks** (filtering before/during retrieval, not post-hoc re-ranking).
- **Generation** (§4.3, p.4): LLM receives query + version-specific context "with explicit instructions to ground responses in the provided information". No citation generation; no abstention mechanism.
- **Stack** (App. E.1, p.20): Neo4j (graph), Milvus Lite v2.4.11 (vectors), text-embedding-3-small (all embeddings), Llama 3 8B / GPT-4o Mini / DeepSeek-R1 70B via GroqCloud/OpenAI (same LLM for indexing and generation per configuration), GPT-4.1 as judge; few-shot prompting, no fine-tuning.

## 5. VersionQA dataset and gold standard

- **Composition** (§5.1, pp.4–5; Tables 2/14): 100 manually curated QA pairs over 34 files / ~710 pages: Apache Spark changelogs 2.4.7–3.5.5 (6 files, 150pp), Bootstrap changelogs 5.2.3–5.3.5 (6, 120pp), Node.js Assert docs 11.15.0–23.11.0 (13, 260pp), Node.js Errors docs 15.14.0–23.11.0 (9, 180pp).
- **Categories** (Tables 3/21): Content Retrieval 20, Content Complex 20, Content Version-Specific 20, Version Listing & Inquiry 20, Change Explicit 10, Change Implicit 10 — 60/100 version-sensitive. Examples for every category in App. F (pp.24–25).
- **Gold:** answers only — **no gold evidence spans, no citations**. Curated by the authors; no annotator count, qualifications, guidelines, or inter-annotator agreement reported.
- **Reviewer finding — corpus-relative implicit-change gold.** Implicit changes are detected "between consecutive versions" of the *included* files (App. D.1.2, p.18: "detected between Node.js 21.7.3→22.14.0"), so gold answers inherit the corpus's sampling. App. F.1.6 Q3 asserts CallTracker "was introduced as a new feature in Node.js Assert version 14.21.3"; Node.js's own documentation and release notes record its introduction in **v14.2.0 (backported to v12.19.0)** — externally verified 2026-11-10 against nodejs.org. "Introduced in 14.21.3" is therefore true only relative to the sampled version set (14.21.3 being the first included file containing it). The paper never states this relativity; a system answering with the real-world introduction version would be scored wrong. The same caveat applies in principle to all implicit-change items (e.g., `partialDeepStrictEqual` "added in 22.14.0", F.1.6 Q1).

## 6. Evaluation methodology

- **Metric:** answer accuracy only — "the proportion of correctly answered questions", judged by **GPT-4.1 as an LLM judge** against ground truth, with the automated evaluation "manually reviewed for all 100 questions to ensure reliability" (§5.2, p.6). No judge-accuracy figures, no review protocol, no IAA, no adjudication description.
- **Statistics:** none — "the dataset size precludes extensive statistical testing" (§5.2). No confidence intervals, no significance tests, single run per configuration. Per-category cells are n=10–20 (the headline "60% vs 0–10% implicit change detection" is 6/10 vs 0–1/10).
- **Baselines** (§5.2, p.5; App. E.6–E.7, pp.22–23): Naive RAG (512-token chunks, text-embedding-3-small, top-5 cosine, no metadata) and GraphRAG (official Neo4j implementation, HybridCypherRetriever, default parameters). Same embedder and same LLM per configuration — reasonable parity on components, but **query-time compute is unmatched**: VersionRAG adds LLM calls (classification + parameter extraction) per query that baselines do not spend; the efficiency analysis (Tables 4/9) covers indexing only.
- **Ablations** (§6.5; App. B/C): full 3×3 indexing-LLM × retrieval-LLM matrix (Tables 11/15); explicit/implicit change split per pair (Tables 12/16); GraphRAG with stronger indexing (+4% only, Tables 13/17); classification accuracy per model (Tables 8/18); graph statistics (Tables 10/19).
- **Reproducibility:** code + dataset public (repo existence verified; no license file, so reuse terms are undefined); models are hosted APIs (reproducibility subject to provider drift); prompts described as few-shot (App. E.4) but not printed in the paper.

## 7. Verified results and arithmetic cross-checks

Headline numbers, all located and cross-checked:

| Result | Value | Location | Arithmetic check |
|---|---|---|---|
| Overall accuracy (DeepSeek-R1 70B) | VersionRAG **90.0** vs naive **58.0** vs GraphRAG **64.0** | Table 5, p.6 | Naive and VersionRAG recompute exactly from Table 7 subcategories (58.0 ✓, 90.0 ✓) |
| Overall (GPT-4o Mini / Llama 3 8B) | 79/53/57 and 55/47/43 | Table 5 | All six rows recompute exactly from Table 7 ✓ |
| Version-specific content retrieval | VersionRAG 100.0 vs naive 55.0 (DeepSeek) | Table 7, p.14; §6.1 prose p.6 | ✓ |
| Implicit change detection | 60.0 vs 0.0 (naive) / 10.0 (GraphRAG) | Tables 7/12; §6.1 | ✓ |
| Query classification accuracy | 92/91/76 (DeepSeek/GPT-4o Mini/Llama) | Tables 8/18 | ✓ consistent with §4.2, §6.2, D.4 |
| Indexing efficiency | 186K vs 2,970K **input** tokens; $0.17 vs $6.67; 25min vs 5h12min | Tables 4/9, pp.6/15 | Input-only reduction is **93.7%**; the headline "97% fewer" holds for input+output totals (224K vs 8,892K = 97.5%, Table 9 DeepSeek row) |
| Graph size | 896 vs 11,761 nodes | Tables 6/10/19 | ✓ |
| Cross-model ablation | DeepSeek-index+Llama-retrieve 68%; Llama-index+DeepSeek-retrieve 74%; self-pairs best (90%) | Tables 11/15; §6.5 | ✓ |

**Reviewer-identified internal inconsistency (material):** Table 7's GraphRAG/DeepSeek row prints CR-VS (version-specific content retrieval) = **100.0**. This is irreconcilable with Table 5's GraphRAG figures: Content 70.0 requires (85+80+x)/3=70 → x=45, and Overall 64.0 requires the same x=45 — both constraints back-solve **uniquely to 45.0**, and every other of the nine model×system rows recomputes exactly. A GraphRAG score of 100% on version-specific retrieval would also contradict the paper's own thesis (Figs. 1/3; §6.1 never mentions it). Conclusion: the 100.0 is almost certainly a typo for 45.0. A smaller editorial defect: App. C.3 cites "Section 4.2.3", which does not exist in the paper (likely a thesis remnant).

**Failure-mode analysis** (§6.4 p.7; App. D.3, p.19 — the decisive passage for our purposes): per system, failed queries out of 100 with author-assigned causes —
- VersionRAG fails 10/100: query misclassification 8, incomplete change extraction 2.
- GraphRAG fails 36/100: version confusion 20, missing change detection 16.
- Naive RAG fails 42/100: **"Version mixing (25%): Retrieves conflicting information from multiple versions"**, no change awareness 17.
The sub-causes sum exactly to each system's failure rate (8+2=10, 20+16=36, 25+17=42, matching Table 5's DeepSeek accuracies), establishing that **the percentages are fractions of all 100 queries, not of errors**.

## 8. Version handling — capability analysis and the 25% figure (highest priority)

**The version-mixing error, precisely:**
- **What it is:** a failed query whose failure the authors attribute to the baseline retrieving "conflicting information from multiple versions" (App. D.3, p.19). That one sentence is the entire definition — no formal criterion, no coding rules.
- **How calculated / denominator:** post-hoc categorisation of the naive-RAG configuration's incorrect answers (incorrectness judged by GPT-4.1 + manual review); denominator is all 100 VersionQA queries. **Version mixing = 25/100 queries = 59.5% of naive RAG's 42 errors.** Our Phase-1 screening note ("25% of naive-RAG errors") was imprecise and is corrected in the matrix with this review.
- **Which config/documents:** the totals match Table 5's DeepSeek-R1 70B row exactly, so the analysis is evidently that configuration; no per-topic or per-document error breakdown exists.
- **Level:** answer-level error counts with a retrieval-level causal attribution. It is **not** a per-answer evidence-version attribution: a *correct* answer grounded in wrong-version evidence is invisible to it, and no protocol, second annotator, or per-item labels are reported.
- **Target version source:** the query text itself (e.g., "…in Node.js version 20.19.0"); an LLM extracts the version parameter. The experimental setup never varies target version independently of available evidence.
- **Filtering point:** before/during retrieval (vector search constrained to the target version's chunks; graph traversal for version/change intents). Not post-retrieval.
- **Deliberate evidence mixing / conflict control:** none. The corpus is naturally multi-version, so conflicts occur **incidentally** (Fig. 1 and App. D.1.1 are both-version-conflict instances: the same symbol carries different stability values across versions and naive retrieval surfaces several). They are never isolated as conditions, and VersionRAG is never tested *with* conflicting evidence in context — its filter removes the other versions before generation.
- **Correct-version vs wrong-version evidence independently controlled:** no — no factorial design of any kind.
- **Answer vs target-version gold:** yes at the answer level (gold answers are version-true; GPT-4.1 + manual review). **Evidence is never checked**; citations do not exist.
- **Error taxonomy:** cause-based (version mixing / no change awareness / misclassification / incomplete extraction) — **not** unsupported/contradicted/wrong-version-supported claim coding.

**The five capabilities, kept distinct:**
1. **Multi-version corpus ingestion — demonstrated** (34 files across version ranges; version nodes/edges).
2. **Version-filtered retrieval — demonstrated and indirectly measured** (CR-VS 100% vs baseline 55%, DeepSeek).
3. **Target-version-conditioned generation — partial:** conditioning happens in retrieval; the generator receives pre-filtered context plus grounding instructions; no generation-level version conditioning or evaluation.
4. **Version-specific evidence attribution — not evaluated:** no per-answer check of which version supplied the evidence; only the aggregate post-hoc failure attribution above.
5. **Version-conditioned correctness evaluation — partial, single-axis:** accuracy on version-sensitive *question categories* with version-true gold; no evidence-distribution axis, no per-cell reporting, no conflict conditions.

## 9. Six-cell comparison (C1 gating analysis)

Our approved design: target T ∈ {old, new} × evidence support S ∈ {old-only, new-only, both (incl. real conflict)}, behaviour-coded with VFR/CVS/VCDR/VAA. Mapping VersionQA's actual conditions:

| NR-SpecRAG cell | VersionRAG status | Why |
|---|---|---|
| 1. T=old, S=old-only | **Partially represented (incidental)** | CR-VS includes old-target questions (e.g., Spark 2.4.7 release type; stability at 20.19.0), but whether answer content exists only in the old version is never coded or controlled; many stability items in fact have evidence in *every* version |
| 2. T=new, S=new-only | **Partially represented (incidental)** | e.g., `partialDeepStrictEqual` at 23.11.0 exists only in corpus versions ≥22.14.0 — but again uncoded, unreported per-condition |
| 3. T=old, S=new-only | **Not evaluated** | No question class targets an old version whose answer content exists only in newer versions |
| 4. T=new, S=old-only | **Not evaluated** | Closest relatives are change questions ("when was X removed?"), which ask *about* the transition rather than posing content QA under stale-only evidence; deprecation questions carry their evidence in the target version's own doc (diagonal, not cell 4) |
| 5. T=old, S=both (conflict) | **Partially represented (incidental), never isolated** | Fig. 1 / Fig. 6 / App. D.1.1 are exactly this regime arising naturally, and the 25/100 version-mixing count quantifies baseline failure in it *in aggregate* — but items are not coded by evidence distribution and no conflict-detection behaviour is measured |
| 6. T=new, S=both (conflict) | **Partially represented (incidental), never isolated** | Same evidence and same defects as cell 5 |

**Verdict: 0/6 cells directly evaluated; cells 1, 2, 5, 6 partially and incidentally represented; cells 3, 4 absent.** The deeper difference is *what is held fixed*: VersionRAG evaluates whether its **retrieval architecture** prevents contamination (the remedy), with evidence conditions left to fall where the corpus puts them; our design holds the **supplied evidence set** fixed per cell and codes the **model's behaviour** (adherence, substitution, conflict detection, attribution) under it. Similar vocabulary ("version-specific", "version mixing") does not make these the same experiment.

**Answer to the brief's central question:** VersionRAG measures the narrower phenomenon — single-axis, question-type-conditioned answer accuracy over a naturally multi-version corpus, plus an aggregate, author-coded failure attribution (25/100 version mixing). It does **not** evaluate controlled target-version correctness under manipulated evidence-version distributions. It is nonetheless the strongest *measured* prior for C1: it quantifies version-conflation damage and its architectural remedy, with a released benchmark.

## 10. Limitations — author-stated vs reviewer-observed

**Author-stated** (Limitations, p.9): 100 questions / 34 documents limit statistical testing and fine-grained error analysis; technical-documentation domain only; DeepDiff misses semantic-only changes; Llama 3 8B produces 42% fewer change nodes; version extraction assumes identifiers in first pages, consistent structure, and semantic-versioning conventions; sequential processing cannot capture non-adjacent-version dependencies; no calendar-time reasoning; pipeline errors compound without end-to-end correction; Neo4j+Milvus operational complexity.

**Reviewer-observed (beyond the above):** the Table 7 GraphRAG CR-VS inconsistency (100.0 printed; 45.0 is the only value consistent with Table 5) and the dangling "Section 4.2.3" reference; corpus-relative implicit-change gold, unstated in the paper (CallTracker example externally falsified against nodejs.org — §5 above); the 25% attribution has no coding protocol, no second annotator, no per-item labels; judge validation is asserted ("manually reviewed") but undescribed, with no IAA; the "97% fewer tokens" headline silently uses input+output totals (input-only is 93.7%); query-time LLM compute unmatched against baselines; single run, no variance; gold has answers only (no evidence spans), so no attribution evaluation is even possible on VersionQA as released; the public repo lacks a license, leaving dataset reuse terms undefined.

## 11. Cross-comparison: VersionRAG vs DocNavigator vs TelcoAI vs Chat3GPP

Updating the five-capability table from the DocNavigator review, with VersionRAG now full-text verified ("D" = design/described only, "M" = measured, "—" = absent):

| Capability | VersionRAG (now verified) | DocNavigator | TelcoAI | Chat3GPP |
|---|---|---|---|---|
| 1. Multi-version ingestion | **M** (graph with version edges, 34 docs) | D (namespaces) | M (metadata incl. release) | M (Rel-17+18, no metadata) |
| 2. Version-filtered retrieval | **M** (CR-VS 100% vs 55%) | D | M (metadata filtering) | — |
| 3. Target-version-conditioned generation | Partial (conditioning in retrieval; generator gets filtered context) | D (confirmation-gated) | — | — |
| 4. Per-answer version attribution | **—** (aggregate post-hoc only: 25/100) | D only (metric defined, never valued) | — | — |
| 5. Version-conditioned correctness evaluation | **M, single-axis** (question-type conditioning; no evidence-distribution control) | Defined, not measured | — (future work) | Reporting split only |

On the brief's specific comparison points: **version-mixing measurement** — VersionRAG is the only one of the four with a *number* (25/100), but it is an aggregate answer-level failure count attributed to retrieval contamination, not an answer-level trust metric (correct-but-wrong-version-grounded answers are invisible); DocNavigator *defines* the per-answer version ("fraction of answers grounded in evidence from the wrong release") but reports no value. **Reproducibility of the 25%** — not reproducible from the paper's methodology alone (no coding protocol or per-item labels in the paper); potentially re-derivable from the released artifacts, which were not inspected. **Citations resolved to authoritative clauses** — none of the four (VersionRAG and Chat3GPP generate no citations; TelcoAI never verifies its references; DocNavigator replaces citations with unevaluated navigation actions). **Shared-evidence / cross-version conflict testing** — nowhere controlled; only VersionRAG's incidental Fig.-1-type items. **3GPP Release/spec-revision semantics** — absent in VersionRAG and DocNavigator (software versioning); TelcoAI carries release metadata in retrieval; Chat3GPP splits reporting by release; none distinguishes Release vs specification version vs editorial revision as evaluation dimensions.

## 12. C1–C4 reassessment

- **C1 — version-conditional evidence trust: PARTIAL OVERLAP — the strongest measured prior; gap NARROWED AND SHARPENED, still open.** Evidence: §§8–9 above (single-axis accuracy; 0/6 cells directly evaluated; no per-answer attribution; no conflict isolation; software docs). Charter §10's falsifier — "prior work … already reporting a *version-attribution metric* over authoritative versioned documents" — is **not triggered**: VersionRAG reports accuracy plus an aggregate post-hoc cause count, not an attribution metric; nothing in it corresponds to VFR/CVS/VCDR/VAA or E4. Required wording discipline from here on: C1 claims must credit VersionRAG with (i) establishing versioned-document QA as a task with a public benchmark, (ii) measuring version-sensitive accuracy, and (iii) quantifying aggregate version mixing — and claim only the controlled target×evidence factorial, per-answer/per-citation version attribution, behaviour coding (incl. conflict detection), and the normative 3GPP instantiation. Both gating papers (VersionRAG, DocNavigator) are now full-text resolved; C1's evidence base is complete pending only the CH-11 check (HoH, outdated-information benchmark).
- **C2 — normative-modality preservation: NOT ADDRESSED.** No modality or deontic notion anywhere; stability labels ("0 - Deprecated", "1_2 - Release candidate") are status metadata, not normative strength. Gap unchanged.
- **C3 — claim-level citation reliability: NOT ADDRESSED.** No citations generated or evaluated; the Ethics section *recommends* "providing source attribution for all retrieved information" (p.10) as practice, which underlines that it is absent. Gap unchanged — still no reviewed system measures citation resolution.
- **C4 — evidence sufficiency + calibrated abstention: NOT ADDRESSED.** The system always answers; no sufficiency judgment, no abstention, no coverage/risk. Gap unchanged. (Charter §10's C4 — attribution-controlled evaluation — is also informative here: VersionRAG's gains are not compute-controlled at query time, since classification/extraction LLM calls have no baseline counterpart; this strengthens the case for our B+/B-τ controls.)

## 13. Unresolved questions

1. Whether a peer-reviewed version appears (ACL-style template suggests submission) — re-check before thesis citation.
2. Whether the released repo contains per-item judge labels that would make the 25/100 attribution independently reproducible (repo contents not inspected; also unlicensed).
3. Whether removal-type implicit-change gold (e.g., the `assert.deepEqual()` removal question of Fig. 2) shares the corpus-relativity defect verified for CallTracker.
4. The true GraphRAG CR-VS value (back-solved 45.0 vs printed 100.0) — immaterial to our gaps, material to citing their baseline numbers.
5. CH-11 (HoH, arXiv 2503.04800, outdated-information RAG benchmark) — the last C1-adjacent lead to screen before final C1 wording.

## 14. Reviewer confidence

**High.** Complete official PDF read (25/25 pages including all appendices); identity verified against the abs page's meta tags; every number cited here located in its table/section, with the full arithmetic cross-check of Tables 5↔7 (eight of nine rows exact; the ninth flagged above); the 25% figure resolved to its definition, denominator and level at App. D.3 p.19; one gold-answer defect externally verified against nodejs.org. VersionRAG's evidential weight is **moderate**: unreviewed v1 preprint, n=100 single run, no statistics, author-coded error attribution — but with released artifacts and internally consistent headline results. It is the benchmark C1 prior; our six-cell design remains unduplicated by it.
