# Zigota — Protocol Model (v2, evidence-grounded)

*Phase 1 design artifact. Companion to `Project_Zigota__IVF_ART_Patient-Education_Knowledge_Base_and_Three-Phase_Protocol_Model.md` (the source-attributed research report / knowledge base). This document describes **the states the product must represent** — the phases, the item types, and their lifecycle. It is deliberately informal: the concrete data schema is Phase 2's job. Every clinical fact referenced here is sourced in the research report; this document does not re-cite.*

**What changed from v1 (single-card model).** v1 was derived from one real prescription card (AVA-Peter superovulation protocol). The evidence-based research confirmed most of it and forced four changes: (1) an outer **three-phase arc** now sits above the item types; (2) `protocol_type` is confirmed as a **Phase-1-only** modifier; (3) a **monitoring/appointment** item type was missing and is now first-class; (4) the single "conditional" type **splits into comfort-conditional (D1) and warning-sign escalation (D2)**, which must behave differently on safety grounds.

---

## 1. The backbone: a fixed three-phase arc

Every controlled-ovarian-stimulation IVF cycle — regardless of clinic or country — is described to patients as the same sequence. The model treats this as a fixed spine; individual cards are instances of it.

| Phase | What it is | Item types that live here | Ends at |
|---|---|---|---|
| **Phase 1 — Stimulation & monitoring** | Daily gonadotropin stimulation + a suppression method + recurring clinic monitoring (ultrasound + bloodwork) | A (fixed daily dose), Mon (monitoring visit), + suppression items whose presence depends on `protocol_type` | The trigger |
| **Phase 2 — Trigger & retrieval** | The single precisely-timed trigger injection, then egg retrieval ~34–36h later | B (single timed event), E (procedure-prep instructions: fasting, no-driving, abstinence) | Retrieval done |
| **Phase 3 — Transfer, luteal support & test** | Fertilization/culture, embryo transfer, luteal support (progesterone ± estrogen), beta-hCG test | C (continue-until-event regimen), D1 (comfort conditional), D2 (warning-sign escalation), the beta-hCG test as a dated event | Beta-hCG result |

The **trigger is the hinge** between Phase 1 and Phase 2 and carries the model's heaviest safety weight (see Type B). The AVA-Peter card maps cleanly: its p5 stimulation grid is Phase 1; its printed trigger row is the Phase 1→2 hinge; its p6 post-transfer memo is Phase 3.

### `protocol_type` — a Phase-1-only attribute

The named protocols (**GnRH antagonist**, **long agonist**, **short/flare agonist**, or generic **COH/superovulation**) differ *only* in how Phase 1 prevents a premature LH surge. They do **not** change Phase 2 or Phase 3. So the model needs no per-protocol branching — just one attribute on the cycle that determines which suppression items appear in Phase 1:

- **antagonist** → gonadotropins from the start; an antagonist item added mid-stimulation.
- **long-agonist** → an agonist down-regulation item *precedes* gonadotropins.
- **short/flare** → agonist started with/just before gonadotropins.
- **generic** → gonadotropins only (suppression not represented / not on card).

The AVA-Peter card lists Декапептил/Диферелин (triptorelin, a GnRH agonist) among the trigger-row alternatives, consistent with an agonist-based protocol. The model records `protocol_type` when it is inferable from the parsed drugs, and otherwise leaves it `unknown` — it never *guesses* the protocol, because protocol choice is clinician-determined and out of scope.

---

## 2. The core split: prescribed layer vs. scheduled layer

**This is the central modeling insight.** Every item has two layers:

- **Prescribed layer** — what the card states: drug/instruction, dose, route, frequency, anchor. Stable; comes from the parse.
- **Scheduled layer** — concrete dates/times. *Frequently absent at parse time.*

The AVA-Peter card shows this directly: doses are often pre-printed (trigger = 10000 ЕД) while the **Date and Time columns are blank for the clinician to fill by hand**. v1 treated this as "the clinic was lazy." The research reframes it correctly and more honestly: **the dates are blank because the protocol is adaptive.** Stimulation length and the trigger moment are set *reactively*, during monitoring, in response to how the follicles and estradiol respond. The blanks are a feature of how IVF works, not an omission.

Consequences for the model:
- The parse populates the **prescribed** layer only. It must never invent dates.
- An item with a prescribed dose but no date is a valid, first-class state: **"prescribed, not yet scheduled."** It is visible to the patient but is *not* an active reminder.
- The **scheduled** layer is filled by the patient (confirming/entering dates the clinic gives them) or, later, potentially by a clinic integration — never by the AI.
- This split is exactly what makes "AI proposes, human ratifies" enforceable: the AI owns the prescribed layer; the human owns the scheduled layer.

