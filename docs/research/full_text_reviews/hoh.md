# Full-Text Review — HoH (ACL 2025)

**Reviewed:** 2026-11-10 · **Reviewer:** NR-SpecRAG literature review (Phase 2.6) · **Screening origin:** CH-11 (citation chase from VersionRAG, FT-05), resolved by FT-06
**Status:** **Peer-reviewed** — ACL 2025 main conference, Long Papers. The strongest-provenance paper in the review series so far.

---

## 1. Verified bibliography

| Field | Verified value | Source |
|---|---|---|
| Title | HoH: A Dynamic Benchmark for Evaluating the Impact of Outdated Information on Retrieval-Augmented Generation ("How Outdated information Harms RAG", footnote 1, p.6037) | ACL PDF p.6036; Anthology BibTeX |
| Authors | Jie Ouyang, Tingyue Pan, Mingyue Cheng (corresponding), Ruiran Yan, Yucong Luo, Jiaying Lin, Qi Liu — State Key Lab of Cognitive Intelligence, University of Science and Technology of China | PDF p.6036 |
| Venue | Proceedings of the 63rd Annual Meeting of the ACL (Volume 1: Long Papers), Vienna, July 27–Aug 1 2025, pp. 6036–6063, ISBN 979-8-89176-251-0 | Anthology BibTeX (`ouyang-etal-2025-hoh`) |
| DOI | 10.18653/v1/2025.acl-long.301 | Anthology BibTeX |
| arXiv | 2503.04800 — v1 posted 2025-03-03; last arXiv update 2025-07-18 (camera-ready window) | arXiv abs meta tags |
| Version reconciliation | The ACL version was read in full and is authoritative here. The current arXiv abstract is textually identical to the ACL abstract; no substantive differences were identified at the metadata/abstract level. A full text-level diff of the two PDFs was **not** performed. | reviewer check 2026-11-10 |
| Code/data | github.com/0russwest0/HoH (public per abstract); **not inspected** per brief scope | PDF p.6036 |

## 2. Full-text access and verification

Official ACL Anthology PDF (7,659,440 bytes) downloaded from the canonical URL; pypdf extracted **28/28 pages** (printed pages 6036–6063). **The complete paper was read**: main body §§1–6 (pp.6036–6043), Limitations §7 + Acknowledgments (p.6044), references (pp.6044–6047), Appendix A effectiveness/efficiency (pp.6048–6049, Tables 7–8), Appendix B construction details (pp.6049–6050, Table 9), Appendix C dataset statistics (pp.6050–6051, Tables 10–12, 21–22), Appendix D experiment setup (pp.6051–6052, Table 13), Appendix E additional experiments (pp.6052–6055, Tables 14–20), and all prompt figures (Figs. 5–13, pp.6058–6063). One extraction caveat: **Figure 3's per-category bar values (p.6041) are CID-garbled in the text layer**, so the end-to-end per-category breakdown is cited from prose only; every controlled-experiment number comes from machine-readable tables.

## 3. Research problem and work type

