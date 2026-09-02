# Zigota — IVF/ART Patient-Education Knowledge Base (readable)

*Human-readable rendering of the extracted KB (`kb.yaml`). Generated from the YAML source of truth — do not edit by hand; regenerate instead. Every entry is drawn from vetted patient-education sources and is pending domain-literate clinical validation (`validated_by` is null until then).*

**60 entries** · 8 out-of-scope refusals · 18 glossary · 16 cycle stages · 7 medications (class-level) · 6 patient instructions · 5 warning signs

## How to read this

- **Scope boundary:** the KB is an interface to vetted content, never medical judgment. Out-of-scope refusals are first-class entries — the app refuses these deliberately.
- **Confidence:** `agreed` (sources concur) · `ranged` (sources differ — full range kept, each figure attributed, never averaged) · `single_source` · `n/a` (refusals).
- **Two review flags** travel with the affected entries: **verify-clinic-number** (warning redirects use the 2019 AVA-Peter card — confirm still current) and **SART-report-only** (report-verified, not live-verified).

## Out-of-scope refusals

### Dose questions

`oos.dose` · Cross-phase · confidence: n/a

I can't advise on doses — how much of each medication you take is set by your clinic specifically for you. Please check with your care team for anything about your dose.

- **Source:** Extraction policy (from_report)

### Which protocol is right for me

`oos.protocol_choice` · Cross-phase · confidence: n/a

I can't tell you which stimulation protocol or medication plan is right for you — that choice is made by your clinic based on your individual history and test results. Please discuss protocol options with your care team.

- **Source:** Extraction policy (from_report)

### Changing or skipping a medication

`oos.med_change` · Cross-phase · confidence: n/a

I can't advise on stopping, skipping, or changing any medication or its timing. Please contact your clinic right away if you're considering or have already made a change — they can tell you what to do next.

- **Source:** Extraction policy (from_report)

### Is this symptom normal

`oos.symptom_normal` · Cross-phase · confidence: n/a

I can't judge whether a symptom you're having is normal, mild, or serious for you — that needs a clinician's assessment. If you're worried, please contact your clinic; if you feel very unwell, seek urgent care.

- **Source:** Extraction policy (from_report)

### What my numbers or scan mean

`oos.lab_interpret` · Cross-phase · confidence: n/a

I can't interpret your personal lab results, scan findings, or hormone levels — only your clinical team has the full picture needed to explain what your numbers mean for you. Please ask your clinic to walk you through your results.

- **Source:** Extraction policy (from_report)

### My personal chances of success

`oos.success_odds` · Cross-phase · confidence: n/a

I can't estimate your personal chances of a successful cycle — that depends on many individual factors your clinic weighs together. Please ask your care team for a personalized assessment.

- **Source:** Extraction policy (from_report)

### Predicting my trigger, retrieval, or transfer date

`oos.date_predict` · Cross-phase · confidence: n/a

I can't predict or calculate your trigger, retrieval, or transfer date — these are set by your clinic based on your monitoring results. Please check with your care team for your exact schedule.

- **Source:** Extraction policy (from_report)

### Do I have this condition

`oos.diagnosis` · Cross-phase · confidence: n/a

I can't diagnose a condition from what you describe — only your clinic can examine you and confirm what's happening. If you have concerning symptoms, please contact them, or seek urgent care if you feel very unwell.

- **Source:** Extraction policy (from_report)

## Glossary

### Follicle

`glossary.follicle` · Phase 1 · Stimulation · confidence: agreed

A fluid-filled structure in the ovary that contains an egg and the cells that produce hormones. During stimulation, several follicles are encouraged to grow and are tracked by ultrasound.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Oocyte / egg

`glossary.oocyte` · Phase 1 · Stimulation · confidence: agreed

The female sex cell (ovum) produced by the ovary. In an IVF cycle, mature eggs are collected at egg retrieval so they can be fertilized in the lab.

- **Source:** ASRM booklet (live)

### Estradiol

`glossary.estradiol` · Phase 1 · Stimulation · confidence: agreed

The main estrogen hormone made by the growing follicles. Its level is checked by blood test during stimulation and normally rises as follicles develop.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Egg retrieval / oocyte aspiration

`glossary.egg_retrieval` · Phase 2 · Trigger & retrieval · confidence: agreed

The procedure in which eggs are collected by guiding a thin needle through the vagina into each ovarian follicle and drawing out the fluid and egg with gentle suction, using ultrasound to guide the needle.

