# Full-Text Review — DocNavigator (Research Square rs-8854256)

**Reviewed:** 2026-11-10 · **Reviewer:** NR-SpecRAG literature review (Phase 2.4) · **Screening origin:** P-01 (Q13), resolved by FT-04
**Status caution:** Research Square preprint, v1, **not peer-reviewed**; no linked journal or conference version exists per Crossref as of 2026-11-10. All findings below must be weighted accordingly before any thesis citation.

---

## 1. Verified bibliography

| Field | Verified value | Source |
|---|---|---|
| Title | Agentic Rag for Version-controlled Documentation: A Pdca-driven Navigation and Semantic Comparison Framework | PDF cover p.1; Crossref; landing-page `citation_title` (the "Rag"/"Pdca" casing is the platform's) |
| System name | DocNavigator | Abstract p.2; used throughout |
| Authors | Vignesh Chinthakuntla (FISAT); Sankar Ganesh Paramasivam (Illinois Institute of Technology); Neelarapu Tejaswini (Illinois Institute of Technology); Jagadesh Radhakrishnan (SRM Institute of Technology) | PDF cover p.1; Crossref |
| Posted | 2026-02-12 (v1; only version) | Crossref `posted`; cover p.1 |
| DOI | 10.21203/rs.3.rs-8854256/v1 | Crossref; cover p.1 |
| Venue status | Research Square preprint, labelled "Research Article"; Crossref type `posted-content`; `relation` empty → **no journal/conference version found** | Crossref API record; cover p.1 |
| License | CC BY 4.0 | Cover p.1 |
| Declarations | No competing interests; no external funding; **"Generative AI assistance was used for language drafting and editing"**; contribution statement credits *all* roles (conceptualization, methodology, system design, manuscript preparation) to **Chinthakuntla alone**, despite four listed authors; data availability: **"No new datasets were created or analyzed in this study"**; code **not publicly released** | pp.1, 11–12 |

## 2. Full-text access and verification

- Phase 1 had only title-level access (P-01: "page returned title only to automated fetch"). This review used the **official PDF** obtained from the landing page's `citation_pdf_url` (`researchsquare.com/article/rs-8854256/latest.pdf`, 922,979 bytes), after the landing page (143 KB, with full `citation_*` meta tags) was retrieved with a browser User-Agent.
- Bibliographic identity cross-checked against the **Crossref API record** for the DOI and the landing page's own meta tags; all fields consistent. The Sciety aggregator mirrors the record.
- Local extraction: pypdf, **13/13 pages** extracted (`file(1)` misreports the page count — known harmless artifact). pypdf drops fi/ffi ligature glyphs ("veri cation" → "verification"); quotes below restore them. **The full body — abstract through references — was read completely.** This is a complete full-text review, not an abstract review.
- Paper structure: cover p.1; abstract p.2; §1 intro pp.2–3 (contributions p.3); §2 related work pp.3–4; §3 framework p.5 (Table 1 p.5); §4 Version-Aware Vector Space pp.6–7; §5 PDCA loop pp.7–8 (Algorithm 1 p.8); §6 deep-link navigator pp.8–9; §7 experimental methodology pp.9–10; §8 results pp.10–11; §9 limitations p.11; §10 conclusion p.11; declarations pp.11–12; 16 references pp.12–13. **No figures; two structured artifacts only (Table 1, Algorithm 1). There is no results table anywhere in the paper.**

## 3. Research problem and work type

The paper targets two failure modes in documentation RAG (abstract p.2; §1 pp.2–3): **temporal blindness** — "retrieval mixes incompatible versions of the same manual" — and **passive retrieval** — answers as static snippets that leave the user to locate and verify evidence. Task definition (§3.1 p.5): given query *q* and an **explicit target version v_t** over releases {v1…vk}, produce a response that "(a) is grounded in evidence from v_t unless a cross-version comparison is requested, (b) avoids contamination from versions different from v_t, and (c) enables rapid verification through navigation"; a second task class covers two-version semantic comparison (v_a, v_b).

**Work type:** a **system-design / architecture paper with a sketched prototype evaluation**. The four claimed contributions (p.3) are: action-first deep-link outputs; version-conditioned retrieval + semantic comparison "that prevents cross-version contamination"; a PDCA agent loop; and "an evaluation protocol for documentation tasks that measures time-to-insight, cross-version error rate, and deprecation hallucinations." Domain is **software/product documentation** (configuration references, migration guides, release branches) — not 3GPP, not telecom.

## 4. System architecture

- **Metadata-tagged ingestion** (§4.1 p.6): structure-aware chunking; tagger assigns M = {version, module, path, section_id, published_date} from repository structure/site selectors before embedding; supports filters such as "version = 2.2 and module = authentication".
- **Version routing and isolation** (§4.2 p.6): embeddings stored in **namespaces keyed by version**; the router selects the namespace for v_t; when v_t is unknown it may infer one, but "generation is gated behind an explicit version confirmation signal to avoid accidental mixing". A **secondary metadata filter** excludes pages marked legacy/deprecated/superseded within a namespace.
- **Cross-version alignment** (§4.3 pp.6–7): for comparisons, candidates C_a, C_b are retrieved independently per namespace; a **generative comparator** aligns conceptually related chunks (renames, relocations) and emits a structured diff: preserved / modified / removed-or-deprecated, each with navigation actions for both versions.
- **PDCA loop** (§5 pp.7–8; Algorithm 1 p.8): Plan decomposes intents (lookup, compare, navigate); Do retrieves top-k from the selected namespace with anchors; **Check scores candidates on four dimensions — semantic relevance to q, metadata consistency with v_t, recency vs target release, internal coherence** — and delays generation on predicted mismatch; Act performs **just-in-time acquisition** of the authoritative live page, optionally re-embedding it.
- **Deep-link navigator** (§6 pp.8–9): output actions A = (url, anchor, offset, hint) with DOM-fragment anchors and least-fragile fallback selectors; navigation replaces citations as the verification modality.
- **Implementation notes** (§7.4 pp.9–10): "version-partitioned embedding store, a retrieval evaluator, and a browser automation layer". **No LLM, embedding model, vector database, evaluator model, or judge is ever named**; the design "supports modular substitution".

## 5. Version handling — detailed assessment (highest priority)

Answers to the ten version-handling questions, each with its evidence location:

1. **Multi-version corpus?** Yes, by design: per-version namespaces over releases {v1…vk} (§3.1 p.5; §4.2 p.6). No actual corpus is named or sized (§7.1 p.9 names only "publicly accessible documentation sets").
2. **Version metadata preserved?** Yes, by design: M includes `version` and `published_date` at chunk level (§4.1 p.6).
3. **Retrieval conditioned on requested target version?** Yes, by design: namespace selection on v_t; inferred-version generation gated on explicit confirmation (§4.2 p.6; Algorithm 1 step 1, p.8).
4. **Distinguishes old-only / new-only / shared evidence?** **No.** The semantic-diff taxonomy (preserved/modified/removed, §4.3 pp.6–7) classifies *content change between two releases* for comparison tasks; it is never used to characterise *where a QA answer's supporting evidence exists*, and no QA task controls or reports evidence distribution.
5. **Detects conflicting requirements across versions?** Only as an online retrieval signal: Check's "internal coherence (absence of contradictory statements among top candidates)" (§5.3 p.7) operates within one retrieval set; cross-version conflict is otherwise handled by routing *around* it (isolation), not by evaluating behaviour under it. No conflict-conditioned evaluation exists.
6. **Answers evaluated against the correct target version?** **Defined but not performed.** §7.3 p.9 defines: "Cross-version error rate measures the fraction of answers grounded in evidence from the wrong release." **No value for this metric appears anywhere in the paper.** §8.2 p.10 reports only that contamination was "reduced … by construction" and wrong-version deprecation hallucinations "were not observed in the evaluated task suite once strict routing and secondary metadata filters were applied" — a zero-incident claim with no denominator.
7. **Cross-version contamination measured?** No (same evidence as item 6): the claim is architectural ("by construction") plus an unquantified observation. The judgment procedure for "grounded in evidence from the wrong release" (who judges, how) is never described.
8. **Version errors categorised/quantified?** No. No error taxonomy, no counts, no examples. (Contrast VersionRAG, which at least attributes 25% of naive-RAG errors to version mixing.)
9. **Metadata only for retrieval, or also for answer verification?** Retrieval routing (§4.2) **plus pre-generation evidence checking** ("metadata consistency with v_t", §5.3 p.7) **plus** traceability plumbing (Algorithm 1 step 5 returns evidence identifiers; actions carry anchors). But there is **no post-generation answer-level verification** — nothing checks that the generated answer's claims are supported by, or attributed to, the correct version. The defined cross-version error rate would be such a check, but it is never operationalised.
10. **Document version vs 3GPP Release vs spec revision?** Not applicable — "version" means product release branches of software manuals (v1, v2, "version = 2.2"). The paper makes no distinction comparable to 3GPP Release vs specification version vs editorial revision, and nothing addresses normative language.

**Bottom line (per the brief's warning):** DocNavigator offers multi-version *coverage* and version-conditioned *retrieval* by design, and it *defines* version-aware evaluation metrics — but it does **not demonstrate version-aware evaluation**: no metric values, no controlled evidence placement, no error analysis, no named data. Multi-version document coverage has not been equated with version-aware evaluation here; the paper itself does not cross that line evidentially.

## 6. Evaluation methodology — defined vs reported

- **Task suite** (§7.1 p.9): four classes — (a) version-specific lookup for a target release; (b) migration guidance v_a→v_b; (c) deprecation avoidance ("confirming that an interface or flag remains supported in v_t"); (d) semantic change detection under renaming. "Tasks are instantiated on publicly accessible documentation sets that provide multiple historical releases and stable URLs" — **the sets are never named; task counts are never given; no gold-labeling procedure is described.**
- **Baselines** (§7.2 p.9): (1) conventional RAG over a single flat index mixing all versions; (2) corrective RAG (CRAG-style evaluator, its ref [1]) **without** version partitioning or navigation. No models, configs, or parity controls stated; both return textual answers with citations.
- **Metrics** (§7.3 p.9): time-to-insight (query submission → confirmed correct supporting context); cross-version error rate (above); deprecation hallucination rate ("fraction of answers that recommend deprecated features when v_t indicates removal or replacement"); diagnostics (evaluator-triggered retrieval iterations; fraction of queries needing just-in-time acquisition).
- **Reported results** (§8 pp.10–11): **exactly one number in the entire paper** — "median time-to-insight decreased by 40% relative to conventional RAG" (§8.1 p.10). Everything else is qualitative ("consistently retrieved evidence from the target namespace", "were not observed", "frequently detected low coherence"). The two defined version metrics and both diagnostics are never reported numerically. There is no results table, no sample size, no variance, no significance testing, no human-participant protocol for the time measurements (which necessarily involve a human or simulated verifier — unspecified).
- **Statistics / human evaluation / IAA:** none of any kind.
- **Reproducibility:** irreproducible as published — no data (declared "No new datasets were created or analyzed", p.12), no code ("not publicly released", p.12), no named components, no prompts.
- **Internal contradiction worth recording:** the contribution list (p.3) claims "an evaluation protocol … that **measures** time-to-insight, cross-version error rate, and deprecation hallucinations", and §8 is titled Results — yet the data-availability declaration states no datasets were created **or analyzed**. The most charitable consistent reading is an informal internal prototype trial on third-party public docs, below the evidential bar of an empirical study.

## 7. Author claims vs demonstrated evidence

| Author claim (location) | Demonstrated? | Reviewer note |
|---|---|---|
| Version-conditioned retrieval "prevents cross-version contamination" (p.3; §8.2 p.10) | **No** — architectural argument ("by construction") + unquantified "not observed" | Plausible mechanism; namespace isolation does make cross-version retrieval impossible *if* routing and tagging are correct — but routing/tagging accuracy is exactly what is unmeasured (author limitation 2, §9 p.11, concedes tagging assumptions) |
| 40% median time-to-insight reduction (§8.1 p.10) | **Asserted single summary statistic** | No N, no task list, no timing protocol, no verifier identity, no variance; unverifiable |
| Deprecation hallucinations from wrong-version evidence eliminated (§8.2 p.10) | **No** — zero-incident claim, unreported denominator | "Not observed" over an unnamed task suite is not a rate |
| Evaluation protocol that "measures" three metrics (p.3) | **Partially** — protocol *described* (§7), measurement mostly absent (§8) | Two of three primary metrics never valued |
| Semantic diffs align renamed concepts (§8.3 p.10) | Qualitative anecdotes only ("in several cases…") | No alignment accuracy, no gold diffs |
| Closed loop prevents stale-context generation (§8.4 pp.10–11) | Qualitative only | Evaluator accuracy never measured |

**Not reported (explicit list):** corpus identity/size; task counts; any model names; gold procedure; all values of cross-version error rate, deprecation hallucination rate, retrieval iterations, JIT-acquisition fraction; baseline absolute numbers; statistics; annotator/participant details; prompts; code.

## 8. Limitations

**Author-stated** (§9 p.11): anchor fragility under site redesigns; version tagging assumes reliable release identifiers in URLs/repo layout; just-in-time acquisition faces rate limits, authentication and compliance constraints; evaluation limited to documentation QA (no ticket triage / code-change workflows).

**Reviewer-observed (beyond the above):** the evidential gap documented in §§6–7 of this review (one number total; defined-but-unmeasured version metrics; no data/code/models; declaration contradiction); contribution statement crediting one of four authors with every role (p.12) with the AI-use declaration referring to "the author" singular — an integrity question a thesis examiner would raise; reference-list quality is mixed (e.g., its ref [13] is a low-tier multidisciplinary journal; ref [12] "LoopRAG" appears in *Buildings*, an MDPI civil-engineering journal; ref [3]'s arXiv ID is malformed "arXiv:240713032"); none of this affects the design's conceptual content but it lowers the weight the paper can carry as prior art.

## 9. Relevance to NR-SpecRAG's six dimensions

| Dimension | DocNavigator coverage |
|---|---|
| Evidence faithfulness | Not evaluated. Pre-generation evidence gating exists (Check), but no claim-level support measurement of generated answers |
| Citation reliability | Not evaluated — citations are *replaced* by navigation actions (url, anchor, offset, hint); anchor-resolution success is never measured (anchor fragility conceded §9). No E1–E5 analogue |
| Calibrated abstention | Not evaluated. Generation delay below a confidence threshold (§5.3) and version-confirmation gating (§4.2) are described; no abstention rate, coverage/risk, or calibration numbers |
| Version-conditioned correctness | **Defined, not measured** (see §5, items 6–8) |
| Modality (normative language) | Absent — software docs; no normative/requirement-language handling |
| Controlled evidence conditions | Absent — no control over evidence distribution; no compute/config-matched baselines |

## 10. Six-cell comparison (C1 gating analysis)

Our approved §7 design: target version T ∈ {old, new} × evidence support S ∈ {old-only, new-only, both}, 16 items × arms {B, C}, metrics VFR/CVS/VCDR/VAA, with conflict defined only by real adjacent-release differences. Mapping DocNavigator's evaluation onto those cells:

| NR-SpecRAG cell | Nearest DocNavigator analogue | Equivalent? |
|---|---|---|
| T=old, S=old-only | Task (a) version-specific lookup with v_t = older release | **No** — evidence location never established or controlled; no per-condition results |
| T=new, S=new-only | Task (a) with v_t = newer release | **No** — same defects |
| T=new, S=old-only | Task (c) deprecation avoidance (support removed in v_t; wrong answer would come from old evidence) — closest conceptual relative; the deprecation hallucination rate targets exactly this failure | **No** — conceptually adjacent, but evidence placement uncontrolled and the metric is never valued ("not observed", no denominator) |
| T=old, S=new-only | — | **Not covered** — no task queries an old target with evidence existing only in the newer release |
| T=old, S=both (incl. real conflict) | Tasks (b)/(d) are *comparison* tasks (explicit v_a vs v_b), a different task class from QA under shared/conflicting evidence | **Not covered** as an evaluation condition |
| T=new, S=both (incl. real conflict) | Same | **Not covered** |

**Conclusion:** DocNavigator instantiates **zero of the six cells** as controlled, reported evaluation conditions. Its version evaluation is one-dimensional (target version only, via isolation), lacks the evidence-distribution axis entirely, and is quantitatively unreported. Its deprecation-avoidance task is the nearest conceptual relative of our (T=new, S=old-only) cell and should be cited as such — as a *defined task*, not a demonstrated result.

## 11. Cross-paper capability separation (with VersionRAG, TelcoAI, Chat3GPP)

Five distinct capabilities the brief requires separating ("D" = by design/described only; "M" = measured; "—" = absent):

| Capability | DocNavigator | VersionRAG (abstract-stage) | TelcoAI | Chat3GPP |
|---|---|---|---|---|
| 1. Multi-version ingestion | D (namespaces) | M (version graph, 34 docs) | M (metadata incl. release) | M (Rel-17+18, filename-only, no metadata) |
| 2. Version-filtered retrieval | D (router + legacy filter) | M (version-filtered search) | M (metadata filtering) | — |
| 3. Target-version-conditioned generation | D (confirmation-gated) | Partial (query-intent routing) | — (single current answer) | — |
| 4. Version-specific evidence attribution (per answer/citation) | D only (metric defined, unmeasured) | Partial, post-hoc (25% of naive-RAG errors attributed to version mixing) | — | — |
| 5. Version-conditioned correctness evaluation | **Defined, not measured** | **M on one axis** (accuracy on version-sensitive queries; 90% vs 58/64%) | — (cross-version comparison = stated future work) | Reporting split only (per-release accuracy; no mechanism, no conditioning) |

No reviewed system reaches capability 5 with an evidence-distribution factorial, conflict cells, or per-citation version attribution (our VAA/E4) — and none operates on normative 3GPP text with these capabilities (TelcoAI/Chat3GPP are 3GPP but sit at capabilities 1–2 and 1 respectively). VersionRAG remains the strongest *measured* prior (single-axis, software docs, abstract-stage knowledge — full text still pending per S-12).

## 12. C1–C4 reassessment

- **C1 (version-conditioned trust evaluation on 3GPP): NARROWED in concept, INTACT in substance — gating condition resolved.** What this paper changes: the *idea* of wrong-release grounding as a named metric is now independently published prior art (its §7.3 definition predates us in print), so C1 may no longer be phrased as "version-conditioned correctness has not been considered." What it does not change: no prior work — this one included — *measures* version-conditioned correctness under controlled target-version × evidence-distribution conditions, with real-conflict cells, per-citation version attribution, on 3GPP specifications. DocNavigator strengthens C1's motivation (a second independent group names the failure mode VersionRAG quantified) while leaving the measurement gap open, because it reports no values. **Required wording discipline:** C1 statements must now cite DocNavigator (metric defined, unmeasured, software docs, non-peer-reviewed) and VersionRAG (measured, single-axis, software docs) as closest prior work, and claim only the controlled-evaluation + attribution + 3GPP delta.
- **C2 (citation-resolution reliability, E1–E5): UNCHANGED — remains the strongest gap.** DocNavigator replaces citations with navigation actions and never measures anchor-resolution success; no reviewed work measures citation resolution, and E4 (right clause, wrong version) still has no analogue anywhere.
- **C3 (answer-blind sufficiency vs verification): UNCHANGED.** The PDCA Check is an *online*, answer-blind sufficiency gate — conceptually adjacent to our Module S — but it is a control mechanism with no measured accuracy and no gold sufficiency labels (same status as the CRAG lineage already screened, P-06). Evaluation-side sufficiency measurement remains absent from the reviewed corpus.
- **C4 (calibrated abstention): UNCHANGED.** Confidence-thresholded generation delay and version-confirmation gating are described but never measured (no abstention rate, coverage, or risk); DocNavigator joins Chat3GPP as a system whose abstention behaviour exists only as an uninstrumented mechanism. The generic abstention literature (RINSE, SURE-RAG, ESB, Sufficient Context, GRAB-RAG) still bounds C4's novelty, not this paper.

## 13. Unresolved questions

1. What data did §8's numbers-free "prototype evaluation" actually run on, given the "no datasets were created or analyzed" declaration? (Unanswerable from the paper.)
2. Whether a journal version with real results appears later — Crossref relation empty as of 2026-11-10; **re-check before thesis submission.**
3. Whether the three non-contributing listed authors reflect an administrative artifact or an authorship-practice issue (relevant only to the weight given this preprint).
4. VersionRAG full text (S-12, still pending) — now the decisive C1 prior; its six-cell proximity must be assessed from its full text, not its abstract.

## 14. Reviewer confidence

**High** on everything stated here: the full 13-page text was read from the official PDF, identity primary-verified (Crossref + landing-page meta tags), and every finding carries a page/section anchor. The paper's own evidential weight is **low** (non-peer-reviewed v1; one reported number; no data/code/models) — it functions as *conceptual* prior art and motivation, not as empirical precedent. Remaining verification work: none for this paper beyond the journal-version re-check; the open C1 dependency is now VersionRAG's full text.
