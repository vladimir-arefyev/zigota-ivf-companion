# Gold-Standard Examples (copy these patterns exactly)

One correct entry per kind. Match the field order, the citation style, the length, and the tone.
These are the target quality bar. Note especially: paraphrased bodies, sources attached, no doses,
ranges attributed, warning redirect uses the clinic's own contact.

```yaml
# ---------- glossary ----------
- id: glossary.follicle
  kind: glossary
  phase: 1
  topic: Follicle
  aliases: [follicle, follicles, фолликул, egg sac]
  body: >
    A small fluid-filled sac in the ovary that contains an egg. During stimulation, several
    follicles are encouraged to grow, and they are tracked by ultrasound.
  scope: in_scope
  confidence: agreed
  source:
    cite:
      - name: ASRM booklet
        ref: https://www.reproductivefacts.org/news-and-publications/fact-sheets-and-infographics/assisted-reproductive-technologies-booklet/
        access: live
  review:
    validated_by: null
    reviewed_on: null
    source_dated: "2015/2018 ed."

# ---------- stage (with attributed range) ----------
- id: stage.trigger
  kind: stage
  phase: 1
  topic: Trigger injection
  aliases: [trigger, trigger shot, hCG shot, укол-триггер, final maturation]
  body: >
    When the follicles are mature, a single "trigger" injection prompts the eggs to finish
    maturing so they can be collected. Its exact timing is set by your clinic and matters a
    great deal — egg retrieval is scheduled a fixed time after it.
  scope: in_scope
  confidence: ranged
  value_range:
    metric: trigger_to_retrieval_hours
    values:
      - source: ASRM booklet
        value: "34-36 h"
      - source: Mayo Clinic
        value: "34-36 h"
      - source: SART
        value: "ovulation ~36-40 h after trigger"
  source:
    cite:
      - name: ASRM booklet
        ref: https://www.reproductivefacts.org/news-and-publications/fact-sheets-and-infographics/assisted-reproductive-technologies-booklet/
        access: live
      - name: Mayo Clinic
        ref: https://www.mayoclinic.org/tests-procedures/in-vitro-fertilization/about/pac-20384716
        access: live
  review:
    validated_by: null
    reviewed_on: null
    source_dated: "Mayo 2023; ASRM 2015/2018 ed."

# ---------- medication (class only, NO dose) ----------
- id: med.progesterone
  kind: medication
  phase: 3
  topic: Progesterone (luteal support)
  aliases: [progesterone, Crinone, Utrogestan, Endometrin, Cyclogest, Крайнон, Утрожестан, прогестерон]
  drug_class: progesterone
  names: [Crinone, Utrogestan, Endometrin, Cyclogest, Gestone]
  purpose: >
    Supports the lining of the uterus after embryo transfer to help an embryo implant.
    Comes in several interchangeable forms (vaginal gel, insert, or injection) chosen by your clinic.
  alternatives_group: luteal_progesterone
  body: >
    A hormone given after transfer to prepare and maintain the uterine lining. Your clinic decides
    the form and how long to continue it, usually up to the pregnancy test and beyond if positive.
  scope: in_scope
  confidence: agreed
  source:
    cite:
      - name: SART
        ref: https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/
        access: from_report
      - name: MedlinePlus
        ref: https://medlineplus.gov/ency/article/007279.htm
        access: live
  review:
    validated_by: null
    reviewed_on: null
    source_dated: "MedlinePlus 2026"

# ---------- instruction (non-med rule) ----------
- id: instr.fasting_before_retrieval
  kind: instruction
  phase: 2
  topic: Fasting before egg retrieval
  aliases: [fasting, empty stomach, no food before retrieval, натощак, nil by mouth]
  body: >
    Because egg retrieval is done under sedation, you are asked not to eat or drink for a set
    period beforehand, as instructed by your clinic. Arriving without fasting can mean the
    procedure has to be postponed.
  unmet_action: escalate_to_clinic
  scope: in_scope
  confidence: agreed
  source:
    cite:
      - name: Mayo Clinic
        ref: https://www.mayoclinic.org/tests-procedures/in-vitro-fertilization/about/pac-20384716
        access: live
  review:
    validated_by: null
    reviewed_on: null
    source_dated: "Mayo 2023"

# ---------- warning (guideline body + clinic verbatim redirect) ----------
- id: warn.ohss.urgent
  kind: warning
  phase: cross
  topic: Possible severe OHSS / urgent symptoms
  aliases: [OHSS, severe bloating, shortness of breath, one-sided pain, shoulder tip pain, гиперстимуляция]
  tier: urgent_care
  match_patterns:
    - difficulty breathing
    - severe or one-sided tummy pain
    - shoulder-tip pain
    - passing very little urine / very thirsty
    - rapid swelling or weight gain
  body: >
    Severe ovarian hyperstimulation syndrome (OHSS) is uncommon but serious. Warning signs
    include difficulty breathing, severe or one-sided abdominal pain, shoulder-tip pain,
    rapid swelling, and passing very little urine.
  verbatim_text: >
    These can be signs of a serious problem. Please contact the clinic's emergency line now:
    AVA-Peter emergency line for ART patients +7 (9XX) XXX-XX-38 (20:00-08:00),
    dispatcher +7 (8XX) XXX-XX-78 (08:00-20:00). If you cannot reach them or feel very unwell,
    seek urgent medical care immediately.
  scope: in_scope
  confidence: agreed
  source:
    cite:
      - name: NHS
        ref: https://www.nhs.uk/tests-and-treatments/ivf/
        access: live
      - name: AVA-Peter card
        ref: SC_PatientCardA6_18sept2019.pdf
        access: clinic_card
  review:
    validated_by: null
    reviewed_on: null
    source_dated: "NHS 2025; card 2019"

# ---------- out_of_scope (refusal stub) ----------
- id: oos.dose
  kind: out_of_scope
  phase: cross
  topic: Dose questions
  aliases: [how much, what dose, how many units, сколько, dosage]
  body: >
    I can't advise on doses — how much of each medication you take is set by your clinic
    specifically for you. Please check with your care team for anything about your dose.
  scope: out_of_scope
  confidence: n/a
  source:
    cite:
      - name: Extraction policy
        ref: 03_OUT_OF_SCOPE.md
        access: from_report
  review:
    validated_by: null
    reviewed_on: null
    source_dated: null
```

Notes the worker must carry into every entry:
- `body` is always paraphrased and short. Only `warning.verbatim_text` is exact/verbatim.
- `medication` entries have NO dose field — do not add one even if the source states a dose.
- ranges are attributed per source; never averaged.
- `review.validated_by` and `reviewed_on` are ALWAYS null at extraction (a human fills them).
- warning `verbatim_text` uses the clinic's own numbers, taken from the card.