- **Source:** ASRM booklet (live); MedlinePlus (live)

### ICSI (intracytoplasmic sperm injection)

`glossary.icsi` · Phase 2 · Trigger & retrieval · confidence: agreed

A lab technique in which a single sperm is injected directly into a mature egg, used mainly when fertilization by mixing sperm and egg together is less likely to succeed.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Embryo

`glossary.embryo` · Phase 2 · Trigger & retrieval · confidence: agreed

A fertilized egg that has begun dividing into cells. Embryos are grown in the lab (embryo culture) for a few days before transfer or freezing.

- **Source:** ASRM booklet (live)

### Blastocyst

`glossary.blastocyst` · Phase 2 · Trigger & retrieval · confidence: agreed

An embryo that has developed a fluid-filled cavity, usually by day 5 after egg retrieval, when the cells begin separating into the parts that will form the placenta and the embryo.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Zona pellucida / assisted hatching

`glossary.zona_pellucida` · Phase 2 · Trigger & retrieval · confidence: agreed

The zona pellucida is the egg's outer shell, which a sperm must penetrate and which the embryo must later break out of ("hatch") to implant. Assisted hatching is a lab technique that opens this shell slightly before transfer to help that process.

- **Source:** ASRM booklet (live)

### hCG (human chorionic gonadotropin)

`glossary.hcg` · Cross-phase · confidence: agreed

A hormone that, in medication form, triggers final egg maturation before retrieval. hCG is also the hormone produced by an early pregnancy and is what pregnancy blood tests measure.

- **Source:** ASRM booklet (live)

### Beta-hCG (pregnancy blood test)

`glossary.beta_hcg` · Phase 3 · Transfer & luteal · confidence: agreed

A blood test that measures the level of hCG hormone to confirm and monitor an early pregnancy after embryo transfer. It's done on a specific day set by the clinic.

- **Source:** MedlinePlus (live); NHS (live)

### Progesterone

`glossary.progesterone_term` · Phase 3 · Transfer & luteal · confidence: agreed

A hormone that prepares and maintains the lining of the uterus so a fertilized egg can attach and grow. It's supplemented after embryo transfer as part of luteal support.

- **Source:** MedlinePlus (live)

### Luteal phase

`glossary.luteal_phase` · Phase 3 · Transfer & luteal · confidence: agreed

The period after egg retrieval (and, in a natural cycle, after ovulation) during which the uterine lining is prepared for a possible pregnancy, typically supported with progesterone medication in IVF.

- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

### GnRH agonist

`glossary.gnrh_agonist_term` · Phase 1 · Stimulation · confidence: agreed

A medication that first briefly stimulates, then suppresses, the pituitary gland's hormone signals, used to prevent the ovaries from releasing eggs too early during stimulation.

- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

### GnRH antagonist

`glossary.gnrh_antagonist_term` · Phase 1 · Stimulation · confidence: agreed

A medication that immediately blocks the pituitary gland's hormone signal to prevent premature ovulation. It's usually started partway through stimulation.

- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

### Gonadotropins (FSH/LH)

`glossary.gonadotropins_term` · Phase 1 · Stimulation · confidence: agreed

Injectable hormones (follicle-stimulating hormone and/or luteinizing hormone) that stimulate the ovaries to grow multiple eggs during a treatment cycle.

- **Source:** Mayo Clinic (live); ASRM booklet (live)

### OHSS (ovarian hyperstimulation syndrome)

`glossary.ohss_term` · Cross-phase · confidence: agreed

A condition that can occur after ovarian stimulation, in which the ovaries become swollen and painful and fluid can build up in the body. Most cases are mild; it's rarely severe.

- **Source:** Mayo Clinic (live); NHS (live)

### Frozen embryo transfer (FET)

`glossary.frozen_embryo_transfer` · Phase 3 · Transfer & luteal · confidence: agreed

A cycle in which a previously frozen embryo is thawed and transferred to the uterus, separate from the original stimulation and retrieval cycle.

- **Source:** MedlinePlus (live); SART (from_report)

### Ovarian reserve

`glossary.ovarian_reserve` · Phase 1 · Stimulation · confidence: agreed

A general measure of a woman's remaining egg supply, assessed before a cycle using blood tests (such as AMH) and an ultrasound count of small follicles (antral follicle count).

- **Source:** ASRM booklet (live)

## Cycle stages

### Overall IVF cycle length

`stage.cycle_overview` · Cross-phase · confidence: ranged

