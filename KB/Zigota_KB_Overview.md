# Zigota Knowledge Base — Overview

*Phase 1 (Domain & requirements). This document summarises how the patient-education knowledge
base was built, the quality-assurance it went through, its current state, and what remains before
it ships. Output files are listed at the end.*

---

## Purpose

The knowledge base (KB) is the single vetted source of truth the Zigota companion is allowed to
speak from. It makes the product's core boundary — **AI as an interface to structured data and
vetted content, never a source of medical judgment** — enforceable rather than aspirational.
Every general statement the app makes about IVF must trace to a KB entry with a source attached;
anything not answerable from the KB is refused and routed to the clinic ("citation-or-refuse").

The KB sits behind three product surfaces, matching the protocol model:
- **Grounded Q&A** — answers process questions from vetted content, or refuses.
- **Symptom capture (warnings)** — logs a report and surfaces the matching clinic-redirect verbatim; never grades or reassures.
- **Prescription parsing** — recognises medications by class/purpose, never by dose.

---

## How it was built

The KB was treated as a **curation task, not a writing task**: nothing was authored from a model's
own knowledge — every entry was extracted from a fixed list of vetted sources, paraphrased to
patient-reading level, tagged, and cited.

1. **Evidence-based research first.** An extended research pass, following evidence-based-medicine
   principles (guideline bodies over marketing; ranges with attribution where sources differ),
   produced a source-attributed research report covering the IVF cycle stages, protocol types,
   medication classes, standard instructions, warning signs, and a glossary. Sources:
   ASRM/reproductivefacts.org, SART, NHS, HFEA, Mayo Clinic, MedlinePlus (with ESHRE, NICE, WHO
   cross-checked).

2. **A constrained extraction packet.** Because the extraction had to follow instructions
   literally rather than exercise judgment, the job was specified as a self-contained packet for a
   worker model: master instructions, an approved-sources register with precedence and fallback
   rules, an out-of-scope/limitations spec, the output schema, one gold-standard example per kind,
   and a per-entry self-check. A hard schema rule — no dose field exists on medication entries —
   enforces the boundary by construction.

3. **Extraction run.** The worker produced 60 YAML entries across the full three-phase arc and all
   protocol types, grouped by kind, with `review.validated_by` left null pending human validation.

---

## Quality assurance

The KB went through a **two-model adversarial check plus a verification arc** that is itself part
of the credibility story:

- **Independent auditor (a different model family).** A separate model verified the KB against the
  same rules — checking that every cited fact is actually in its source, that no doses leaked into
  medication entries, that warnings never grade severity, that ranges were attributed not averaged,
  and that all out-of-scope refusal stubs were present.
- **The trace that mattered.** A first audit returned clean, but a summary-only report proved
  nothing about whether the checker had done the checking. Requiring a per-entry verification trace
  exposed that both the extractor and the auditor had fallen back to the same research report rather
  than the live sources — so "clean" meant *internally consistent*, not *re-verified against
  primary sources*.
- **Live-fetch re-run.** Pushing the auditor to fetch the live URLs closed the gap for the
  weight-bearing entries: the warning symptom lists (NHS, Mayo, MedlinePlus) and the
  trigger-to-retrieval window (ASRM + Mayo, independently) were confirmed against primary sources.
  The residual: SART's live page kept returning an index, so SART-only facts remain report-verified.

**Lesson worth keeping:** a checker that shares its evidence base with the thing it checks is not an
independent check. The audit is now trustworthy for the entries that carry clinical weight, and
honest about the ones it couldn't reach live.

---

## Current state

- **60 entries**: 8 out-of-scope refusals · 18 glossary · 16 cycle stages · 7 medications (class-level) · 6 instructions · 5 warnings.
- **Confidence mix**: 35 `agreed` · 12 `single_source` · 5 `ranged` (attributed, never averaged) · 8 `n/a` (refusals).
- **Audit**: 0 blockers, 0 majors, 0 minors; weight-bearing entries live-verified.
- **Human validation**: not yet done — `validated_by` is null across all entries. This is the gate that earns the word "vetted."

Two review flags travel with the affected entries:
- **verify-clinic-number (5 warning entries).** Redirects use the AVA-Peter patient card dated 2019.
  The phone numbers must be confirmed current with the clinic — this is the highest-harm single fact
  in the KB and is outside any model's reach.
- **SART-report-only (7 entries).** Luteal-phase and GnRH glossary terms, the flare protocol,
  agonist-trigger, estrogen, and partner-abstinence rely on SART via the research report
  (report-verified, not live-verified).

---

## Next steps

1. **Confirm the clinic contact numbers** on the 5 warning entries against the current AVA-Peter
   card / clinic (not the 2019 file). Highest priority; real-world check.
2. **Domain-literate clinical validation.** Send the interactive review table to a reviewer with IVF
   knowledge; collect Approve / Needs-fix / Reject verdicts and notes. Pay special attention to
   flagged entries.
3. **Apply verdicts to `kb.yaml`**, then regenerate the human-readable views (they are generated from
   the YAML and stay in sync). Populate `review.validated_by` / `reviewed_on` on approved entries.
4. **Optionally re-source the SART-only entries** to a live page, or tag them report-only in the
   shipped KB.
5. **Hand off to the next Phase 1 output** — job stories + testable acceptance criteria — which the
   protocol model and this KB were designed to make enforceable.

Passing the audit is necessary, not sufficient: the KB ships only after step 2.

---

## Output files

Source of truth:
- **`kb.yaml`** — the 60-entry knowledge base (canonical; all views are generated from this).

Extraction packet (the job spec that produced the KB):
- **`00_README.md`**, **`01_INSTRUCTIONS.md`**, **`02_SOURCES.md`**, **`03_OUT_OF_SCOPE.md`**,
  **`04_KB_STRUCTURE.md`**, **`05_EXAMPLES.md`**, **`06_SELF_CHECK.md`**, **`07_AUDITOR.md`**

Evidence base:
- **`Project_Zigota__IVF_ART_Patient-Education_Knowledge_Base_and_Three-Phase_Protocol_Model.md`** — the source-attributed research report / fallback source.

Quality assurance:
- **`07_AUDIT_REPORT.md`** — the auditor's report, including the live-fetch verification trace.

Human-readable views (generated from `kb.yaml`):
- **`Zigota_KB_review.html`** — interactive clinical review table (Approve / Needs-fix / Reject + notes + export). For the validation pass.
- **`Zigota_KB_readable.md`** — browsable KB rendering for GitHub / portfolio.

Related design artifact:
- **`Zigota_Protocol_Model_v2.md`** — the evidence-grounded protocol model the KB feeds.
