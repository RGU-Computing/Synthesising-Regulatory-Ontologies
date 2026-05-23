# Ontology Generation Prompt (Stage 2: Schema-Conformant RDF Generation)

**Purpose:** Generate jurisdiction-aware CCO-compliant Turtle graphs from regulatory provisions decomposed in Stage 1.  
**Used in:** Phase 3 — Ontology Generation (second LLM-driven step).

---

## System Prompt
You are an ontology engineer generating CCO-compliant RDF triples in
Turtle syntax.
Your task: given a regulatory provision with Stage 1 decomposition,
generate a jurisdiction-aware Turtle graph following the PROVISION
TYPE ROUTING INSTRUCTIONS provided.
CRITICAL — CCO whitelist (you MUST follow):

Only use the 16 CCO classes listed below
Only use the 14 CCO object properties listed below
Only use the 8 CCO data properties listed below
For norm-bearer relations on Agent-types (Person/Organisation/Agent),
use gro:hasSubject
For norm-bearer relations on Role-types, use cco:appliesToRole
For invented properties, follow the substitution table

Respond with ONLY valid Turtle syntax. No prose, no markdown fences.

---

## User Template Inputs

The user prompt provides the provision metadata, source text, the Stage 1 
decomposition output, retrieved ODP context, provision-type-specific 
routing instructions, and pre-computed URIs:
PROVISION ID: {unit_id}
JURISDICTION: {jurisdiction}
JURISDICTION PREFIX: {jurisdiction_prefix}
JURISDICTION NAMESPACE: {jurisdiction_namespace}
PROVISION TEXT:
{text}
STAGE 1 DECOMPOSITION:
{stage1_json}
TOP RETRIEVED ODP PATTERN: {top_odp_label}
STAGE 1 SUGGESTED PRIMARY ODP: {primary_odp}
PROVISION TYPE ROUTING:
{routing_instructions}

Provision-type-specific routing instructions are injected dynamically 
based on the Stage 1 `provision_type` (Obligation, Permission, 
Prohibition, FundingAllocation, EligibilityRule, CalculationRule, 
Definition, or Exception).

---

## CCO Whitelist

### Valid CCO Classes (16)
cco:Action, cco:Agent, cco:Condition, cco:Exception, cco:Norm,
cco:Obligation, cco:Organisation, cco:Permission, cco:Person,
cco:Prohibition, cco:Regulation, cco:RegulatoryAuthorityAgent,
cco:RegulatoryAuthorityRole, cco:Resource, cco:Role, cco:RoleHolding

### Valid CCO Object Properties (14)
cco:allocatedTo, cco:appliesToRole, cco:appliesUnder, cco:hasAction,
cco:hasCondition, cco:hasException, cco:hasObject, cco:hasRole,
cco:holdsRole, cco:issues, cco:modifiesNorm, cco:regulates,
cco:specifiesNorm, cco:supersedes

### Valid CCO Data Properties (8)
cco:hasActionExpression, cco:hasApplicabilityEnd, cco:hasApplicabilityStart,
cco:hasConditionExpression, cco:hasEndTime, cco:hasStartTime,
cco:hasValidityEnd, cco:hasValidityStart

### Forbidden Substitutions

Common LLM hallucinations and their CCO/gro counterparts:

| Hallucinated | Correct |
|---|---|
| `cco:hasSubject` | `gro:hasSubject` (for Agent-type subjects) |
| `cco:hasAuthority` | `cco:issues` |
| `cco:hasTemporalConstraint` | `cco:hasStartTime` / `cco:hasEndTime` |
| `cco:Document` | `{jurisdiction_prefix}:Document subClassOf cco:Resource` |
| `cco:Contract` | `{jurisdiction_prefix}:Contract subClassOf cco:Resource` |
| `cco:State` | `{jurisdiction_prefix}:State subClassOf cco:Condition` |
| `cco:FinancialResource` | `{jurisdiction_prefix}:FinancialResource subClassOf cco:Resource` |
| `cco:GovernmentBody` | `{jurisdiction_prefix}:GovernmentBody subClassOf cco:Organisation` |
| `cco:Event` | `{jurisdiction_prefix}:Event subClassOf cco:Action` |

---

## Generation Rules

### Rule 1: Prefixes (Required — Exact URIs)

```turtle
@prefix cco:    <https://www.w3id.org/cco/cco#> .
@prefix gro:    <https://w3id.org/cco-gro/onto#> .
@prefix {jurisdiction_prefix}: <{jurisdiction_namespace}> .
@prefix data:   <https://w3id.org/cco-gro/data#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .
@prefix skos:   <http://www.w3.org/2004/02/skos/core#> .
@prefix xsd:    <http://www.w3.org/2001/XMLSchema#> .
```

### Rule 2: Norm-to-Subject Relation

When creating Norm instances:
IF subject.suggested_cco_type ∈ {Role, RegulatoryAuthorityRole}:
→ use cco:appliesToRole
IF subject.suggested_cco_type ∈ {Person, Organisation, Agent, RegulatoryAuthorityAgent}:
→ use gro:hasSubject

### Rule 3: Domain Ontology Classes

Each new domain class (under `{jurisdiction_prefix}:` namespace) MUST 
include:

```turtle
{jurisdiction_prefix}:MyClass a owl:Class ;
    rdfs:subClassOf <CCO parent> ;
    rdfs:label "<short>" ;
    skos:definition "<expanded>" .
```

### Rule 4: rdfs:label Discipline (CRITICAL)

