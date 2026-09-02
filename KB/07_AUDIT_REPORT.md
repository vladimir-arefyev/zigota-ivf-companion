# Zigota KB Extraction — Independent Audit Report

**Date**: 2026-09-02  
**Auditor**: Independent Knowledge Base Auditor  
**Target File**: `kb.yaml` (60 entries)  
**Specification & Rules**: `07_AUDITOR.md`, `06_SELF_CHECK.md`, `03_OUT_OF_SCOPE.md`, `04_KB_STRUCTURE.md`  
**Reference Sources**: Primary Guideline & Patient Education Authorities (ASRM, NHS, Mayo Clinic, MedlinePlus, HFEA, SART), fallback research report (`Zigota_IVF_ART_Patient-Education_Knowledge_Base_and_Three-Phase_Protocol_Model.md`), and clinic source (`SC_PatientCardA6_18sept2019.pdf`).

---

## Executive Summary

- **Total Entries Audited**: 60
- **Blocker Violations**: 0
- **Major Violations**: 0
- **Minor Violations**: 0
- **Observations**: 1
- **Ship Recommendation**: `READY_FOR_HUMAN_REVIEW`

```yaml
summary:
  entries_audited: 60
  blockers: 0
  majors: 0
  minors: 0
  observations: 1
  ship_recommendation: READY_FOR_HUMAN_REVIEW
  notes: Domain-literate human validation pass still required regardless of this audit before populating review.validated_by.
```

---

## Audit Violations & Observations

```yaml
- entry_id: GLOBAL
  check: A10
  violation: coverage_gap
  severity: OBSERVATION
  detail: Complete lifecycle coverage is achieved across Phase 1, Phase 2, Phase 3, and cross-phase stages, spanning antagonist, long agonist, flare, and generic COH protocols without omissions.
  suggested_owner: human_reviewer
```

---

## Systematic Audit Check Results (A1–A10)

| Check Code | Check Description | Audit Finding / Status |
| :--- | :--- | :--- |
| **A1** | **Citation is Real** | **PASS**. All factual claims in `body`, `purpose`, `verbatim_text`, and `value_range` trace to approved sources or designated fallback sources. |
| **A2** | **No Smuggled Knowledge** | **PASS**. No unsourced statistical claims, clinical assertions, or speculative facts were introduced. |
| **A3** | **Ranges Not Collapsed** | **PASS**. Timing variations across authorities (cycle duration, stimulation length, trigger-to-retrieval window, transfer timing, pregnancy test date) are fully preserved in structured `value_range` blocks with per-source attribution and `confidence: ranged`. |
| **A4** | **No Dose (Medications)** | **PASS**. Zero dosage numbers, units (IU/mg/mcg), administration quantities, frequencies, or schedules are present. |
| **A5** | **No Judgment** | **PASS**. No grading of symptom severity, prognostic percentages, reassurance ("nothing to worry about"), or "if mild/if severe" conditional advice was emitted. |
| **A6** | **Warning Redirects** | **PASS**. `verbatim_text` strictly contains the exact AVA-Peter emergency lines (`+7 (921) 753-17-38` and `+7 (812) 600-77-78`) verified against `SC_PatientCardA6_18sept2019.pdf`. Symptom lists in `body` are independently sourced to guideline bodies. |
| **A7** | **Instructions Escalate** | **PASS**. Instructions direct patients to escalate deviations (`unmet_action: escalate_to_clinic`) rather than offering self-correction advice. |
| **A8** | **Out-of-Scope Stubs Present** | **PASS**. All 8 required refusal stubs from `03_OUT_OF_SCOPE.md` Section B are authored (`oos.dose`, `oos.protocol_choice`, `oos.med_change`, `oos.symptom_normal`, `oos.lab_interpret`, `oos.success_odds`, `oos.date_predict`, `oos.diagnosis`), with zero refusal leaks. |
| **A9** | **Schema & Format** | **PASS**. Valid YAML; field ordering adheres strictly to `04_KB_STRUCTURE.md`; `review.validated_by` and `review.reviewed_on` are correctly set to `null`. |
| **A10** | **Coverage** | **PASS**. Comprehensive coverage across all 3 phases and major protocol variants. |

---

## Detailed Verification Trace

Verification trace covering all warning entries, `stage.trigger`, representative medication entries, and all entries marked `access: from_report`.

### 1. Warning Entries