A full IVF cycle covers ovarian stimulation, egg retrieval, fertilization, embryo transfer, and luteal support. Sources give different overall lengths depending on whether a suppression stage is included.

- **full_cycle_duration:** Mayo Clinic: about 2-3 weeks; NHS: around 3-6 weeks; HFEA: 4-6 weeks (about 3 weeks if the suppression stage is skipped); SART: 2-6 week period
- **Source:** Mayo Clinic (live); NHS (live); HFEA (live); SART (from_report)

### Ovarian stimulation

`stage.ovarian_stimulation` · Phase 1 · Stimulation · confidence: ranged

Daily hormone injections encourage the ovaries to grow several eggs instead of the single egg that develops naturally each month. Progress is tracked with vaginal ultrasounds and blood tests.

- **stimulation_duration_days:** ASRM booklet: 8-14 days; SART: 8-10 days (most women); Mayo Clinic: 1-2 weeks
- **Source:** ASRM booklet (live); SART (from_report); Mayo Clinic (live)

### Monitoring during stimulation

`stage.monitoring` · Phase 1 · Stimulation · confidence: agreed

While stimulating, you'll have repeated vaginal ultrasounds to track follicle growth and blood tests to check hormone response. Estrogen typically rises as follicles grow, while progesterone stays low until after ovulation.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Cycle cancellation before retrieval

`stage.cycle_cancellation` · Phase 1 · Stimulation · confidence: single_source

A cycle may be cancelled before egg retrieval, most often because too few follicles are developing; other reasons include premature ovulation or an excessive response raising OHSS risk. Up to about 20% of cycles are cancelled before retrieval.

- **Source:** ASRM booklet (live)

### Trigger injection

`stage.trigger` · Phase 1 · Stimulation · confidence: agreed

When follicles are mature, a "trigger" injection (hCG or another medication) causes the eggs to finish maturing so they can be retrieved. Its exact timing is set by your clinic and is critical — egg retrieval is scheduled a fixed time afterward.

- **Source:** ASRM booklet (live); SART (from_report)

### Trigger-to-retrieval timing window

`stage.trigger_to_retrieval_window` · Phase 2 · Trigger & retrieval · confidence: ranged

Egg retrieval is scheduled a set number of hours after the trigger injection, timed to happen just before natural ovulation would occur. Sources give slightly different windows.

- **trigger_to_retrieval_hours:** ASRM booklet: 34-36 h; Mayo Clinic: 34-36 h; SART: ovulation typically 36-40 h after trigger
- **Source:** ASRM booklet (live); Mayo Clinic (live); SART (from_report)

### Egg retrieval procedure

`stage.egg_retrieval_procedure` · Phase 2 · Trigger & retrieval · confidence: agreed

Using ultrasound guidance, a thin needle is passed through the vagina into each follicle to draw out the fluid and egg. The procedure usually takes under 30 minutes and is done with sedation or pain medication; cramping and fullness afterward are common.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Fertilization (IVF vs ICSI)

`stage.fertilization` · Phase 2 · Trigger & retrieval · confidence: agreed

Retrieved eggs are mixed with sperm (conventional insemination) or, if fertilization is less likely to succeed on its own, a single sperm is injected into each mature egg (ICSI). Usually 65-75% of mature eggs fertilize.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Embryo culture

`stage.embryo_culture` · Phase 2 · Trigger & retrieval · confidence: agreed

Fertilized eggs are grown in the lab for several days. By day 3 an embryo usually has about 6-10 cells; by day 5 it may reach the blastocyst stage. Embryos can be transferred any time from day 1 to day 6 after retrieval.

- **Source:** ASRM booklet (live); Mayo Clinic (live)

### Embryo transfer

`stage.embryo_transfer` · Phase 3 · Transfer & luteal · confidence: ranged

A thin catheter guided through the cervix places one or more embryos into the uterus. No anesthesia is usually needed, though a mild sedative is an option; some cramping is common. Timing after retrieval varies by source.

- **days_after_retrieval_to_transfer:** Mayo Clinic: 2-6 days; HFEA: 2-5 days after fertilisation; MedlinePlus: 3-5 days after retrieval
- **Source:** ASRM booklet (live); Mayo Clinic (live); HFEA (live); MedlinePlus (live)

### Luteal phase support

`stage.luteal_support_stage` · Phase 3 · Transfer & luteal · confidence: agreed

After transfer, progesterone (and sometimes estrogen) is given to help the uterine lining support a pregnancy. Support is continued until the pregnancy test, and further if the result is positive, per your clinic's plan.

