# IVF / ART Patient-Education Knowledge Base and Protocol Model

## TL;DR
- A controlled-ovarian-stimulation IVF cycle is described to patients everywhere as the same three-phase arc: (1) daily ovarian stimulation with monitoring, (2) a precisely-timed trigger injection followed ~34–36 hours later by egg retrieval, and (3) fertilization/culture, embryo transfer, luteal (progesterone ± estrogen) support, and a beta-hCG blood test. All the "named protocols" (GnRH antagonist, long agonist, short/flare agonist) are just variations in HOW premature ovulation is prevented during phase 1 — they map onto the identical three-phase skeleton.
- The authoritative patient sources (ASRM/reproductivefacts.org, SART, NHS, HFEA, Mayo Clinic, MedlinePlus) agree closely on the stages, the medication classes, the standard non-medication instructions (fasting before retrieval, no driving after sedation, exact trigger timing), and the "contact your clinic if…" warning signs (chiefly OHSS). Where they differ it is on numbers (trigger-to-retrieval window, beta-hCG timing, stimulation duration), so the app should present ranges with attribution, never a single collapsed value.
- Everything the app surfaces should be framed as vetted, general patient-education content and capture-and-escalate symptom logging — NOT as medical judgment. Dosing, protocol choice, medication adjustments, and any interpretation of a patient's symptoms/labs are individualized and clinician-determined, and must be flagged OUT OF SCOPE.

## Key Findings
1. **A single three-phase model fits every cycle.** Stimulation → trigger+retrieval → transfer+luteal support+test. This is the correct backbone for the "protocol model."
2. **Protocol names describe phase-1 suppression only.** Antagonist vs long-agonist vs flare differ in when/how a premature LH surge is blocked, not in the overall shape.
3. **Trigger timing is the single most time-critical patient action.** Sources converge on egg retrieval ~34–36 hours after the trigger; the app's job is to reinforce exact timing and escalate any deviation to the clinic — never to advise a correction.
4. **Warning-sign content maps cleanly to capture-and-escalate.** OHSS, bleeding, severe pain, fever/infection signs are all framed by authorities as "contact your clinic / seek urgent care if you experience X," which is exactly the non-assessing posture the app requires.
5. **Medication recognition is well-bounded.** A finite, internationally-recognizable list of gonadotropins, GnRH agonists/antagonists, triggers, and luteal-support agents can be parsed by class/purpose without ever touching dose.

## Details

### 1. The standard stages / phases of an IVF cycle (patient-facing)

**Overall duration.** Mayo Clinic: "One full cycle of IVF takes about 2 to 3 weeks. Sometimes these steps are split into different parts and the process can take longer." NHS: "A full cycle of IVF takes around 3 to 6 weeks to complete." HFEA: "For most people, one cycle of IVF will take between four and six weeks," dropping to "around three weeks" if the suppression stage is skipped. SART's step-by-step guide says each step "occurs at a specific time during a two to six-week period." → The app should present "about 2–6 weeks" with attribution rather than one figure.

**Stage A — Ovarian stimulation.** Lab-made hormones stimulate the ovaries to grow multiple eggs "rather than the single egg that normally develops each month" (ASRM booklet). MedlinePlus labels this "Stimulation, also called super ovulation." Mayo notes the shot "may contain a follicle-stimulating hormone (FSH), a luteinizing hormone (LH) or both." Duration: ASRM booklet "Generally, 8 to 14 days of stimulation are required"; SART "Most women require between 8 to 10 days of gonadotropin therapy"; Mayo "1 to 2 weeks of ovarian stimulation."

**Stage A monitoring — ultrasound + bloodwork.** ASRM: "The ovaries are evaluated during treatment with vaginal ultrasound examinations to monitor the development of ovarian follicles. Blood samples are drawn to measure the response to ovarian stimulation medications. Normally, estrogen levels increase as the follicles develop, and progesterone levels are low until after ovulation." Mayo: vaginal ultrasound tracks developing follicles; blood tests check estrogen (rises) and progesterone (stays low until after ovulation).