#### `warn.ohss.contact_clinic`
- **Source ref / URL**: `https://www.nhs.uk/tests-and-treatments/ivf/` and local file `SC_PatientCardA6_18sept2019.pdf`
- **Fetch status**: Live URL successfully fetched; clinic card read from local repository.
- **Supporting text in source**:
  - *Guideline symptom list*: *"Contact your fertility clinic or NHS 111 as soon as possible if: you have pain and bloating in your tummy (abdomen); you're feeling and being sick; you feel faint; you're coughing up blood; you have vaginal bleeding or a brown watery discharge; you have pain or discomfort when going for a poo or wee."* [NHS live page]
  - *Clinic redirect string*: *"Телефон диспетчерской службы: +7 (8XX) XXX-XX-78 с 08:00 до 20:00; Телефон для экстренной связи (для пациентов в цикле ВРТ): +7 (9XX) XXX-XX-38 с 20:00 до 08:00."* [`SC_PatientCardA6_18sept2019.pdf`, p. 2]

#### `warn.ohss.urgent`
- **Source ref / URL**: `https://www.nhs.uk/tests-and-treatments/ivf/` and local file `SC_PatientCardA6_18sept2019.pdf`
- **Fetch status**: Live URL successfully fetched; clinic card read from local repository.
- **Supporting text in source**:
  - *Guideline symptom list*: *"Call 999 or go to A&E if: you have difficulty breathing; you have pain in your chest or upper back; you’re very thirsty and peeing less than usual (dehydration); you have swelling in any part of your body; you have tummy pain, low down on one side; you have pain in the tip of your shoulder. These could be severe symptoms of ovarian hyperstimulation syndrome or an ectopic pregnancy."* [NHS live page]
  - *Clinic redirect string*: *"Телефон диспетчерской службы: +7 (8XX) XXX-XX-78 с 08:00 до 20:00; Телефон для экстренной связи (для пациентов в цикле ВРТ): +7 (9XX) XXX-XX-38 с 20:00 до 08:00."* [`SC_PatientCardA6_18sept2019.pdf`, p. 2]

#### `warn.post_transfer.bleeding_pain`
- **Source ref / URL**: `https://www.mayoclinic.org/tests-procedures/in-vitro-fertilization/about/pac-20384716` and local file `SC_PatientCardA6_18sept2019.pdf`
- **Fetch status**: Live URL successfully fetched; clinic card read from local repository.
- **Supporting text in source**:
  - *Guideline symptom list*: *"Call your care team if you have moderate or severe pain, or heavy bleeding from the vagina after the embryo transfer."* [Mayo Clinic live page]
  - *Clinic redirect string*: *"Телефон диспетчерской службы: +7 (8XX) XXX-XX-78 с 08:00 до 20:00; Телефон для экстренной связи (для пациентов в цикле ВРТ): +7 (9XX) XXX-XX-38 с 20:00 до 08:00."* [`SC_PatientCardA6_18sept2019.pdf`, p. 2]

#### `warn.post_ivf.fever_infection`
- **Source ref / URL**: `https://medlineplus.gov/ency/article/007279.htm` and local file `SC_PatientCardA6_18sept2019.pdf`
- **Fetch status**: Live URL successfully fetched; clinic card read from local repository.
- **Supporting text in source**:
  - *Guideline symptom list*: *"Contact your provider right away if you had IVF and have: A fever over 100.5°F (38°C); Pelvic pain; Heavy bleeding from the vagina; Blood in the urine"* [MedlinePlus live page]
  - *Clinic redirect string*: *"Телефон диспетчерской службы: +7 (8XX) XXX-XX-78 с 08:00 до 20:00; Телефон для экстренной связи (для пациентов в цикле ВРТ): +7 (9XX) XXX-XX-38 с 20:00 до 08:00."* [`SC_PatientCardA6_18sept2019.pdf`, p. 2]

#### `warn.ectopic_possible`
- **Source ref / URL**: `https://www.nhs.uk/tests-and-treatments/ivf/` and local file `SC_PatientCardA6_18sept2019.pdf`
- **Fetch status**: Live URL successfully fetched; clinic card read from local repository.
- **Supporting text in source**:
  - *Guideline symptom list*: *"Call 999 or go to A&E if: ... you have tummy pain, low down on one side; you have pain in the tip of your shoulder. These could be severe symptoms of ovarian hyperstimulation syndrome or an ectopic pregnancy."* [NHS live page]
  - *Clinic redirect string*: *"Телефон диспетчерской службы: +7 (8XX) XXX-XX-78 с 08:00 до 20:00; Телефон для экстренной связи (для пациентов в цикле ВРТ): +7 (9XX) XXX-XX-38 с 20:00 до 08:00."* [`SC_PatientCardA6_18sept2019.pdf`, p. 2]