- **Source:** SART (from_report); MedlinePlus (live)

### Pregnancy blood test (beta-hCG) timing

`stage.pregnancy_test` · Phase 3 · Transfer & luteal · confidence: ranged

Sources give different windows for the beta-hCG blood test after embryo transfer or retrieval. All stress testing only on the exact date the clinic assigns, since testing early can give a false result.

- **days_to_pregnancy_test:** MedlinePlus: about 12-14 days after embryo transfer; Mayo Clinic: at least 12 days after egg retrieval; NHS: around 16 days after embryo transfer
- **Source:** MedlinePlus (live); Mayo Clinic (live); NHS (live)

### GnRH antagonist protocol

`stage.protocol_antagonist` · Phase 1 · Stimulation · confidence: single_source

A stimulation approach where gonadotropin injections start first, and an antagonist medication is added partway through stimulation to prevent premature ovulation. It uses fewer injection days than the long agonist protocol.

- **Source:** HFEA (live); SART (from_report)

### Long GnRH agonist protocol

`stage.protocol_long_agonist` · Phase 1 · Stimulation · confidence: single_source

A stimulation approach where a daily agonist injection or nasal spray first suppresses natural hormone production for a couple of weeks; once suppression is confirmed by scan, gonadotropin stimulation begins.

- **Source:** HFEA (live); NHS (live)

### Short / flare agonist protocol

`stage.protocol_flare` · Phase 1 · Stimulation · confidence: single_source

A stimulation approach where the agonist is started at the beginning of the cycle, alongside or just before gonadotropins, using its initial stimulating effect to help recruit follicles early. Sometimes used for lower ovarian responders.

- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

### Controlled ovarian stimulation (generic / umbrella term)

`stage.protocol_generic_coh` · Phase 1 · Stimulation · confidence: agreed

The umbrella term for using hormone medications to stimulate the ovaries to produce multiple eggs. All named protocols (antagonist, long agonist, flare) are variations of this same underlying process, differing mainly in how premature ovulation is prevented.

- **Source:** ASRM booklet (live); MedlinePlus (live)

## Medications (class-level)

### Gonadotropins (ovarian stimulation)

`med.gonadotropin` · Phase 1 · Stimulation · confidence: agreed

An injectable hormone medication (FSH, sometimes combined with LH) that drives the ovaries to grow several eggs at once. Given daily during the stimulation phase; your clinic chooses the specific product.

- **Class:** gonadotropin
- **Recognised names:** Menopur, Gonal-F, Puregon, Follistim, Bravelle, Repronex
- **Purpose:** Stimulates the ovaries to grow multiple follicles/eggs during a treatment cycle, instead of the single egg that develops in a natural cycle.
- **Interchangeable group:** gonadotropin_injectable
- **Source:** Mayo Clinic (live); SART (from_report)

### GnRH agonists (suppression / premature-ovulation prevention)

`med.gnrh_agonist` · Phase 1 · Stimulation · confidence: agreed

A medication (injection or nasal spray) that suppresses natural ovulation signals so the ovaries can be stimulated on the clinic's schedule instead of the body's own cycle.

- **Class:** gnrh_agonist
- **Recognised names:** Lupron, Leuprolide, Synarel, Suprecur, Zoladex, Decapeptyl, Diphereline, Triptorelin
- **Purpose:** Prevents the pituitary gland from triggering a premature LH surge and early ovulation during stimulation. Used in the long and flare protocols; can also be used as a trigger.
- **Interchangeable group:** gnrh_agonist_class
- **Source:** SART (from_report); HFEA (live)

### GnRH antagonists (suppression / premature-ovulation prevention)

`med.gnrh_antagonist` · Phase 1 · Stimulation · confidence: agreed

A medication used mid-stimulation to stop the ovaries from releasing eggs too early, part of the antagonist protocol.

- **Class:** gnrh_antagonist
- **Recognised names:** Cetrotide, Orgalutran, Ganirelix, Fyremadel
- **Purpose:** Immediately blocks the pituitary signal that would otherwise trigger a premature LH surge, typically started partway through stimulation once follicles reach a certain size.
- **Interchangeable group:** gnrh_antagonist_class
- **Source:** SART (from_report); HFEA (live)

### hCG trigger

`med.trigger_hcg` · Phase 1 · Stimulation · confidence: agreed

A single injection given once follicles are mature; it prompts the eggs to finish maturing ahead of a precisely timed egg retrieval.

