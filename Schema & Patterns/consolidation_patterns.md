# Consolidation Patterns

Two patterns guide the consolidation phase of the schema- and 
pattern-guided ontology synthesis pipeline. They complement the 
twelve extraction ODPs by addressing cross-jurisdiction concept 
alignment and shared abstraction introduction.

---

## Pattern 1: Functional Equivalence Pattern

**Purpose:** Decide whether two domain classes from different 
regulatory jurisdictions are functionally equivalent, and 
determine the appropriate alignment relation.

**Applicability:** Two classes from different jurisdictions that 
embedding similarity suggests may refer to the same or related 
regulatory concept.

### Equivalence Criteria

Each criterion is checked independently. Alignment is asserted 
only if **all five** criteria pass.

1. **same_cco_parent** — Both classes share the same CCO parent 
   class. PASS only if both have the exact same parent.

2. **same_regulated_domain** — Both regulate the same kind of 
   activity or entity in their respective jurisdictions. PASS 
   if both deal with the same regulatory subject area; FAIL if 
   different subject areas.

3. **same_normative_role** — Both play the same structural role 
   in normative statements. PASS if both are norm subjects, OR 
   both are norm objects, OR both are conditions, etc.

4. **compatible_scope** — Both apply at the same scope level. 
   PASS if both are individual-level, OR both organisational-level, 
   OR both procedural-level.

5. **swap_test** — A regulation written about one class could 
   be sensibly rewritten substituting the other class, preserving 
   the regulatory STRUCTURE (e.g., "obligation-on-party-X-to-do-Y" 
   pattern). PASS if substitution maintains the 
   obligation/permission/prohibition pattern AND the regulated 
   parties are equivalent types, even if the specific action 
   differs. FAIL ONLY if substitution changes the normative 
   type or entity types.

### Alignment Relation Choice

Applied only when all five criteria PASS. One of:

- **`owl:equivalentClass`** — Used only if classes are legally 
  AND functionally identical (same statute, same regulatory 
  body, same legal effect). Extremely rare for cross-jurisdiction 
  alignment.

- **`skos:closeMatch`** — Default for cross-jurisdiction 
  alignment. Used when classes are functionally similar but 
  exist in different legal frameworks (UK vs US vs CA vs AU 
  laws). Almost all cross-jurisdiction alignments use this.

- **`skos:broadMatch`** — Class A is BROADER than Class B 
  (A conceptually subsumes B).

- **`skos:narrowMatch`** — Class A is NARROWER than Class B 
  (B conceptually subsumes A).

### Decision Rule

- `decision = "ALIGN"` only if ALL FIVE criteria PASS.
- `decision = "REJECT"` if ANY criterion fails.
- If REJECT, `alignment_relation` is `null`.
- If ALIGN, choose appropriate `alignment_relation` (default: 
  `skos:closeMatch`).

### Operationalisation

### LLM Output Format

```json
{
  "decision": "ALIGN|REJECT",
  "alignment_relation": "owl:equivalentClass|skos:closeMatch|skos:broadMatch|skos:narrowMatch|null",
  "criteria": {
    "same_cco_parent":       { "result": "PASS|FAIL", "justification": "1 sentence" },
    "same_regulated_domain": { "result": "PASS|FAIL", "justification": "1 sentence" },
    "same_normative_role":   { "result": "PASS|FAIL", "justification": "1 sentence" },
    "compatible_scope":      { "result": "PASS|FAIL", "justification": "1 sentence" },
    "swap_test":             { "result": "PASS|FAIL", "justification": "1 sentence" }
  },
  "overall_justification": "2-3 sentence summary of the decision"
}
```

The full prompt is in [`prompts/alignment_prompt.txt`](../prompts/alignment_prompt.txt).

---

## Pattern 2: Shared Domain Superclass Pattern

**Purpose:** Introduce a shared superclass for cross-jurisdiction 
equivalent classes, generalising recurring regulatory concepts 
without jurisdiction-specific commitments.

**Trigger:** At least two confirmed Functional Equivalence 
alignments with compatible CCO supertypes.

### OWL Axioms (must hold)

1. **Subclass of CCO supertype.** The shared class must be a 
   subclass of the confirmed CCO supertype. It cannot be 
   introduced as a new root class above CCO.

2. **Universal axioms only.** Only axioms that hold universally 
   across all aligned jurisdictions are placed on the shared 
   class. Jurisdiction-specific axioms are retained at subclass 
   level.

3. **CCO disjointness preserved.** CCO disjointness axioms 
   remain in force. A shared class cannot be typed as both 
   `cco:Obligation` and `cco:Permission` — the 
   `AllDisjointClasses` axiom forbids this.

4. **Domain/range satisfaction.** No axiom involving a CCO 
   property whose domain or range constraint is not satisfied 
   across all aligned jurisdictions is placed on the shared 
   class.

### Review Criteria

- **Jurisdiction-neutral label.** The label must NOT contain 
  jurisdiction-specific vocabulary (no `Apprenticeship`, `VET`, 
  `HELP`, `Canada Student Loan`, `Pell Grant`, `Title IV`, or 
  equivalent jurisdiction-specific program names).

- **Universal axioms.** Every axiom on the shared class must 
  hold without exception across all aligned jurisdictions.

- **CCO-grounded position.** The shared class must be a subclass 
  of the confirmed CCO supertype — not a new root class.

### Operationalisation

### Constraints on LLM Output

- **Name** must be CamelCase, descriptive, jurisdiction-neutral.
- **Label** is 2-4 words, human-readable.
- **Definition** is 1-2 sentences and must NOT mention 
  jurisdiction-specific programs.

### LLM Output Format

```json
{
  "shared_class_name": "CamelCaseName",
  "label": "Human Readable Label",
  "definition": "Jurisdiction-neutral definition capturing the universal concept (1-2 sentences).",
  "naming_justification": "1-2 sentences explaining the chosen name and how it satisfies the pattern's review criteria."
}
```

The full prompt is in [`prompts/shared_naming_prompt.txt`](../prompts/shared_naming_prompt.txt).

---

## Pattern Interaction

The two patterns are applied sequentially:

1. **Pattern 1** (Functional Equivalence) first identifies 
   equivalent or related classes across jurisdictions and 
   materialises SKOS or OWL alignment relations.

2. **Pattern 2** (Shared Domain Superclass) then introduces 
   shared abstractions for clusters of mutually aligned classes 
   sharing a common CCO parent.

Together, they produce the **domain regulatory ontology** layer, 
which captures recurring regulatory concepts at a jurisdiction-
neutral level while preserving jurisdiction-specific content in 
the per-jurisdiction subclasses.

---

## Application Results (Education Funding Demonstration)

Applied across four jurisdictions (UK, US, CA, AU):

| Metric | Value |
|---|---|
| Embedding candidate pairs | 949 |
| SKOS alignments produced | 362 |
| - `skos:closeMatch` | 357 |
| - `skos:narrowMatch` | 3 |
| - `skos:broadMatch` | 2 |
| Shared classes introduced | 69 |
| CCO parents covered | 10 |

Human review of the 69 shared classes against three correctness 
criteria (name appropriateness, definition accuracy, parent 
correctness) is available in 
[`evaluation/ontology_correctness/`](../evaluation/ontology_correctness/).