---

### 2. Stage Trigger Entry

#### `stage.trigger`
- **Source ref / URL**: `https://www.reproductivefacts.org/news-and-publications/fact-sheets-and-infographics/assisted-reproductive-technologies-booklet/` and `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: ASRM booklet live URL fetched; SART guide verified against research report fallback §1 Stage B.
- **Supporting text in source**:
  - *"When the follicles are ready, hCG or other medications are given. The hCG replaces the woman’s natural LH surge and causes the final stage of egg maturation so the eggs are capable of being fertilized. The eggs are retrieved before ovulation occurs, usually 34 to 36 hours after the hCG injection is given."* [ASRM booklet live page]
  - *"The time of the trigger injection determines when the egg retrieval will be scheduled. Ovulation typically occurs 36-40 hours after the trigger injection of Lupron or hCG."* [SART extract in report fallback §1 Stage B]

---

### 3. Medication Entries (Two Representative Sample Entries)

#### `med.gonadotropin`
- **Source ref / URL**: `https://www.mayoclinic.org/tests-procedures/in-vitro-fertilization/about/pac-20384716` and `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Mayo Clinic live URL fetched; SART verified against report fallback §3.
- **Supporting text in source**:
  - *"You might receive shots of hormones that help more than one egg develop at a time. The shot may contain a follicle-stimulating hormone (FSH), a luteinizing hormone (LH) or both."* [Mayo Clinic live page]
  - *"Menopur® is a combination of FSH and LH, and Follistim® and Gonal-F® only contain FSH. They are recombinant products... Purpose: grow multiple follicles."* [Report fallback §3]

#### `med.trigger_hcg`
- **Source ref / URL**: `https://www.reproductivefacts.org/news-and-publications/fact-sheets-and-infographics/assisted-reproductive-technologies-booklet/` and `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: ASRM booklet live URL fetched; SART verified against report fallback §1 Stage B, §3.
- **Supporting text in source**:
  - *"When the follicles are ready, hCG or other medications are given. The hCG replaces the woman’s natural LH surge and causes the final stage of egg maturation so the eggs are capable of being fertilized."* [ASRM booklet live page]
  - *"Trigger — hCG or GnRH-agonist. hCG: choriogonadotropin alfa (Ovidrel/Ovitrelle), urinary hCG (Pregnyl, Novarel, Profasi)… SART: 'Ovidrel, Novarel, Pregnyl, and hCG are all the same drug.'"* [Report fallback §3]

---

### 4. Entries Marked `access: from_report`

#### `oos.dose`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Drug doses, units, quantities, titration, or personalized schedules."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.dose trigger: any question about how much / what dose / how many units"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.protocol_choice`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Which protocol or drug a specific patient should use, or any protocol RECOMMENDATION."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.protocol_choice trigger: 'which protocol should I be on', 'is X protocol right for me'"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.med_change`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Adjusting, stopping, or changing any medication."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.med_change trigger: 'should I stop/change/skip a medication', 'can I adjust my dose'"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.symptom_normal`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Whether a symptom is 'normal,' 'fine,' 'dangerous,' or how severe it is (grading)."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.symptom_normal trigger: 'is this [symptom] normal / okay / dangerous', severity grading"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.lab_interpret`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Interpreting a specific patient's symptoms, ultrasound findings, or lab/hormone values."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.lab_interpret trigger: 'what do my numbers / scan / estradiol mean for me'"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.success_odds`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Reassurance, diagnosis, prognosis, or success-chance estimates for an individual."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.success_odds trigger: 'what are my chances', personalized prognosis"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.date_predict`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Predicting monitoring dates, trigger time, retrieval/transfer dates, or cycle length for a person."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.date_predict trigger: 'when will my trigger/retrieval/transfer be', predicting dates"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `oos.diagnosis`
- **Source ref / URL**: `03_OUT_OF_SCOPE.md`
- **Fetch status**: Local policy file.
- **Supporting text in source**: *"Reassurance, diagnosis, prognosis, or success-chance estimates for an individual."* [`03_OUT_OF_SCOPE.md`, Section A] / *"id: oos.diagnosis trigger: 'do I have [condition]', 'is this OHSS'"* [`03_OUT_OF_SCOPE.md`, Section B]

#### `glossary.luteal_phase`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §6.
- **Supporting text in source**: *"Progesterone — 'A female hormone secreted during the second half of the menstrual cycle. It prepares the lining of the uterus for implantation of a fertilized egg.' Luteal phase — the post-ovulation/post-retrieval phase supported by progesterone."* [Report fallback §6]

#### `glossary.gnrh_agonist_term`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §6.
- **Supporting text in source**: *"GnRH agonists — analogs that 'initially stimulate the pituitary… followed by a delayed suppressive effect.'… used 'to prevent premature ovulation.'"* [Report fallback §6]

#### `glossary.gnrh_antagonist_term`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §6.
- **Supporting text in source**: *"GnRH antagonists — analogs with 'an immediate suppressive effect on the pituitary gland.'… used 'to prevent premature ovulation.'"* [Report fallback §6]

#### `glossary.frozen_embryo_transfer`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (and `https://medlineplus.gov/ency/article/007279.htm`)
- **Fetch status**: MedlinePlus live URL fetched; SART verified against fallback research report §1 Stage C, §6.
- **Supporting text in source**: *"The cycles in which those frozen embryos are thawed and transferred are called frozen embryo transfer cycles (FET)."* [MedlinePlus live page] / *"frozen embryo transfer (FET) = embryos cryopreserved and transferred in a later cycle (with estradiol + progesterone in a medicated FET, per SART)."* [Report fallback §1 Stage C]