---

## 3. Item types

Six types. Each item carries: `phase`, `type`, prescribed layer, scheduled layer, `alternatives[]`, and `confirmation_state`.

### Type A — Fixed daily dose over a window
Recurring dose, once or twice daily, across a span of days (the stimulation gonadotropins; the suppression agonist/antagonist; most luteal drugs before their end date is known).
- Anchor: time-of-day and/or meal; the AVA-Peter grid also tracks **by cycle-day**, with a per-row date the clinician fills.
- The closest thing to an ordinary recurring reminder — but still subject to the prescribed/scheduled split (the window's start/end may be open).

### Type B — Single precisely-timed event (the trigger)
One injection, one exact moment. Categorically distinct from Type A with n=1.
- **Why separate:** the research independently confirms (across ASRM, Mayo, SART, NHS) that exact trigger timing is *the* most time-critical patient action, and that egg retrieval is scheduled ~34–36h after it. Mistiming can make the cycle fail. This is universal, not one clinic's emphasis — so the categorical separation rests on evidence, not tone.
- **Model obligations:** requires its own **distinct, deliberate confirmation** (never the one-tap of a daily vitamin); on any reported timing problem (late/missed/wrong-time/spilled), it triggers **escalate-to-clinic**, never a self-correction. Dose-timing correction is clinician-determined and out of scope.
- Alternatives are common here (see §4): hCG-trigger *or* agonist-trigger *or* dual.

### Type Mon — Monitoring / appointment
A clinic visit for ultrasound + bloodwork during Phase 1 (and the retrieval, transfer, and beta-hCG visits).
- **New in v2.** v1, built from the drug grid alone, missed that Phase 1 is not just "take drugs" but "take drugs *and come in on clinic-set dates as they watch your response*."
- Almost always **prescribed-but-unscheduled**: the next monitoring date is set at the *previous* visit, reactively. This is a second, independent confirmation of the prescribed/scheduled split.
- The app surfaces and reminds; it never predicts the next date.

### Type C — Continue-until-date/event regimen
A daily item whose **end is an event that hasn't happened yet** at parse time.
- The luteal support: "taken until the beta-hCG test day, inclusive" — a date that is blank on the card and unknown until the clinic sets it.
- Model needs an **open-ended duration** that resolves when its terminating event acquires a date: `until-event(beta_hCG)`, `until-date(<blank>)`, `until-instructed`.
- Until the end resolves, the item runs as a Type-A-like daily reminder with an unbounded tail; when the terminating event is dated, the tail closes.

### Type D1 — Card-stated conditional (comfort / as-needed)
An `if <condition> then <action>` rule **written on the card** for patient comfort or minor contingencies.
- AVA-Peter examples: "at pain → Кетонал suppositories or other NSAIDs"; "ice on abdomen day of puncture, 3× for 30 min"; "if spotting after transfer → add Прогестерон / Дицинон / Транексам."
- **Boundary:** because the *card itself* states these, surfacing them is **transcription, not advice.** But the app presents them strictly as *"your card says: if X, then Y"* — it never asserts that condition X has occurred, never decides for the patient that the trigger is met, and never recommends on its own initiative.

### Type D2 — Warning-sign → escalate (capture-and-escalate)
A red-flag rule that routes the patient to care. **Split out from D1 in v2 on safety grounds.**
- Sourced from the research's two-tier escalation set (NHS clinic/111 vs 999/A&E; Mayo post-transfer; MedlinePlus post-IVF) plus any red flags the card names. Chief concern: **OHSS.**
- **Behaviour differs fundamentally from D1:** D2 is not a card convenience, it is a safety net. When a patient logs a symptom matching a D2 pattern, the app **logs it and surfaces the matching "contact your clinic / seek urgent care" message verbatim.** It never grades severity, never interprets, never reassures. (This is the symptom-capture feature's escalation path — see §5.)
- D2 rules are general patient-education content, present for *every* cycle regardless of what any individual card lists.

### Type E — Non-medication instruction
Instructions with no dose, mostly clustered around Phase 2.
- Fasting before retrieval (anesthesia safety); no alcohol/driving/machinery ~24h after sedation; partner abstinence 2–7 days before the sample; "come with your partner"; "keep this card and record self-administered doses."
- Surfaced and reminded as procedure-prep; escalated if reported unmet (e.g., patient ate before a fasting-required retrieval → route to clinic); **never assessed.**

---

## 4. Cross-cutting attribute: either/or alternatives

One prescription slot, mutually exclusive fills. Confirmed by the research to be **structural, not a card quirk** — luteal progesterone routinely ships as interchangeable brands/routes, and the trigger and estrogen support likewise.

- AVA-Peter examples: Прогинова **or** Дивигель (estrogen); Крайнон **or** Утрожестан (progesterone); trigger = Прегнил **or** Овитрель **or** Декапептил/Диферелин.
- **Model obligation:** the parse must **not silently pick one.** The item holds `alternatives[]`, and the item **cannot leave the proposed state until the patient confirms which drug they were actually given.** This is a hard confirmation gate, like the Type B trigger gate.

---

## 5. Item lifecycle (confirmation states & transitions)

The safety boundary is enforced by the state machine, not by prompt wording.

```
[parsed] --confirm--> [active] --complete--> [done]
   |                     |
   |                     +--report-problem--> [escalated]   (Type B timing, Type E unmet, Type D2 match)
   |
   +-- (prescribed, no date) --> [prescribed_unscheduled] --date-added--> [active]
   +-- (has alternatives)   --> [needs_selection]        --patient-picks--> [confirmable]
   +-- (Type B trigger)     --> [needs_deliberate_confirm]
```

Hard rules (each is written to be **testable** — see the acceptance-criteria artifact):
- **No item enters `active` without an explicit patient confirm.** The parse writes only to `parsed`. This is the single transition the AI may never perform autonomously.
- **Type B requires a distinct, deliberate confirmation** — not the same gesture as a daily item.
- **An item with `alternatives[]` cannot be confirmed** until the patient selects which drug was given.
- **A prescribed-but-undated item stays `prescribed_unscheduled`** and never fires a reminder until a date exists.
- **Type D2 matches route to `escalated` with a verbatim clinic-redirect** and emit no assessment token.

---

## 6. What the model deliberately does NOT represent (non-requirements)

Restated from the vision brief §5 and confirmed by the research's OUT-OF-SCOPE list. Each is a testable "must-not":
- No dose values reasoned about, suggested, adjusted, or optimized. Doses are transcribed and displayed; never computed.
- No `protocol_type` *chosen* for the patient; only inferred-or-unknown.
- No prediction of monitoring dates, trigger time, stimulation length, or retrieval/transfer dates.
- No interpretation of symptoms, ultrasound findings, or lab/hormone values (D2 escalates on pattern-match without grading).
- No reassurance, no diagnosis, no free-generated medical content (Q&A is citation-or-refuse against the bounded KB).
- No autonomous write to the active schedule.

MVP scope-cuts (not permanent boundaries): no clinic dashboard; single anchored protocol family (superovulation/stimulation) for the demo; no cross-protocol generalization beyond `protocol_type` variants that have sourced content.

---

## 7. Traceability to the real card (AVA-Peter)

| Card element (page) | Phase | Type | Notable |
|---|---|---|---|
| Stimulation grid, 4 drug columns, cycle-day rows (p5/8) | 1 | A | Dates hand-filled → prescribed/unscheduled |
| Ultrasound + bloodwork visits (implied, p3 "punctuality") | 1 | Mon | Dates set reactively at prior visit |
| Trigger row: Прегнил/Овитрель/Декапептил, 10000 ЕД (p5/9) | 1→2 hinge | B + alternatives | "Inexact timing may make cycle impossible" |
| Fast 6h; no alcohol/driving 24h; partner present (p6/9) | 2 | E | Procedure-prep; escalate if unmet |
| Luteal: Прогинова/Дивигель, Крайнон/Утрожестан, vit E, folate "until hCG test inclusive" (p6/10–11) | 3 | C + alternatives | until-event(beta_hCG); end date blank |
| "At pain → Кетонал"; "ice on abdomen"; "if spotting → add Прогестерон/Дицинон/Транексам" (p6/10–11) | 3 | D1 | Card-stated; transcription not advice |
| OHSS & red-flag symptoms (general, + any card flags) | 1–3 | D2 | Capture-and-escalate, verbatim redirect |
| Beta-hCG test, date blank (p6/11) | 3 | dated event | Terminates Type C items |

---

## 8. Open question carried into Phase 2

- **KB sourcing/validation stance** remains the one genuine product-risk decision (research report §Recommendations offers a/b/c). It governs what the citation-or-refuse Q&A can answer, so it must be settled before grounded Q&A is built — but it does not block the data schema.
