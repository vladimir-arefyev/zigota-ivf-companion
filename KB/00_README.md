# Zigota KB Extraction Packet

A self-contained job spec for a constrained worker model to build the Zigota patient-education
knowledge base by EXTRACTING from approved sources (not writing from its own knowledge).

## Files (give all six to the worker, in this order)
- 01_INSTRUCTIONS.md — the task, the steps, the hard rules
- 02_SOURCES.md      — the only allowed sources, precedence, access fallback
- 03_OUT_OF_SCOPE.md — what to refuse + the refusal stubs to emit first
- 04_KB_STRUCTURE.md — the exact YAML schema, field by field
- 05_EXAMPLES.md     — one gold-standard entry per kind (the quality bar)
- 06_SELF_CHECK.md   — the checklist run on every entry before emitting

## Also provide as input
- The research report file
  (Project_Zigota__IVF_ART_Patient-Education_Knowledge_Base_and_Three-Phase_Protocol_Model.md)
  — the extraction fallback when a URL won't load.
- The AVA-Peter card file (SC_PatientCardA6_18sept2019.pdf) — for clinic contact numbers
  (warning verbatim_text) and to confirm the real drug/instruction list.

## Run configuration (decisions locked for this run)
- Source access: worker fetches the URLs itself; falls back to the report if a URL fails.
- Output format: YAML.
- Scope: full three-phase arc, ALL protocol types (antagonist / long-agonist / flare / generic).

## Output
One YAML document of KB entries, grouped by kind, each passing 06_SELF_CHECK.md.
`review.validated_by` / `reviewed_on` are null until a domain-literate human validates them —
that human validation pass is what earns the word "vetted" and is required before the KB ships.

## Where this sits in the build
Phase 1 (Domain & requirements). The extracted KB feeds three product surfaces — grounded Q&A
(citation-or-refuse), symptom capture (capture-and-escalate, Type D2), and prescription parsing
(class-only medication recognition) — as defined in Zigota_Protocol_Model_v2.md.
