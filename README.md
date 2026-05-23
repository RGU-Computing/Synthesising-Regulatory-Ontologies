# Synthesising Regulatory Ontologies: A Schema and Pattern-Guided Approach

This repository contains the code, ontologies, prompts, and evaluation materials accompanying the paper:

**Synthesising Regulatory Ontologies: A Schema and Pattern-Guided Approach**  
Umair Arshad, David Corsar, Ikechukwu Nkisi-Orji  
*Robert Gordon University, Aberdeen, United Kingdom*  
Submitted to EKAW 2026.

---

## Overview

We present a schema- and pattern-guided approach for regulatory ontology synthesis, in which LLM extraction is constrained directly within a target ontology rather than mapped to it afterward. The method integrates the [Core Compliance Ontology (CCO)](https://github.com/RGU-Computing/CCO) with twelve extraction patterns and two consolidation patterns, combining two-stage retrieval, two-step LLM-driven extraction with SHACL validation and deterministic repair, and a consolidation phase that produces both jurisdiction-scoped and shared domain regulatory abstractions under OWL RL reasoning.

The approach is demonstrated on education funding regulations across four jurisdictions (United Kingdom, United States, Canada, Australia), producing a unified legislation regulatory ontology of 1,295 named classes layered over CCO.

<div align="center">
<img src="fig/model_pipeline.png" width="800" alt="A Schema and Pattern-Guided Approach"/>
</div>

*Figure 1: Overview of the proposed four-phase schema- and pattern-guided pipeline for regulatory ontology synthesis, comprising preparation; retrieval, extraction and validation, and consolidation and evaluation. Blocks marked with the AI icon denote stages in which LLMs are used.*


---

## Documentation

Full ontology documentation is available at:
**[Unified Legislation Regulatory Ontology](https://rgu-computing.github.io/Synthesising-Regulatory-Ontologies/)**

**Note:** The published documentation shows the ontology schema (classes, properties, hierarchy) only. Individual instances (extracted RDF data) are available in the TTL files under `ontology/`.

---

## Repository Structure

```
.
├── ontology/                    # Synthesised regulatory ontology
│   ├── per-jurisdiction/         # Per-jurisdiction ontologies (UK, US, CA, AU)
│   ├── shared/                   # Domain regulatory ontology (69 shared classes)
│   ├── alignments/               # SKOS cross-jurisdiction mappings (362)
│   └── unified/                  # Unified legislation regulatory ontology
│
├── schema-and-patterns/             # Schema and pattern artefacts
│   ├── cco_schema.json               # CCO base schema (classes + properties whitelist)
│   ├── extraction_odps.json          # 12 extraction ODPs
│   └── consolidation_patterns.md     # Functional equivalence + shared domain superclass
│
├── prompts/                     # LLM prompts (Phase 3 + Phase 4)
│   ├── decomposition_prompt.txt
│   ├── generation_prompt.txt
│   ├── alignment_prompt.txt
│   └── shared_naming_prompt.txt
│
├── shacl-shapes/                # SHACL validation shapes
│   └── cco_shapes.ttl
│
├── evaluation/                  # Evaluation materials
│   ├── quality_metrics/          # Domain ontology structural metrics
│   ├── ontology_correctness/     # 69-class correctness review
│   └── source_faithfulness/      # 59-sample faithfulness annotations
│
├── code/                        # Pipeline implementation (Jupyter notebooks)
│   ├── phase1_preparation/
│   ├── phase2_retrieval/
│   ├── phase3_extraction_validation/
│   └── phase4_consolidation/
│
├── data/                        # Source document manifests
│   └── manifests/                # Document inventory and URLs
│
├── LICENSE
└── README.md
```

## Key Results

### Dataset

| Metric | Value |
|---|---|
| Jurisdictions | 4 (United Kingdom, United States, Canada, Australia) |
| Source provisions processed | 3,580 |
| Per-jurisdiction breakdown (UK / CA / AU / US) | 1,225 / 886 / 1,055 / 414 |

### Pipeline Inputs

| Metric | Value |
|---|---|
| CCO classes | 16 |
| CCO object properties | 14 |
| CCO data properties | 8 |
| Extraction ODPs | 12 |
| Consolidation patterns | 2 |
| Architectural rules in prompts | 18 |

### Retrieval (Phase 2)

| Metric | Value |
|---|---|
| Similarity threshold | 0.60 |
| Schema candidates (k_s) | 8 |
| ODP candidates (k_p) | 5 |
| Schema-gate exclusion list size | 9 CCO labels |
| Chunks selected after retrieval | 1,291 |
| Selected (UK / CA / AU / US) | 543 / 321 / 338 / 89 |

### Extraction and Validation (Phase 3)

| Metric | Value |
|---|---|
| Extractable chunks | 1,178 |
| Extraction rate | 91.2% |
| Initial SHACL pass rate | 81.9% (965/1,178) |
| Deterministic repair functions | 6 |
| **SHACL pass rate (after repair)** | **95.8% (1,128/1,178)** |

### Consolidation (Phase 4)

| Metric | Value |
|---|---|
| Per-jurisdiction dedup threshold | 0.85 |
| Candidate pairs reviewed | 189 |
| Approved / Rejected | 183 / 6 |
| Raw class declarations | 2,109 |
| After string-matching dedup | 1,154 |
| Semantic clusters formed | 83 |
| Classes merged in clusters | 154 |
| **Canonical jurisdiction-scoped classes** | **1,000** |
| Canonical (UK / CA / AU / US) | 379 / 271 / 264 / 86 |
| Cross-jurisdiction alignment threshold | 0.70 |
| Embedding candidate pairs | 949 |
| **SKOS alignments produced** | **362** (357 closeMatch, 3 narrowMatch, 2 broadMatch) |
| Shared cross-jurisdiction classes | 69 (across 10 CCO parents) |

### Unified Ontology

| Metric | Value |
|---|---|
| **Total named classes** | **1,295** |
| CCO classes (foundation) | 16 |
| Shared classes (domain regulatory ontology) | 69 |
| Jurisdiction-scoped classes | 1,210 (1,000 canonical + 210 retained from per-chunk RDF) |
| Hierarchy depth | 3 levels |
| Average branching factor | 6.04 |
| Max branching factor | 46 |
| Annotation completeness (labels) | 100% |
| Annotation completeness (definitions) | 100% |
| Annotation completeness (comments) | 63.8% |

### Cross-Jurisdiction Coverage (69 shared classes)

| Coverage | Count | Percentage |
|---|---|---|
| 4 jurisdictions | 6 | 13.6% |
| 3 jurisdictions | 7 | 15.9% |
| 2 jurisdictions | 30 | 68.2% |
| 1 jurisdiction | 1 | 2.3% |
| Norm-type abstractions (no direct subclasses) | 25 | — |

### Evaluation: Ontology Correctness (69 shared classes)

| Criterion | Correct | Partial | Incorrect |
|---|---|---|---|
| Name appropriateness | 64 (92.8%) | 5 (7.2%) | 0 |
| Definition accuracy | 58 (84.1%) | 11 (15.9%) | 0 |
| Parent correctness | 65 (94.2%) | 3 (4.3%) | 1 (1.4%) |
| **Fully correct (all three)** | **50 (72.5%)** | — | — |

### Evaluation: Source Faithfulness (59 stratified samples)

| Axis | Faithful | Partial | Unfaithful |
|---|---|---|---|
| Subject | 84.7% | 15.3% | 0.0% |
| Predicate | 64.4% | 33.9% | 1.7% |
| Object | 72.9% | 27.1% | 0.0% |
| **Overall** | **55.9%** | **42.4%** | **1.7%** |

## Evaluation Summary

The synthesised ontology is evaluated along three dimensions:

- **Quality Metrics** — Domain regulatory ontology contains 69 shared 
  classes across 10 CCO parents, with 257 per-jurisdiction descendants. 
  Annotation completeness: 100% labels, 100% definitions, 63.8% comments. 
  3-level hierarchy with average branching factor of 6.04 (max 46).

- **Ontology Correctness** — 69 shared classes human-reviewed against 
  three criteria. **72.5% (50/69) rated Correct across all criteria.** 
  Parent correctness: 94.2% Correct; Name appropriateness: 92.8% Correct; 
  Definition accuracy: 84.1% Correct (lowest, dominated by definitions 
  aggregating multiple concepts).

- **Source Faithfulness** — 59 stratified samples (5% of 1,178 validated 
  chunks) rated along subject, predicate, and object axes. **Overall: 
  55.9% Faithful, 42.4% Partial, 1.7% Unfaithful.** Subjects most 
  reliably extracted (84.7% Faithful), followed by objects (72.9%) and 
  predicates (64.4%). Partial ratings dominated by multi-clause provisions 
  with lower-granularity capture of secondary clauses.

---

## Browsing the Ontology

The main artefacts are TTL files in `ontology/`:

- **`ontology/unified/unified_legislation_regulatory.ttl`** — Complete unified ontology (CCO + 69 shared classes + 1,210 jurisdiction-scoped classes + 362 SKOS alignments)
- **`ontology/shared/domain_regulatory.ttl`** — Domain regulatory ontology (69 shared classes only)
- **`ontology/per-jurisdiction/gro-{uk,us,ca,au}.ttl`** — Per-jurisdiction ontologies

You can browse these in any ontology editor (Protégé, WebVOWL) or RDF tool.

---

### Requirements

- Python 3.10+
- See `requirements.txt` for dependencies

### Pipeline Stages

The pipeline is implemented as Jupyter notebooks in `code/`, organised by pipeline phase:

1. **Phase 1 (Preparation):** Document chunking, embedding index construction
2. **Phase 2 (Retrieval):** Schema-grounded gating + pattern-aware ODP retrieval
3. **Phase 3 (Extraction & Validation):** Provision decomposition, ontology generation, SHACL validation, deterministic repair
4. **Phase 4 (Consolidation):** Per-jurisdiction deduplication, cross-jurisdiction alignment, domain regulatory ontology, merging with OWL RL reasoning

---

## License

This work is licensed under the [MIT License](LICENSE).


## Citation

```
@misc{arshad2026synthesising,
  author = {Arshad, Umair and Corsar, David and Nkisi-Orji, Ikechukwu},
  title  = {Synthesising Regulatory Ontologies: A Schema and Pattern-Guided Approach},
  year   = {2026},
  url    = {https://github.com/RGU-Computing/Synthesising-Regulatory-Ontologies}
}
```