- **Class:** trigger_hcg
- **Recognised names:** Ovidrel, Ovitrelle, Pregnyl, Novarel, Profasi
- **Purpose:** Replaces the body's natural LH surge to cause the final maturation of the eggs so they are ready to be retrieved and are capable of being fertilized.
- **Interchangeable group:** trigger_medication
- **Source:** ASRM booklet (live); SART (from_report)

### GnRH-agonist trigger

`med.trigger_agonist` · Phase 1 · Stimulation · confidence: single_source

A trigger option using a GnRH agonist instead of, or together with, hCG to mature the eggs before retrieval. Your clinic decides which trigger type fits your situation.

- **Class:** trigger_agonist
- **Recognised names:** Leuprolide, Lupron
- **Purpose:** An alternative to the hCG trigger that also causes final egg maturation ahead of retrieval; sometimes combined with hCG as a "dual trigger."
- **Interchangeable group:** trigger_medication
- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

### Progesterone (luteal support)

`med.progesterone` · Phase 3 · Transfer & luteal · confidence: agreed

A hormone given after transfer to prepare and maintain the uterine lining. Your clinic decides the form and how long to continue it, usually up to the pregnancy test and beyond if positive.

- **Class:** progesterone
- **Recognised names:** Crinone, Utrogestan, Endometrin, Cyclogest, Gestone
- **Purpose:** Supports the lining of the uterus after embryo transfer to help an embryo implant. Comes in several interchangeable forms (vaginal gel, insert, or injection) chosen by your clinic.
- **Interchangeable group:** luteal_progesterone
- **Source:** SART (from_report); MedlinePlus (live)

### Estrogen support (mainly frozen embryo transfer)

`med.estrogen` · Phase 3 · Transfer & luteal · confidence: single_source

A hormone used, chiefly in a medicated frozen embryo transfer cycle, to help build the uterine lining before and around the time of transfer.

- **Class:** estrogen
- **Recognised names:** Progynova, Estradiol valerate
- **Purpose:** Helps prepare and thicken the uterine lining, mainly used in medicated frozen embryo transfer cycles alongside progesterone.
- **Interchangeable group:** luteal_estrogen
- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

## Patient instructions

### Fasting before egg retrieval

`instr.fasting_before_retrieval` · Phase 2 · Trigger & retrieval · confidence: agreed

Because egg retrieval is done under sedation, you are asked not to eat or drink for a set period beforehand, as instructed by your clinic. Arriving without fasting can mean the procedure has to be postponed.

- **If not followed:** escalate_to_clinic
- **Source:** Mayo Clinic (live)

### No driving after sedation

`instr.no_driving_after_sedation` · Phase 2 · Trigger & retrieval · confidence: single_source

Because a sedative is used for egg retrieval, you cannot drive yourself home afterward. A responsible adult should take you home, and you should avoid driving or operating machinery for the rest of the day.

- **If not followed:** escalate_to_clinic
- **Source:** ASRM booklet (live)

### Activity limits after egg retrieval

`instr.rest_after_retrieval` · Phase 2 · Trigger & retrieval · confidence: single_source

Expect cramping and a feeling of fullness for some time after retrieval because the ovaries remain enlarged. Vigorous activity and sex should be avoided until your clinic advises it's fine to resume.

- **If not followed:** escalate_to_clinic
- **Source:** ASRM booklet (live)

### Partner ejaculatory abstinence before sample

`instr.partner_abstinence` · Phase 2 · Trigger & retrieval · confidence: single_source

Your partner may be asked to abstain from ejaculation for a set number of days before giving the semen sample used on retrieval day, as instructed by the clinic's lab.

- **If not followed:** escalate_to_clinic
- **Source:** SART (from_report)
- ⚠️ **SART-report-only** — Only reachable source was SART via the research report (live SART fetch returned an index). Report-verified, not live-verified.

### Take the trigger injection at the exact prescribed time

`instr.trigger_exact_timing` · Phase 1 · Stimulation · confidence: agreed

The trigger injection must be given at the exact time your clinic specifies, since egg retrieval is scheduled a fixed number of hours afterward. If it is given late, missed, or incorrectly, contact your clinic immediately rather than adjusting it yourself.

- **If not followed:** escalate_to_clinic
- **Source:** ASRM booklet (live); SART (from_report)

### Take the pregnancy test on the assigned date

`instr.pregnancy_test_on_assigned_date` · Phase 3 · Transfer & luteal · confidence: agreed

