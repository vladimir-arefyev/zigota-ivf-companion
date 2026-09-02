# Out-of-Scope & Limitations

Two jobs here: (A) the categories you must NEVER emit as guidance, and (B) the explicit refusal
stubs you MUST emit as `out_of_scope` entries so the app can refuse cleanly instead of improvising.

## A. Never emit as guidance (recognize and refuse)
These are individualized / clinician-determined. If a fact falls in any category below, it is
out of scope — do not state it as guidance, do not compute it, do not infer it:
- Drug doses, units, quantities, titration, or personalized schedules.
- Which protocol or drug a specific patient should use, or any protocol RECOMMENDATION.
- Adjusting, stopping, or changing any medication.
- Interpreting a specific patient's symptoms, ultrasound findings, or lab/hormone values.
- Whether a symptom is "normal," "fine," "dangerous," or how severe it is (grading).
- Reassurance, diagnosis, prognosis, or success-chance estimates for an individual.
- Predicting monitoring dates, trigger time, retrieval/transfer dates, or cycle length for a person.
- Anything not traceable to an approved source.

Note: comparative clinical statistics (e.g. OHSS-risk differences between protocols) are design
BACKGROUND only. Do NOT emit them as patient-facing KB content, and never use them to imply one
protocol is better for a user.

## B. Refusal stubs to emit (write these FIRST, as kind: out_of_scope)
Emit one entry per item below, using the out_of_scope pattern in 05_EXAMPLES.md. The `body` is the
refusal the app will show. Keep refusals warm, brief, and always route to the clinic.

- id: oos.dose            trigger: any question about how much / what dose / how many units
- id: oos.protocol_choice trigger: "which protocol should I be on", "is X protocol right for me"
- id: oos.med_change      trigger: "should I stop/change/skip a medication", "can I adjust my dose"
- id: oos.symptom_normal  trigger: "is this [symptom] normal / okay / dangerous", severity grading
- id: oos.lab_interpret   trigger: "what do my numbers / scan / estradiol mean for me"
- id: oos.success_odds    trigger: "what are my chances", personalized prognosis
- id: oos.date_predict    trigger: "when will my trigger/retrieval/transfer be", predicting dates
- id: oos.diagnosis       trigger: "do I have [condition]", "is this OHSS"

For every out_of_scope stub, set `scope: out_of_scope` and route to the clinic. If a symptom-grading
question could also indicate a red flag, the refusal should still point the patient to contact their
clinic (the app's warning layer handles genuine escalation separately — you only write the refusal).

## C. The fallback (context, not an entry)
If a patient question matches no in-scope entry, the app returns a standard clinic-redirect. You do
not author that here; just make sure every out_of_scope category above has a stub so common
boundary-crossing questions hit a deliberate refusal rather than the generic fallback.