**Stage B — Trigger shot.** When follicles are mature, "hCG or other medications are given. The hCG replaces the woman's natural LH surge and causes the final stage of egg maturation so the eggs are capable of being fertilized" (ASRM). SART: "Both leuprolide acetate (Lupron®) and Human chorionic gonadotropin (hCG) (Profasi®, Novarel®, Pregnyl®, Ovidrel®) are hormonal drugs that stimulate the final maturation of the oocytes… The time of the trigger injection determines when the egg retrieval will be scheduled. Ovulation typically occurs 36-40 hours after the trigger injection of Lupron or hCG." Significance of timing: retrieval is deliberately scheduled BEFORE natural ovulation (~38–40h) would release the eggs.

**Stage B — Egg retrieval / follicular puncture, and the trigger-to-retrieval window.** ASRM: eggs "are retrieved before ovulation occurs, usually 34 to 36 hours after the hCG injection is given." Mayo: "The procedure is done 34 to 36 hours after the final shot of fertility medicine and before ovulation." MedlinePlus calls it "follicular aspiration"; Mayo/ASRM call it "transvaginal ultrasound aspiration." Procedure: ultrasound probe + needle through the vaginal wall aspirate follicular fluid; ~20–30 minutes; done under sedation/pain medication; cramping/fullness afterward. → Report the window as "34–36 hours" (ASRM, Mayo), noting some clinics phrase it "about 36 hours."

**Stage C — Fertilization (IVF vs ICSI).** Conventional insemination = "motile sperm are placed together with the oocytes and incubated overnight" (ASRM). ICSI = "a single sperm is directly injected into each mature egg" (ASRM), used mainly for male-factor infertility or prior fertilization failure. On how common ICSI is, sources differ and the app should present the range with attribution: the ASRM patient booklet (2015 ed.) says "In the United States, ICSI is performed in approximately 60% of ART cycles," whereas Boulet et al., JAMA 2015 (national surveillance data) found "the proportion of fresh IVF cycles using ICSI increased from 36.4% (15,073/41,450) in 1996 to 76.2% (74,512/97,756) in 2012." ASRM: "Usually 65% to 75% of mature eggs will fertilize."

**Stage C — Embryo culture.** By day 3 "approximately 6 to 10 cells"; by day 5 "a fluid cavity forms in the embryo… An embryo at this stage is called a blastocyst" (ASRM). Embryos may be transferred "at any time between one and six days after the egg retrieval."

**Stage C — Embryo transfer (fresh vs frozen).** A thin catheter passes through the cervix to place the embryo(s) in the uterus. ASRM: "No anesthesia is necessary, although some women may wish to have a mild sedative… usually painless, although some women experience mild cramping." Mayo: transfer "often takes place 2 to 6 days after eggs are collected." Fresh = transfer in the same cycle; frozen embryo transfer (FET) = embryos cryopreserved and transferred in a later cycle (with estradiol + progesterone in a medicated FET, per SART).

**Stage C — Luteal phase support.** MedlinePlus: "Women who undergo IVF must take daily shots or pills of the hormone progesterone for 8 to 10 weeks after the embryo transfer. Progesterone… prepares the lining of the uterus so that an embryo can attach." SART: "In a fresh embryo transfer, progesterone is routinely given. In a medicated frozen embryo transfer, estradiol and progesterone supplementation are given. Hormone support is given until the pregnancy test… typically continued until 9-12 weeks of gestational age if the pregnancy test is positive." Routes (class-level only): vaginal (gel/insert/pessary), intramuscular (progesterone in oil), subcutaneous, and oral; plus estrogen support in some protocols. HFEA: "Medication will help to prepare the lining of the womb. This is usually taken as a pessary or gel which you can insert yourself into the vagina/rectum."

**Stage C — Pregnancy test (beta-hCG).** Sources differ on timing: MedlinePlus "About 12 to 14 days after the embryo transfer"; Mayo "At least 12 days after egg retrieval"; NHS "around 16 days after the embryo transfer"; clinic patient materials commonly "9 to 14 days after embryo transfer." → Present the range (~9–16 days depending on source/clinic and whether measured from retrieval or transfer) with attribution. All sources stress taking the test only on the clinic-assigned date, because testing early can give false results (residual trigger hCG → false positive; too-low levels → false negative). NHS: "It's important to take your test on the date you're given to get an accurate result."