HoH targets a failure mode the authors call the prevalence of **outdated information coexisting in retrieval sources**: RAG is used to fix LLM knowledge staleness, but the knowledge base itself accumulates stale content (Fig. 1's Biden/Trump example, p.6036). The paper positions itself within Time-Sensitive QA and explicitly distinguishes **Time-Situated Questions** (explicit temporal context, e.g., "Who was president in 2023?") from **Present-Anchored Questions** (implicitly require the most current answer) — and places itself in the present-anchored family (§2.2, p.6038). Contributions (p.6037): (1) the benchmark (HOH-QA + HOH-SearchEngine: 96,124 QA pairs, 219,463 documents, with — "for the first time" — outdated answer/evidence annotations); (2) the construction method (token-level diff + LLM pipelines); (3) the first systematic analysis of outdated information's harm to RAG. **Work type:** peer-reviewed benchmark + empirical-analysis paper; no new RAG system is proposed.

## 4. Benchmark construction (HOH-QA)

- **Source:** Wikipedia snapshots, initial pair 2024-06-01/2024-07-01, monthly updates through 2024-11-01; articles <200 characters excluded (§3, p.6038).
- **Factual change extraction** (§3.1, pp.6038–6039): Myers diff over sentence sequences → character-level diff → heuristic filters (pronoun/spelling/high-frequency stylistic edits, additions/deletions dropped — App. B.1, p.6049) → **fine-tuned Qwen2.5-0.5B screening classifier** operating on token-level-diff-marked input. Pipeline volume (Table 8, p.6049): 3,708,543 raw sentence pairs → −74.57% heuristic → −85.80% screening → 133,973 LLM-processed → 101,253 final (96,124 new + 5,129 updated answers; Table 10, p.6050).
- **Human annotation:** three annotators labelled ~2,000 heuristically-filtered sentence pairs (~400 factual / ~1,600 not), "annotated and mutually verified" — **no inter-annotator agreement statistic reported** (App. B.2, pp.6049–6050). On the 8:2 split, the screener reaches **96.80% accuracy / 95.08 F1**, vs EvolvingQA's diff-only 20.85/34.50 and GrowOVER's SimCSE 41.26/41.08 (Table 7, p.6049).
- **QA generation** (App. B.3, p.6050): Llama-3.1-70B-Instruct generates a question answerable from each sentence of the pair with **contradictory answers** (old vs new), plus evidence sentences and timestamps (Fig. 5 prompt, p.6058). LLM quality review on five criteria (same-answer check, answer accuracy, clarity, completeness, **temporal independence**); ≤3 regenerations then discard. **Manual validation: 500 QA pairs**, accuracy 98.4–100% per criterion (Table 9, p.6050); annotator count unstated.
- **Temporal independence (decisive for us):** generation and review prompts *forbid* time-situated phrasing — "Avoid using specific date references, such as 'as of [date]', 'in [year]'" and avoid asking when information was updated (Figs. 5–6, pp.6058–6059). **Every question is therefore present-anchored; there are no old-target questions anywhere in the benchmark.**
- **Answer chains:** each question stores the current answer + evidence + last-modified date and a chronological `outdated_infos` list (answer/evidence/date per superseded version — Table 21/22 field spec and examples, pp.6056–6057). Distribution: 91,856 questions with one outdated answer, up to 66 with five (Table 11, p.6051). Domains: Sports 31.2%, Open 33.0%, Finance 23.7%, Movie 6.3%, Music 5.8% (Table 12).
- **Maintenance:** monthly diff-based updates; superseded answers preserved and marked outdated (§3.1 Automatic Update; App. B.4).

## 5. HOH-SearchEngine and retrieval conditions

Elasticsearch BM25 extended with a **Gaussian temporal decay** to "discount outdated information, mimicking the temporal preference of real-world search engines"; corpus = current **and historical versions** of all articles (§3.2, p.6040). Document categories (§4.1, p.6040): **R** (relevant: contains the correct answer), **O** (outdated: "previously relevant but have become obsolete"), **D** (distracting: semantically similar, no correct answer). Retrieval metric: binary HR@k computed separately for R and O (App. D.2, p.6051). Default end-to-end pipeline: bge-m3, no rerank, k=20 documents, 256-token passages with 32 overlap, top n=5 passages (App. D.4, p.6052).

## 6. Evaluation methodology

- **Eval set:** 10,000 QA (10 clusters × 1,000 random each) (§4.3, p.6040). Generators: Llama-3.1-70B/8B-Instruct, Qwen-2.5-7B-Instruct, local bfloat16 vLLM, temperature 0.3, ≤100 output tokens (App. D.5); GPT-4o on a 1,000-sample subset (App. E.3).
- **Time-aware prompting is the default:** system prompt carries "Today Date: 01 November 2024", every document carries "Last Modified Time", and the model is instructed to prefer the most current information and answer "Unsure" when not sure (Fig. 10, p.6062; §4.2). Failures occur *despite* explicit temporal metadata and instructions.
- **Scoring** (§4.4, p.6040): 3-way answer-level judgment following CRAG (Yang et al., 2024b): **Perfect = +1, Missing = 0, Harmful = −1**; Score = mean. "This method penalizes false content, prioritizing user safety by favoring omissions over errors."
- **Judge** (App. D.3, p.6051): **Llama-3.1-70B-Instruct**, prompt in Fig. 9 (p.6061): match any ground-truth answer → Correct; explicitly unsure → Unsure; **otherwise → Harmful**. Manual validation reports near-perfect precision/recall (Perfect 98.5/100.0, Harmful 100.0/98.5, Missing 100/100 — Table 13) but **the validation sample size and annotator count are unstated**. Two reviewer notes: (i) the judge never sees the *outdated* answer, so "Harmful" is any non-gold non-abstention — harmful outputs are **not** coded as "adopted the outdated answer" specifically; (ii) the judge model is also one of the evaluated generators (self-judging for the Llama-70B rows).
- **Conditions actually tested:** (a) **end-to-end** with natural retrieval, post-hoc categorised by retrieved content {Both, R-only, O-only, None} (Fig. 3 right, p.6041); (b) **controlled fixed-context generation** — evidence supplied, not retrieved: {R×1, D×n} (Table 4), {R×1, O×1, D×n} under three orderings Score↓/Date↓/Date↑ (Table 5), {O×1, D×n} (Table 18, p.6054), long-answer variants (Tables 19–20), GPT-4o variants (Tables 16–17); (c) **vanilla** no-retrieval (Table 14); (d) **timeliness-awareness probes**: Current Awareness AC and Outdated Awareness AO, judged via the Fig. 11 yes/no prompt with both R and O supplied (Fig. 4; Table 6). "Since for most questions, R and O typically have only one instance, we primarily conduct experiments by adjusting the number of D" (App. D.4).
- **Statistics:** no confidence intervals or significance tests; large n (10,000) but single run.

## 7. Verified results and cross-checks

| Finding | Value | Location | Check |
|---|---|---|---|
| End-to-end overall score | Llama-70B 51.7, Llama-8B 40.0, Qwen-7B 29.9 | §5.1 prose, p.6041 (Fig. 3) | Figure values unreadable in extraction; prose values used |
| Search engine, no decay | O@5 0.8837 **>** R@5 0.8707 | Table 2, p.6042 | ✓ outdated slightly *out-ranks* relevant |
| Gaussian decay trade-off | O@5 → 0.4950; R@5 → 0.7023 | Table 2 | ✓ "nearly 50%" O in top-5; R −16.8pp ≈ the stated ~17% |
| Embedding models | bge-m3 R@5 0.7312 / O@5 0.5507; O hit >50% across models | Table 3, p.6042 | ✓ |
| Temporal retrievers | MRAG: O@5 0.0534 but R@5 0.2921 (bge-base); TempRALM R@5 0.4473 / O@5 0.2264 | Table 15, p.6054 | ✓ O filtered only at severe R cost |
| Distractors alone are mild | Llama-70B score 89.12 → 87.98 as D: 1→6 (R present) | Table 4, p.6043 | ✓ <2 points |
| One outdated passage is severe | Llama-70B 89.12 → 63.88–67.00 (Score↓); harmful 4.12 → 14–16% | Tables 4–5, p.6043 | ✓ intro's "at least 20% drop" is conservative (24–42 across models); "harmful up to 11%" ≈ +11.8pp |
| Ordering sensitivity | Qwen-7B spans 61.54 (Date↑) to **−2.77** (Date↓ n=5); Llama-70B best under Date↑... **no — Date↓** (83.88) | Table 5 | ✓ −2.77 verified; models disagree on best ordering (Llama-70B: newest-first; Qwen-7B: oldest-first) |
| O-only context | Harmful 83.07/79.73/71.21 (n=1), scores −71.15/−68.77/−63.00 | Table 18, p.6054 | ✓ confident misinformation |
| Vanilla (no retrieval) | Missing 62.03/80.84/97.21; scores −19.21/−10.98/−1.47 | Table 14, p.6052 | ✓ "appropriate uncertainty" without evidence |
| Awareness probes | AC/AO: Llama-70B 75.2/87.2, Llama-8B 72.0/58.5, Qwen-7B 41.3/77.2 | Fig. 4, p.6043 | ✓ |
| Awareness ↔ outcome | Both-aware: Perfect 93.24/83.86/76.75; neither: Harmful 72.67/50.44/49.41 | Table 6, p.6043 | ✓; prose "11% more harmful with AO than AC" is actually 35.94−23.25 = **12.69pp** — minor prose/table drift |
| GPT-4o (1,000 subset) | R+D stable ~88–89.5; with O: 58.0–71.4 depending on order | Tables 16–17, p.6054 | ✓ same qualitative pattern |

**Key analytic distinction the paper itself draws (§5.2):** retrieval contamination (R/O hit rates — O cannot be filtered without sacrificing R) versus generation vulnerability (fixed-context degradation with O present even when R is present and temporal metadata is explicit) — the two are measured separately, which is precisely what makes the controlled tables valuable.

## 8. Temporal/version handling — capability analysis

Against the review series' five capabilities:
1. **Multi-version ingestion — demonstrated:** the search corpus holds current + historical article versions (219,463 docs); the QA store holds full answer chains with per-version evidence and timestamps.
2. **Version-filtered retrieval — partial, as a *negative result*:** Gaussian decay and temporal retrievers (MRAG/TempRALM) are evaluated and shown to trade R recall for O suppression; no version/namespace isolation exists (there is no version identity, only recency).
3. **Target-version-conditioned generation — absent by design:** the target is always "now" (present-anchored questions; current date in prompt). No question requests a past state.
4. **Per-answer version attribution — absent:** outcome scoring never attributes a generated answer to the outdated vs current evidence (the judge does not receive the outdated answer). The AC/AO probes test model *capability* to recognise currency, not attribution of produced answers.
5. **Version-conditioned correctness evaluation — partial, recency-anchored:** correctness is conditioned on evidence composition (R/O/D, controlled) but always judged against the *current* gold. "Version" here is document recency of encyclopedic facts — there is no document-revision vs specification-release distinction, no version identifiers in questions, and no normative text.

## 9. Six-cell comparison (C1 gating analysis)

HoH's R/O conditions must not be auto-equated with our cells: its questions are **current-anchored only** and its gold is always the current answer. Mapping:

| NR-SpecRAG cell | HoH status | Evidence |
|---|---|---|
| A. Old target / old-only | **Not evaluated** — no old-target questions exist (temporal-independence criterion, Figs. 5–6; §2.2 present-anchored) | pp.6038, 6050, 6058–6059 |
| B. New target / new-only | **Directly evaluated** (outcome level): controlled {R×1, D×n} fixed-context experiments + end-to-end "Relevant" subset | Table 4 p.6043; Tables 16/19; Fig. 3 |
| C. Old target / new-only | **Not evaluated** — no old targets | — |
| D. New target / old-only | **Directly evaluated** (outcome level): controlled {O×1, D×n} — harmful 67–83%, scores −58 to −71 — plus end-to-end "Outdated" subset | Table 18 p.6054; Fig. 3; §5.1 |
| E. Old target / both | **Not evaluated** — no old targets | — |
| F. New target / both (real conflict) | **Directly evaluated** (outcome level): controlled {R×1, O×1, D×n} with three ordering strategies; R and O contradict by construction (QA generation requires contradictory answers) and conflicts are real Wikipedia revisions, not synthetic | Table 5 p.6043; Tables 17/20; Fig. 5 prompt |

**Reading:** HoH directly instantiates the **entire current-target half of our factorial (cells B, D, F) at outcome level, with controlled, fixed evidence and real conflicts** — the strongest evidence-condition prior in the series. What it does not do: (i) the old-target half (A, C, E) — structurally excluded by its present-anchored design, so target-version symmetry is untested; (ii) behaviour coding — outcomes are Perfect/Missing/Harmful vs current gold, with no adherence/substitution/detection coding and no check of *which* evidence an answer used; (iii) citation-level anything; (iv) version semantics beyond recency (no explicit version identifiers, releases, or normative clauses). Its "correct behaviour under O-only" is also defined differently: HoH rewards abstention or parametric recovery (recency norm), whereas our cell 4 asks whether the model follows, substitutes, or flags mismatched evidence under an explicit version target.

## 10. Limitations — author-stated vs reviewer-observed

**Author-stated** (§7, p.6044): fixed snapshot frequency cannot capture fast-moving domains; QA derived from single articles (no multi-source synthesis); the mock search engine's outdated corpus is homogenized (historical versions of the same article are similar to each other, unlike diverse real-world staleness).

**Reviewer-observed:** present-anchored-only design (a scoping choice, but it means temporal *symmetry* is untested); harmful responses never coded against the annotated outdated answer — the "misleading" mechanism is inferred from condition deltas, not per-answer attribution, even though the dataset's `outdated_infos` would permit it; judge is one of the evaluated generators (Llama-70B self-judging) and judge-validation/manual-QA sample sizes and annotator counts are partly unstated, with no IAA statistics anywhere; no uncertainty quantification; Fig. 3's numeric values are only in a vector figure (unverifiable from the text layer); minor prose/table drifts (the "11%" vs 12.69pp AO/AC harmful gap; "variations up to 25%" for Llama-70B is 22.8–23.9 across orderings); Wikipedia's "last modified" timestamp is article-level recency, which conflates edit time with fact validity.

## 11. Cross-paper synthesis (vs VersionRAG, DocNavigator, TelcoAI, Chat3GPP, RAGChecker)

| Capability | HoH (ACL 2025) | VersionRAG | DocNavigator | TelcoAI | Chat3GPP | RAGChecker |
|---|---|---|---|---|---|---|
| Controlled evidence-mixture conditions | **Yes — R/O/D counts + ordering, fixed contexts** | No (natural corpus) | No (none reported) | No | No | No (natural retrieval) |
| Explicit version/target conditioning | No (present-anchored) | **Yes (version in query)** | Design only | No | No | No |
| Per-answer/claim version attribution | No | No (aggregate post-hoc 25/100) | Defined, unmeasured | No | No | No (no temporal notion) |
| Outcome taxonomy | Perfect/Missing/Harmful (+1/0/−1) | Accuracy | 3 metrics defined, 1 valued | RAGChecker metrics | Accuracy/LLM-Eval | Claim-entailment P/R suite |
| Conflict testing | **Real revision conflicts, controlled (R vs O in one context)** | Incidental only | None | None | None | None |
| Abstention | **Scored outcome (Missing/Unsure)** | None | Described, unmeasured | None | Instructed, unmeasured | None |
| Domain | Encyclopedic Wikipedia | Software docs | Software docs | 3GPP | 3GPP | Multi-domain open QA |
| Peer-reviewed | **Yes (ACL 2025)** | No (v1 preprint) | No (v1 preprint) | Yes (Findings IJCNLP-AACL) | No (v1 preprint) | Yes (NeurIPS 2024 D&B) |

HoH and VersionRAG are now revealed as **complementary halves of C1's prior art**: VersionRAG has explicit version targets but no evidence control; HoH has rigorous evidence control but no version targets. Neither attributes answers or citations to versions; neither touches normative text. HoH additionally supplies the first *scored* abstention outcome in the series (C4-adjacent) and borrows its scoring scheme from CRAG (Yang et al., 2024b), tying it to the benchmark lineage RAGChecker also inhabits. Its Table 1 landscape identifies CLARK-News (Li et al., 2024) as the only other benchmark with both evidence-text and outdated-info annotations — added as screening lead CH-12.

## 12. C1–C4 reassessment

- **C1 — version-conditional grounding: PARTIAL OVERLAP — the strongest controlled-evidence prior; gap NARROWED AGAIN, still open.** What HoH already establishes (and our wording must no longer claim as novel): *controlled mixtures of current and outdated evidence supplied to generators*; *measured harm from outdated evidence even when current evidence is present, under explicit temporal metadata and instructions*; *real (non-synthetic) conflicts*; *retrieval-vs-generation decomposition of temporal failure*; *ordering effects*. What HoH does not do (the surviving delta): the **old-target half of the factorial** (its questions are present-anchored by explicit construction); **correctness conditioned on a requested version** rather than on recency; **per-answer/per-claim/per-citation version attribution** (its judge never consults the outdated answer); **behaviour coding** (adherence/substitution/conflict-detection: our VFR/CVS/VCDR/VAA have no analogue); **normative versioned specifications** (3GPP releases vs encyclopedic fact churn; no revision-vs-release semantics). Charter §10's falsifier — "prior work in temporal/knowledge-conflict QA already reporting a **version-attribution metric** over authoritative versioned documents" — is **not triggered**: HoH reports outcome rates under evidence conditions, not attribution. **Wording now to avoid:** "first controlled current-vs-outdated evidence conditions", "first measurement of wrong-era evidence harming RAG", "first real-conflict evaluation". **Defensible remaining claim:** first controlled, *symmetric* target-version × evidence-version factorial with per-answer and per-citation version attribution and behaviour coding, instantiated on normative 3GPP specifications.
- **C2 — normative-modality preservation: NOT ADDRESSED.** Encyclopedic facts; no deontic/normative language notion. Gap unchanged.
- **C3 — citation-to-clause reliability: NOT ADDRESSED.** No citations are generated, resolved, or checked; the dataset's evidence fields serve construction and retrieval scoring, not output attribution. Gap unchanged.
- **C4 — sufficiency and calibrated abstention: PARTIAL OVERLAP (new).** HoH *scores* abstention: "Unsure" is instructed, detected by the judge, and carries 0 in a scheme that explicitly favors omission over error (§4.4); missing-rates are reported per condition (e.g., 62–97% vanilla, Table 14). This is the first reviewed paper that integrates abstention into its primary metric. It does **not** evaluate calibration, selective risk, coverage-risk trade-offs, or abstention correctness (abstaining when evidence is sufficient vs insufficient is not separated) — so C4's core (answer-blind sufficiency S, calibrated/selective abstention) stands, but C4's framing must now cite HoH as outcome-level prior alongside the known generic abstention literature.

## 13. Unresolved questions

1. Whether HoH's released per-item outputs would allow re-coding harmful answers against the annotated outdated answers (repo not inspected) — would sharpen the "misled by O" mechanism claim.
2. CH-12 (CLARK-News, arXiv 2406.11830): the only other outdated-info-annotated benchmark per HoH's Table 1 — verify it does not do controlled conditions or attribution before final C1 wording.
3. The exact protocol of the AC/AO probes (what candidate responses are judged, over which subset) is under-specified in the paper; immaterial to our gaps but material to citing those numbers.
4. Full arXiv-vs-ACL text diff not performed; abstracts identical, ACL read in full — revisit only if a claim is later sourced to the arXiv version.

## 14. Reviewer confidence

**High.** Complete official ACL PDF read (28/28 pages, all appendices and prompt figures); bibliography primary-verified via the Anthology BibTeX; every number in §7 located in its table with arithmetic/consistency checks (one figure unreadable in extraction, flagged; two minor prose/table drifts, flagged). HoH's evidential weight is **high** (peer-reviewed, large-scale, released artifacts, validated pipeline components) — which is exactly why its scope boundary matters: it is a *present-anchored, outcome-level, attribution-free* evaluation. The six-cell design remains justified: its old-target half, attribution metrics, behaviour coding, and normative-spec instantiation are all absent from HoH, VersionRAG, and the rest of the reviewed corpus.