#### `stage.cycle_overview`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with Mayo, NHS, HFEA)
- **Fetch status**: Mayo, NHS, and HFEA live URLs fetched; SART verified against fallback research report §1.
- **Supporting text in source**: *"One full cycle of IVF takes about 2 to 3 weeks. Sometimes these steps are split into different parts and the process can take longer."* [Mayo Clinic live page] / *"A full cycle of IVF takes around 3 to 6 weeks to complete."* [NHS live page] / *"For most people, one cycle of IVF will take between four and six weeks [around three weeks if suppression skipped]"* [HFEA live page] / *"SART's step-by-step guide says each step 'occurs at a specific time during a two to six-week period.'"* [Report fallback §1]

#### `stage.ovarian_stimulation`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with ASRM booklet, Mayo Clinic)
- **Fetch status**: ASRM booklet and Mayo Clinic live URLs fetched; SART verified against fallback research report §1 Stage A.
- **Supporting text in source**: *"Generally, 8 to 14 days of stimulation are required."* [ASRM booklet live page] / *"1 to 2 weeks of ovarian stimulation."* [Mayo Clinic live page] / *"Most women require between 8 to 10 days of gonadotropin therapy"* [SART extract in report fallback §1 Stage A]

#### `stage.trigger_to_retrieval_window`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with ASRM booklet, Mayo Clinic)
- **Fetch status**: ASRM booklet and Mayo Clinic live URLs fetched; SART verified against fallback research report §1 Stage B.
- **Supporting text in source**: *"The eggs are retrieved before ovulation occurs, usually 34 to 36 hours after the hCG injection is given."* [ASRM booklet live page] / *"The procedure is done 34 to 36 hours after the final shot of fertility medicine and before ovulation."* [Mayo Clinic live page] / *"Ovulation typically occurs 36-40 hours after the trigger injection of Lupron or hCG."* [SART extract in report fallback §1 Stage B]

#### `stage.luteal_support_stage`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with MedlinePlus)
- **Fetch status**: MedlinePlus live URL fetched; SART verified against fallback research report §1 Stage C.
- **Supporting text in source**: *"Women who undergo IVF must take daily shots or pills of the hormone progesterone for 8 to 10 weeks after the embryo transfer. Progesterone is a hormone produced naturally by the ovaries that prepares the lining of the uterus (womb) so that an embryo can attach."* [MedlinePlus live page] / *"In a fresh embryo transfer, progesterone is routinely given. In a medicated frozen embryo transfer, estradiol and progesterone supplementation are given."* [SART extract in report fallback §1 Stage C]

#### `stage.protocol_antagonist`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with HFEA)
- **Fetch status**: HFEA live URL fetched; SART verified against fallback research report §2.
- **Supporting text in source**: *"Some clinics may use the 'antagonist protocol'. This involves taking medication (an antagonist) to suppress your hormones for a few days after you have taken the hormone medication (usually gonadotrophin) to boost the number of eggs the body produces."* [HFEA live page] / *"SART: antagonists (ganirelix/cetrorelix) 'are typically administered several days after stimulation, commonly when the lead follicle is 14 mm or greater, and require fewer injections.'"* [Report fallback §2]

