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