### 2. Common stimulation protocol types

All of these are variations of "controlled ovarian (hyper)stimulation" / "superovulation" and share the same three-phase skeleton; they differ only in how phase 1 prevents a premature LH surge.

- **Controlled ovarian hyperstimulation (COH) / superovulation / ovulation induction.** ASRM glossary: "Ovulation induction. The administration of hormone medications (ovulation drugs) that stimulate the ovaries to produce multiple eggs. Sometimes called enhanced follicular recruitment or controlled ovarian hyperstimulation." This is the umbrella term for the whole stimulation phase.

- **GnRH antagonist protocol.** SART: antagonists (ganirelix/cetrorelix) "are typically administered several days after stimulation, commonly when the lead follicle is 14 mm or greater, and require fewer injections." HFEA "antagonist protocol": "taking medication (an antagonist) to suppress your hormones for a few days after you have taken the hormone medication… to boost the number of eggs." Shape: start gonadotropins first → add antagonist mid-stimulation → trigger → retrieval. It is shorter and uses fewer injections, and the evidence shows a meaningfully lower OHSS risk versus the long agonist protocol: the Cochrane review (Al-Inany et al., 2016, CD001750.pub4) states that "if the risk of OHSS following GnRH agonist is assumed to be 11%, the risk following GnRH antagonist would be between 6% and 9%," and a 1,050-patient RCT (Toftager et al., Human Reproduction 2016) found a significant reduction in both severe OHSS (5.1% vs 8.9%, P=.02) and moderate OHSS (10.2% vs 15.6%, P=.01) with the antagonist protocol, with no difference in live birth rate (22.8% vs 24.4%). (These comparative statistics are clinical background for content design, not patient-facing claims the app should assert.)

- **Long GnRH agonist protocol.** HFEA "long protocol": "suppress natural hormones before taking hormone medication to stimulate the ovaries… a daily injection or nasal spray to suppress hormone production. A scan checks the woman's natural cycle is fully suppressed. If it is, hormone treatment (usually gonadotrophin) is started." NHS step 1 mirrors this: "You use an injection or nasal spray every day for 2 to 3 weeks to stop your ovaries producing eggs naturally." Shape: agonist FIRST to down-regulate/suppress the pituitary → then gonadotropins → trigger → retrieval.

- **Short / "flare" (and microdose-flare) agonist protocol.** SART: "Some protocols also might begin GnRH agonist… after the start of menses in the 'flare' or 'micro-flare' protocol." The agonist's initial stimulating ("flare") effect is used to boost early follicle recruitment; often used for poorer responders. Shape: agonist started at cycle start (concurrent with/just before gonadotropins) to exploit the initial FSH/LH surge → trigger → retrieval.

**Mapping to the three-phase model:** In every case Phase 1 = daily gonadotropin stimulation + suppression method + monitoring; Phase 2 = timed trigger + retrieval at ~34–36h; Phase 3 = fertilization/culture, transfer, luteal support, beta-hCG. The protocol name only tags the Phase-1 suppression strategy.

### 3. Medication classes (class/purpose level; recognition only — NOT dosing)