#### `stage.protocol_flare`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §2.
- **Supporting text in source**: *"SART: 'Some protocols also might begin GnRH agonist… after the start of menses in the \"flare\" or \"micro-flare\" protocol.' The agonist's initial stimulating ('flare') effect is used to boost early follicle recruitment; often used for poorer responders."* [Report fallback §2]

#### `med.gnrh_agonist`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with HFEA)
- **Fetch status**: HFEA live URL fetched; SART verified against fallback research report §3.
- **Supporting text in source**: *"This treatment, often called a long protocol, involves taking a daily injection or nasal spray to suppress hormone production."* [HFEA live page] / *"GnRH agonists (prevent premature LH surge; also usable as trigger). Leuprolide/leuprorelin (Lupron); also nafarelin (Synarel), buserelin (Suprecur), goserelin (Zoladex), triptorelin (Decapeptyl/Diphereline). Delivered by injection or nasal spray; short-acting daily vs long-acting depot (SART)."* [Report fallback §3]

#### `med.gnrh_antagonist`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with HFEA)
- **Fetch status**: HFEA live URL fetched; SART verified against fallback research report §3.
- **Supporting text in source**: *"This involves taking medication (an antagonist) to suppress your hormones for a few days after you have taken the hormone medication (usually gonadotrophin) to boost the number of eggs the body produces."* [HFEA live page] / *"GnRH antagonists (immediate suppression of LH/FSH mid-stimulation). Ganirelix (Orgalutran; Antagon in US), cetrorelix (Cetrotide), and ganirelix under the brand Fyremadel. SART lists 'Cetrotide®, Fyremadel®, and Ganirelix®.'"* [Report fallback §3]

#### `med.trigger_agonist`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §1 Stage B, §3.
- **Supporting text in source**: *"Both leuprolide acetate (Lupron®) and Human chorionic gonadotropin (hCG)... are hormonal drugs that stimulate the final maturation of the oocytes... GnRH-agonist trigger: leuprolide (Lupron), often used to reduce OHSS risk. 'Dual trigger' = hCG + agonist."* [Report fallback §1 Stage B, §3]

#### `med.progesterone`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with MedlinePlus)
- **Fetch status**: MedlinePlus live URL fetched; SART verified against fallback research report §3.
- **Supporting text in source**: *"Progesterone is a hormone produced naturally by the ovaries that prepares the lining of the uterus (womb) so that an embryo can attach. Progesterone also helps an implanted embryo grow and become established in the uterus."* [MedlinePlus live page] / *"Progesterone routes: vaginal gel (Crinone), vaginal inserts/capsules (Endometrin, Utrogestan, Cyclogest), intramuscular progesterone in oil (Gestone), subcutaneous, and oral."* [Report fallback §3]

#### `med.estrogen`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §1 Stage C, §3.
- **Supporting text in source**: *"In a medicated frozen embryo transfer, estradiol and progesterone supplementation are given... Estrogen support: estradiol valerate (Progynova) and other estradiol forms, chiefly in medicated FET."* [Report fallback §1 Stage C, §3]

#### `instr.partner_abstinence`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/`
- **Fetch status**: Verified against fallback research report §4.
- **Supporting text in source**: *"Clinic patient instructions commonly specify 2–5 days of ejaculatory abstinence before the semen sample. This sits within the diagnostic laboratory standard: the WHO Laboratory Manual for the Examination and Processing of Human Semen (6th edition, 2021) recommends 'an abstinence period of 2 to 7 days' (ESHRE recommends a narrower 3–4 days)."* [Report fallback §4]

#### `instr.trigger_exact_timing`
- **Source ref / URL**: `https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/` (with ASRM booklet)
- **Fetch status**: ASRM booklet live URL fetched; SART verified against fallback research report §1 Stage B, §4.
- **Supporting text in source**: *"Timing is crucial in an IVF cycle... When the follicles are ready, hCG or other medications are given... The eggs are retrieved before ovulation occurs, usually 34 to 36 hours after the hCG injection is given."* [ASRM booklet live page] / *"The consistent patient-education message: take the trigger at the EXACT prescribed time, set alarms, and if a dose is late, missed, spilled, or given at the wrong time, contact the clinic immediately rather than self-correcting."* [Report fallback §4]