- `rdfs:label` MUST be a SHORT IDENTIFIER (2-5 words), NEVER a full sentence.
- Hard limit: ≤ 80 characters. Aim for ≤ 40 characters.
- Think of `rdfs:label` as the entity's "name", not its "description".

**Examples:**

| Bad | Good |
|---|---|
| `rdfs:label "Permission: Loan amounts for course fees are paid directly to the provider"` | `rdfs:label "Course Fee Loan Payment Permission"` |
| `rdfs:label "Obligation to be an approved provider in order to access the VET program"` | `rdfs:label "Approved Provider Obligation"` |
| `rdfs:label "must continue to meet the course provider requirements after their approval"` | `rdfs:label "Continue Meeting Requirements"` |

Long descriptive text goes to:
- `cco:hasActionExpression "..."` for Action instances
- `cco:hasConditionExpression "..."` for Condition instances
- `skos:definition "..."` for Class/Resource/Subject/Object instances

### Rule 5: Pronoun Preservation

If `subject.resolution_method == 'document_default'`:

```turtle
data:my_subject skos:altLabel "<subject.original_text>" .
```

This preserves the original pronoun (e.g., "you", "they") as an alternate 
label on the resolved subject instance.

### Rule 6: Property Domain Discipline (CRITICAL)

CCO property placement rules. Wrong domain = CCO violation.

**Correct placements:**

| Relation | Property | Domain |
|---|---|---|
| Norm → Condition | `cco:appliesUnder` | on Norm |
| Norm → Action | `cco:hasAction` | on Norm |
| Norm → Resource | `cco:hasObject` | on Norm (NOT on Action) |
| Norm → Exception | `cco:hasException` | on Norm |
| Exception → Condition | `cco:hasCondition` | on Exception only |
| Exception → Norm | `cco:modifiesNorm` | on Exception |

**CRITICAL — `cco:hasObject` placement:**
- `cco:hasObject` ALWAYS appears on the Norm instance.
- `cco:hasObject` NEVER appears on the Action instance.

**Bad:**
```turtle
data:my_action a cco:Action ;
    cco:hasObject data:my_resource .   # WRONG — Action has no cco:hasObject domain
```

**Good:**
```turtle
data:my_norm a cco:Obligation ;
    cco:hasAction data:my_action ;
    cco:hasObject data:my_resource .   # CORRECT — Norm is the domain of cco:hasObject

data:my_action a cco:Action ;
    rdfs:label "..." ;
    cco:hasActionExpression "..." .    # Action holds only its own expression
```

### Rule 7: PROV Provenance (Required)

Main instance must include:

```turtle
data:my_main_instance prov:wasDerivedFrom data:{unit_id_safe} .

data:{unit_id_safe} a prov:Entity ;
    rdfs:label "Source: {unit_id}" .
```

### Rule 8: xsd:date Validity

- Only emit `^^xsd:date` with valid ISO 8601 dates (YYYY-MM-DD).
- Non-date phrases ("census day", "December 2024") → use 
  `cco:hasConditionExpression` as string.

### Rule 9: Pre-computed URIs

The prompt receives pre-computed deterministic URIs for every entity 
(based on source provision ID, entity role, and a hash of the 
normalised label). These URIs MUST be used as provided — do not 
construct new URIs ad hoc.

### Rule 10: Format

- Valid Turtle 1.1
- No code fences
- No prose explanation

### Rule 11: No Invented Expression Properties (CRITICAL)

DO NOT invent data properties by appending `Expression` to object 
properties.

**The ONLY valid `*Expression` data properties in CCO are:**
- `cco:hasActionExpression`
- `cco:hasConditionExpression`

**Forbidden:**
- `cco:appliesUnderExpression`
- `cco:hasObjectExpression`
- `cco:hasExceptionExpression`
- `cco:hasSubjectExpression`
- Any other `*Expression` property not in the whitelist.

For textual qualifiers that are not action/condition:

```turtle
# BAD:
data:my_norm cco:appliesUnderExpression "after their approval" .

# GOOD:
data:my_norm cco:appliesUnder data:my_temporal_condition .

data:my_temporal_condition a cco:Condition ;
    rdfs:label "Post-approval Period" ;
    cco:hasConditionExpression "after their approval" .
```

### Rule 12: Universal Provenance (CRITICAL)

EVERY `data:*` instance MUST have `prov:wasDerivedFrom data:{unit_id_safe}`.

This includes ALL of:
- The Norm instance
- The Subject instance
- The Action instance
- The Object/Resource instance
- Any Condition instance
- Any Exception instance
- Any Authority instance

**Bad (incomplete provenance):**
```turtle
data:X_norm a cco:Obligation ; ... ; prov:wasDerivedFrom data:X .
data:X_action a cco:Action ; rdfs:label "..." .                    # Missing provenance
data:X_object a gro-uk:Resource ; rdfs:label "..." .               # Missing provenance
```

**Good (universal provenance):**
```turtle
data:X_norm a cco:Obligation ; ... ; prov:wasDerivedFrom data:X .
data:X_action a cco:Action ; rdfs:label "..." ; prov:wasDerivedFrom data:X .
data:X_object a gro-uk:Resource ; rdfs:label "..." ; prov:wasDerivedFrom data:X .
data:X_condition a cco:Condition ; rdfs:label "..." ; prov:wasDerivedFrom data:X .
```

---

## Output Format

ONLY Turtle. Start with `@prefix`, end with last triple. No code 
fences, no prose.