- **Gonadotropins (stimulate the ovaries; FSH ± LH).** SART: "Menopur® is a combination of FSH and LH, and Follistim® and Gonal-F® only contain FSH. They are recombinant products." Menotropins/hMG (FSH+LH from urine of postmenopausal women, e.g., Menopur; historically Repronex, Bravelle/urofollitropin). Follitropin alfa/beta = recombinant FSH (Gonal-F; Puregon/Follistim). Purpose: grow multiple follicles.
- **GnRH agonists (prevent premature LH surge; also usable as trigger).** Leuprolide/leuprorelin (Lupron); also nafarelin (Synarel), buserelin (Suprecur), goserelin (Zoladex), triptorelin (Decapeptyl/Diphereline). Delivered by injection or nasal spray; short-acting daily vs long-acting depot (SART).
- **GnRH antagonists (immediate suppression of LH/FSH mid-stimulation).** Ganirelix (Orgalutran; Antagon in US), cetrorelix (Cetrotide), and ganirelix under the brand Fyremadel. SART lists "Cetrotide®, Fyremadel®, and Ganirelix®."
- **Trigger — hCG or GnRH-agonist.** hCG: choriogonadotropin alfa (Ovidrel/Ovitrelle), urinary hCG (Pregnyl, Novarel, Profasi). GnRH-agonist trigger: leuprolide (Lupron), often used to reduce OHSS risk. "Dual trigger" = hCG + agonist. SART: "Ovidrel, Novarel, Pregnyl, and hCG are all the same drug."
- **Luteal support — progesterone ± estrogen.** Progesterone routes: vaginal gel (Crinone), vaginal inserts/capsules (Endometrin, Utrogestan, Cyclogest), intramuscular progesterone in oil (Gestone), subcutaneous, and oral. Estrogen support: estradiol valerate (Progynova) and other estradiol forms, chiefly in medicated FET.

For a parser, the recognizable international brand/generic set includes: Menopur, Gonal-F, Puregon/Follistim, Bravelle, Repronex, Cetrotide, Orgalutran/Ganirelix, Fyremadel, Lupron/Leuprolide, Synarel, Suprecur, Zoladex, Decapeptyl/Diphereline/Triptorelin, Pregnyl, Novarel, Profasi, Ovitrelle/Ovidrel, Crinone, Utrogestan, Endometrin, Cyclogest, Gestone, Progynova/estradiol valerate.

**OUT OF SCOPE (flag explicitly):** doses/IU, titration, which protocol or drug a given patient should use, and any adjustment for response. SART itself notes dosage "may vary depending on the patient's history" and is set by ultrasound/estradiol results — i.e., clinician-determined.

### 4. Standard non-medication patient instructions

- **Fasting before egg retrieval (anesthesia safety).** Because retrieval uses sedation/anesthesia, patients are told to fast — commonly "nothing to eat or drink after midnight" the night before, or an 8-hour (some clinics 6–7 hour) window. This is a universal anesthesia-safety rule; arriving un-fasted typically causes the procedure to be postponed.
- **No driving / activity after retrieval (sedation).** Patients cannot drive or travel home alone after sedation; a responsible adult should drive them and often stay with them; no driving/operating machinery for ~24 hours. Rest the remainder of the day; expect mild cramping/spotting/bloating. Avoid vigorous activity and sex while ovaries are enlarged (Mayo).
- **Partner abstinence before retrieval.** Clinic patient instructions commonly specify 2–5 days of ejaculatory abstinence before the semen sample. This sits within the diagnostic laboratory standard: the WHO Laboratory Manual for the Examination and Processing of Human Semen (6th edition, 2021) recommends "an abstinence period of 2 to 7 days" (ESHRE recommends a narrower 3–4 days). Sample given by masturbation the morning of retrieval, or a frozen backup arranged in advance.
- **Missed / mistimed doses — especially the trigger.** The consistent patient-education message: take the trigger at the EXACT prescribed time, set alarms, and if a dose is late, missed, spilled, or given at the wrong time, contact the clinic immediately rather than self-correcting. → The app must escalate any trigger-timing issue to the clinic and NOT advise a fix (dose timing is clinician-determined).
- **General "call your clinic if…".** All patient sources direct patients to phone the clinic for concerning symptoms after retrieval/transfer (below) rather than self-managing.

### 5. Warning signs / when to contact the clinic (capture-and-escalate, never assess)

Frame strictly as "contact your clinic / seek urgent care if you experience X."

