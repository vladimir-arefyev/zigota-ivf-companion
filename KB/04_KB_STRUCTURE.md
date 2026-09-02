# KB Structure (output schema)

Emit YAML. Every entry shares the common envelope; each kind adds only its own fields.
Field order matters — follow it exactly. For unknown/empty optional fields, use `null` (do not omit).

## Common envelope (every entry)
```
- id:            # stable key, dot.namespaced. e.g. stage.trigger, glossary.follicle
  kind:          # glossary | stage | medication | instruction | warning | out_of_scope
  phase:         # 1 | 2 | 3 | cross   (which cycle phase; 'cross' = spans phases)
  topic:         # short human label
  aliases:       # list of patient phrasings + multilingual terms used for retrieval
    - ...
  body:          # vetted patient-facing text, paraphrased, short (<= ~60 words)
  scope:         # in_scope | out_of_scope
  confidence:    # agreed | ranged | single_source | n/a (n/a for out_of_scope)
  source:
    cite:        # list of {name, url_or_file, access}. access: live | from_report | clinic_card
      - name:    # e.g. ASRM booklet
        ref:     # URL or file name
        access:  # live | from_report | clinic_card
  review:
    validated_by:  # null at extraction time (human fills later)
    reviewed_on:   # null at extraction time
    source_dated:  # the source's own review/publication date if known, else null
```

## kind: glossary  (adds nothing)
Envelope only. `body` = the plain definition.

## kind: stage  (adds value_range)
```
  value_range:     # null, OR structured range with per-source attribution:
    metric:        # e.g. trigger_to_retrieval_hours
    values:        # list of {source, value}
      - source: ASRM booklet
        value: "34-36 h"
      - source: SART
        value: "36-40 h (ovulation)"
```
Use value_range whenever sources give differing numbers. If they agree, give a single value entry
and set confidence: agreed.

## kind: medication  (adds class fields — NO dose field exists)
```
  drug_class:        # gonadotropin | gnrh_agonist | gnrh_antagonist | trigger_hcg |
                     # trigger_agonist | progesterone | estrogen
  names:             # brand/generic, multilingual
    - ...
  purpose:           # class-level, dose-free ("stimulates the ovaries to grow follicles")
  alternatives_group:# tag for interchangeable fills (e.g. "luteal_progesterone"); null if none
```
There is deliberately NO dose/quantity/schedule field. Do not add one.

## kind: instruction  (adds unmet_action)
```
  unmet_action:      # what the app does if patient reports not following it.
                     # usually: escalate_to_clinic
```

## kind: warning  (adds tier, verbatim_text, match_patterns)
```
  tier:            # contact_clinic | urgent_care
  match_patterns:  # symptom phrasings that trigger this entry
    - ...
  verbatim_text:   # the EXACT redirect string shown to the patient.
                   # Uses the AVA-Peter clinic's own contact details (from clinic_card).
                   # Stored and shown verbatim. No grading, no "if mild".
```
`body` = the guideline-sourced symptom description; `verbatim_text` = the clinic-sourced redirect.

## kind: out_of_scope  (refusal stub)
```
  scope: out_of_scope
  confidence: n/a
  body:            # the warm, brief refusal shown to the patient; always routes to clinic
  # source.cite may reference 03_OUT_OF_SCOPE.md as the policy basis; no medical source needed.
```

## Citation writing rule
Each cite lists name + ref + access. If access is `from_report`, the fact came from the research
report because the URL didn't load. If `clinic_card`, it came from the AVA-Peter card (only valid
for verbatim_text and drug-list confirmation). `live` = fetched from the URL this run.