Take your pregnancy blood test only on the date your clinic assigns. Testing earlier can give an inaccurate result — a false positive from residual trigger medication, or a false negative if levels are still too low.

- **If not followed:** escalate_to_clinic
- **Source:** NHS (live)

## Warning signs

### Possible OHSS or other concerning symptoms — contact clinic

`warn.ohss.contact_clinic` · Cross-phase · confidence: agreed

These symptoms can occur after ovarian stimulation or embryo transfer and should be reported to your clinic promptly rather than managed on your own.

- **Tier:** contact_clinic
- **Triggers on:** pain and bloating in the tummy, feeling or being sick, feeling faint, coughing up blood, vaginal bleeding or brown watery discharge, pain or discomfort passing urine or stool
- **Verbatim redirect:** These can be signs that need clinic assessment. Please contact the clinic's emergency line now. If you cannot reach them or feel very unwell, seek urgent medical care immediately.
- **Source:** NHS (live); AVA-Peter card (clinic_card)
- ⚠️ **verify-clinic-number** — Redirect uses the AVA-Peter card (2019). Confirm the phone numbers are still current with the clinic.

### Possible severe OHSS / urgent symptoms

`warn.ohss.urgent` · Cross-phase · confidence: agreed

Severe ovarian hyperstimulation syndrome (OHSS) is uncommon but serious. Warning signs include difficulty breathing, severe or one-sided abdominal pain, shoulder-tip pain, rapid swelling, and passing very little urine.

- **Tier:** urgent_care
- **Triggers on:** difficulty breathing, severe or one-sided tummy pain, shoulder-tip pain, passing very little urine / very thirsty, rapid swelling or weight gain, chest or upper-back pain
- **Verbatim redirect:** These can be signs of a serious problem. Please contact the clinic's emergency line now. If you cannot reach them or feel very unwell, seek urgent medical care immediately.
- **Source:** NHS (live); AVA-Peter card (clinic_card)
- ⚠️ **verify-clinic-number** — Redirect uses the AVA-Peter card (2019). Confirm the phone numbers are still current with the clinic.

### Moderate/severe pain or heavy bleeding after embryo transfer

`warn.post_transfer.bleeding_pain` · Phase 3 · Transfer & luteal · confidence: single_source

Mild cramping and spotting are common after embryo transfer, but moderate-to-severe pain or heavy bleeding should be reported to your care team rather than waited out.

- **Tier:** contact_clinic
- **Triggers on:** moderate or severe pain after embryo transfer, heavy vaginal bleeding after embryo transfer
- **Verbatim redirect:** Please contact the clinic's emergency line now. If you cannot reach them or feel very unwell, seek urgent medical care immediately.
- **Source:** Mayo Clinic (live); AVA-Peter card (clinic_card)
- ⚠️ **verify-clinic-number** — Redirect uses the AVA-Peter card (2019). Confirm the phone numbers are still current with the clinic.

### Fever, pelvic pain, or blood in urine after IVF

`warn.post_ivf.fever_infection` · Cross-phase · confidence: single_source

A fever, new pelvic pain, heavy bleeding, or blood in the urine after an IVF procedure can signal infection or another complication and should be reported right away.

- **Tier:** contact_clinic
- **Triggers on:** fever over 100.5F / 38C, pelvic pain, heavy vaginal bleeding, blood in the urine
- **Verbatim redirect:** Please contact the clinic's emergency line now. If you cannot reach them or feel very unwell, seek urgent medical care immediately.
- **Source:** MedlinePlus (live); AVA-Peter card (clinic_card)
- ⚠️ **verify-clinic-number** — Redirect uses the AVA-Peter card (2019). Confirm the phone numbers are still current with the clinic.

### Possible ectopic pregnancy signs

`warn.ectopic_possible` · Phase 3 · Transfer & luteal · confidence: single_source

Low one-sided abdominal pain or shoulder-tip pain after a positive pregnancy test can, per guideline sources, indicate either severe OHSS or an ectopic pregnancy and needs prompt evaluation.

- **Tier:** urgent_care
- **Triggers on:** tummy pain low down on one side, pain in the tip of the shoulder, dehydration signs after positive test
- **Verbatim redirect:** These can be signs of a serious problem. Please contact the clinic's emergency line now. If you cannot reach them or feel very unwell, seek urgent medical care immediately.
- **Source:** NHS (live); AVA-Peter card (clinic_card)
- ⚠️ **verify-clinic-number** — Redirect uses the AVA-Peter card (2019). Confirm the phone numbers are still current with the clinic.