- **OHSS (ovarian hyperstimulation syndrome).** Mayo: swollen, painful ovaries; "mild belly pain, bloating, upset stomach, vomiting and diarrhea," and rarely "rapid weight gain and shortness of breath." MedlinePlus lists "abdominal pain, bloating, rapid weight gain (10 pounds or 4.5 kilograms within 3 to 5 days), decreased urination despite drinking plenty of fluids, nausea, vomiting, and shortness of breath."
- **NHS "contact clinic / NHS 111 as soon as possible if":** pain and bloating in the tummy; feeling and being sick; feeling faint; coughing up blood; vaginal bleeding or brown watery discharge; pain/discomfort passing urine or stool.
- **NHS "call 999 or go to A&E if":** difficulty breathing; chest or upper-back pain; very thirsty and passing little urine (dehydration); swelling anywhere; tummy pain low down on one side; pain in the tip of the shoulder — "could be severe symptoms of ovarian hyperstimulation syndrome or an ectopic pregnancy."
- **Mayo post-transfer:** "Call your care team if you have moderate or severe pain, or heavy bleeding from the vagina after the embryo transfer."
- **MedlinePlus post-IVF, contact provider right away if:** fever over 100.5°F (38°C); pelvic pain; heavy vaginal bleeding; blood in the urine.

The app should present these verbatim as escalation triggers, log the patient's report, and route to the clinic — never grade severity or reassure.

### 6. Key patient-facing glossary (vetted definitions, largely ASRM)

- **Follicle** — "A fluid-filled structure in the ovary containing an egg and the surrounding cells that produce hormones."
- **Endometrium / uterine lining** — the lining of the uterus that thickens to allow implantation (ASRM: uterus lining "called the endometrium").
- **Oocyte / egg** — "The female sex cell (ovum) produced by the ovary."
- **Estradiol** — "The predominant estrogen produced by the follicular cells of the ovary" (monitored during stimulation).
- **Egg retrieval / oocyte aspiration** — "The procedure in which eggs are obtained by inserting a needle into the ovarian follicle and removing the fluid and the egg by suction."
- **Transvaginal ultrasound aspiration** — ultrasound-guided needle egg retrieval through the vaginal wall.
- **ICSI** — "A micromanipulation procedure in which a single sperm is injected directly into an egg."
- **Embryo** — "A fertilized egg that has begun cell division." **Embryo culture** — "Growth of the embryo in a laboratory (culture) dish."
- **Blastocyst** — "An embryo that has formed a fluid-filled cavity and the cells have begun to form the early placenta and embryo, usually 5 days after ovulation or egg retrieval."
- **Embryo transfer** — placement of an embryo into the uterus.
- **Zona pellucida** — "The egg's outer layer that a sperm must penetrate"; **assisted hatching** opens it before transfer.
- **hCG** — "A hormone produced by the placenta; its detection is the basis for most pregnancy tests. Also refers to the medication used to induce ovulation and during the final stages of egg maturation."
- **Beta-hCG** — the quantitative blood test measuring hCG to confirm/monitor early pregnancy after transfer.
- **Progesterone** — "A female hormone secreted during the second half of the menstrual cycle. It prepares the lining of the uterus for implantation of a fertilized egg." **Luteal phase** — the post-ovulation/post-retrieval phase supported by progesterone.
- **GnRH agonists** — analogs that "initially stimulate the pituitary… followed by a delayed suppressive effect." **GnRH antagonists** — analogs with "an immediate suppressive effect on the pituitary gland." Both used "to prevent premature ovulation."
- **Gonadotropins (FSH/LH/hMG)** — injectable hormones that stimulate follicle growth.
- **OHSS** — "A condition that may result from ovulation induction characterized by enlargement of the ovaries, fluid retention, and weight gain."
- **Biochemical pregnancy** — "When a woman's pregnancy test is initially positive but becomes negative before a gestational sac is visible on ultrasound." **Clinical pregnancy** — "confirmed by an increasing level of hCG and the presence of a gestational sac detected by ultrasound."
- **Cryopreservation / vitrification** — freezing embryos/eggs/sperm; vitrification = ultra-rapid freezing. **FET** — frozen embryo transfer cycle.
- **Ovarian reserve / AMH / antral follicle count** — measures of egg supply used in pre-cycle testing.

## Recommendations

**Stage 1 — Model the cycle as a fixed three-phase state machine.** Build the protocol model as Phase 1 (stimulation + suppression + monitoring), Phase 2 (trigger + retrieval), Phase 3 (fertilization/culture + transfer + luteal support + beta-hCG). Attach a `protocol_type` attribute (antagonist / long-agonist / short-flare / COH-generic) that only modifies Phase-1 items. This matches how every authority explains the process and keeps the model source-faithful.

**Stage 2 — Seed the knowledge base from the priority sources with citation-or-refuse.** Prefer ASRM booklet + SART for US framing and glossary, NHS + HFEA for UK framing, Mayo + MedlinePlus for plain-language cross-checks. Store each fact with its source tag; if a query has no grounded, sourced answer, the app refuses and routes to the clinic. Present ranges (duration ~2–6 weeks; trigger-to-retrieval 34–36h; beta-hCG ~9–16 days) with attribution, never a single number.

**Stage 3 — Implement symptom logging as capture-and-escalate only.** Encode the NHS two-tier list (clinic/111 vs 999/A&E), Mayo post-transfer, and MedlinePlus post-IVF triggers as literal escalation rules. The app logs the report and surfaces the matching "contact your clinic/seek urgent care" message verbatim; it never scores severity, interprets, or reassures.

**Stage 4 — Build a medication recognizer keyed to class/purpose.** Parse the brand/generic list into {gonadotropin, GnRH-agonist, GnRH-antagonist, trigger-hCG, trigger-agonist, progesterone, estrogen}. Surface only the vetted class description. Hard-block any generation of doses, schedules, or adjustments.

**Thresholds that change the design:** (a) If a source updates its stated window/timing (e.g., ASRM revises the 34–36h figure), update the stored range and attribution rather than the code path. (b) If clinics using the app follow protocols not covered by the three-phase model (e.g., natural-cycle IVF, IVM, duo-stim), add them as explicit `protocol_type` variants with their own sourced content before enabling related Q&A. (c) If patient-facing regulators (HFEA/NICE/ASRM) publish revised warning-sign wording, replace the escalation strings verbatim.

## Caveats
- **Individualized/clinician-determined items are OUT OF SCOPE and must never be presented as guidance:** drug doses and units, protocol selection, medication changes/titration, cycle cancellation decisions, and any interpretation of a specific patient's symptoms, ultrasound findings, or lab/hormone values. Multiple sources explicitly note these are set by the care team based on the individual.
- **Numbers legitimately differ across reputable sources** (cycle length, trigger-to-retrieval window, beta-hCG timing, stimulation duration, ICSI utilization). These are framing/clinic-practice/vintage differences, not errors; report ranges with attribution.
- **The comparative OHSS/live-birth statistics in §2 (Cochrane 2016; Toftager 2016 RCT) are clinical background** to justify why protocols exist and are chosen — they are NOT claims the app should present to patients, and the app must not use them to imply one protocol is "better" for a given user (protocol choice is clinician-determined).
- **Some retrieved pages are commercial clinic blogs.** Their factual claims (trigger timing, fasting, OHSS symptom lists) corroborate the authoritative sources but should not be cited as primary; anchor the knowledge base on ASRM/SART/NHS/HFEA/Mayo/MedlinePlus.
- **UpToDate "Beyond the Basics"** could not be retrieved (paywalled/blocked); its framing is cross-checked via the guideline bodies above. **SART pages** intermittently return server errors on direct fetch; content here was verified via full fetch of the step-by-step guide and via consistent quoted snippets for the medications page.
- **The ASRM booklet quoted is the Revised 2015/2018 patient edition.** Verify against the current edition on reproductivefacts.org before publishing, as figures (e.g., ICSI proportion) may be updated; note the ASRM booklet's "~60%" is superseded by later national surveillance data (76.2% of fresh cycles in 2012, JAMA 2015).
- **Source dates:** Mayo reviewed Sept 2023; NHS reviewed Apr 2025; MedlinePlus updated/reviewed 2026; HFEA IVF page publication 2016 (ongoing). Confirm currency at ingestion time.
