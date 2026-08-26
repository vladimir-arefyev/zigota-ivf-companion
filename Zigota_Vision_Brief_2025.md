# Zigota — Vision Brief (2025 Rework)

*An AI companion for IVF patients. Rethought from a 2021 mobile-app concept against 2025 tooling.*

**Status:** Draft v0.1 · **Author:** Vladimir Arefyev · **Format:** living document

---

## TL;DR

Zigota began in 2021 as an iOS app to reduce IVF patient anxiety through a medication calendar and an educational section. The job-to-be-done has not changed. What changed is that the three hardest parts of that original app — manual setup, static generic content, and inert data — are exactly the frictions that large language models now remove cheaply.

The 2025 rework is therefore **not a pivot but a change of interface paradigm**: from a UI-first app (tab bar, sections, forms) to a **conversation-first companion on top of a structured backend**. The patient photographs their prescription list; the companion turns it into a living, day-by-day plan, answers "what do I do today?" and "is this normal?" from verified information, and quietly builds the structured protocol record their clinic never had.

The AI is an **interface to structured data and vetted content — never a source of medical judgment.** That boundary is what keeps the product safe, defensible, and out of medical-device territory.

---

## 1. Problem, re-examined for 2025

The original problem statement still holds; three years of hindsight only sharpen it.

An IVF cycle is a weeks-long, high-stakes medical protocol that a patient largely self-administers at home: timed injections, medications keyed to meals, clinic visits, and monitoring — under significant emotional load. The patient's felt problem is **anxiety**, and it has two roots:

- **Loss of control** — "Did I take that at the right time? Have I missed something? What happens today?" The default tools are a paper prescription list, a mix of phone reminders, and memory.
- **Loss of understanding** — "Is this cramping normal or a warning sign? What does this stage even do?" Information is fragmented across forums, clinic leaflets, and search results that often contradict each other.

What is genuinely different in 2025:

- **Setup friction is now optional.** In 2021, giving a patient a plan meant they had to build it — install an app, register, and manually enter every medication and time. Vision-capable models can read a photographed prescription list and propose the schedule, collapsing setup to a single confirmation step.
- **Content can be personal, not generic.** A static "stages of IVF" article is the same for everyone. A retrieval-grounded assistant can answer the patient's *actual* question, in context, against a curated knowledge base.
- **Data no longer has to be inert.** The record a patient generates — adherence, symptoms, questions — can be structured at the point of capture rather than lost, making it useful downstream (see §6).

## 2. The reframe

| | 2021 concept | 2025 rework |
|---|---|---|
| **Paradigm** | UI-first mobile app | Conversation-first companion |
| **Interaction** | Navigate tabs → sections → forms | Ask, or be nudged, in one place |
| **Setup** | Manual profile + manual schedule entry | Photograph prescription list → confirm |
| **Content** | Static educational section | Grounded answers to the actual question |
| **Data** | Stored, then read | Structured at capture, usable downstream |
| **Job-to-be-done** | Reduce anxiety (control + understanding) | *Unchanged* |

The point of the table is the last row. Everything above it is *how*; the *what* is the same product promise made in 2021 — now deliverable with far less friction.

## 3. Who this is for

The original personas still describe the user well: a patient (typically mid-30s), often partnered, either deciding whether to pursue IVF or already mid-protocol with a chosen clinic. Their goals — *have a plan on hand, don't forget anything important, understand what's happening, know what to do if it doesn't work* — and their pains — *side effects, stress on self and partner, fear of doing something wrong, needing reassurance* — map directly onto the two roots of anxiety in §1.

The 2025 companion serves the *mid-protocol* moment most sharply: the patient who has a prescription list in hand and needs it to become a plan they can trust.

## 4. What the companion is

Three capabilities, in priority order.

**1. Onboarding by parsing — the centerpiece.**
The patient photographs their prescription list. The companion extracts each medication, dose, timing, and duration, then **presents every item for the patient to confirm or correct** before anything becomes a schedule. AI proposes; the human ratifies. This is the "send a photo, done" moment — the single biggest UX gain over the 2021 design, and the piece a demo lives or dies on.

**2. The daily plan and grounded Q&A.**
Once the plan exists, the companion answers "what's today?", sends a morning summary of what to take and when, and retrieves past context ("what was my last result for X?"). Questions about the *process* are answered **only from a curated, verified knowledge base** — never free-generated. When the companion doesn't have a grounded answer, it says so and points to the clinic.

**3. Symptom capture — not assessment.**
The patient can log how they feel. The companion **records and structures** the symptom, surfaces it for the clinician, and flags predefined signal patterns for escalation. It never tells the patient whether a symptom is fine or dangerous. Every such interaction carries an explicit "this is informational, not medical advice" and, where appropriate, a prompt to contact the clinic.

## 5. What the companion is *not* — the hard boundary

This is the most important section in the document.

**The AI is an interface to structured data and vetted content. It is never a source of medical judgment.**

Concretely, the companion **does not**:

- recommend, adjust, or reason about dosages;
- interpret a symptom or lab value, or reassure ("you're fine");
- diagnose, or imply a diagnosis;
- generate medical information freely — process answers come only from the curated knowledge base;
- act autonomously on the parsed prescription — the patient confirms every item.

Two reasons this boundary is load-bearing:

- **Safety.** IVF is a high-consequence, self-administered protocol. A hallucinated dose or a false "that's normal" carries real harm. Constraining the AI to *structured data in, vetted content out* removes the class of errors that matters most.
- **Regulatory posture.** By confining itself to reminders, administrative scheduling, information delivery, and structured capture — and explicitly *not* interpreting or advising — the product stays on the wellness / patient-support side of the line rather than becoming a regulated medical device (SaMD). This is a design decision, made deliberately and up front, not a disclaimer bolted on afterward.

## 6. Why it matters beyond the patient — the clinic layer

The patient-facing companion is the product. But its by-product is what makes Zigota more than a nice utility.

Today, clinics hand out paper prescription lists, receive no feedback on whether medications were actually taken, and store unstructured histories that are hard to analyze. The 2021 pitch already identified this: clinics and pharma want **standardized, digitized protocols and structured adherence data**, and patients generating that data as a side-effect of getting help is the mechanism.

Because the 2025 companion structures data *at the point of capture* — a confirmed schedule, adherence marks, logged symptoms, questions asked — it produces exactly the clean protocol-adherence record clinics never had. That record is the foundation of the eventual business (clinic dashboards, protocol standardization, de-identified data for research) and the reason the companion is a platform, not a toy.

*Scope note:* the clinic-facing surface is **not** part of the first MVP. It is stated here as the "why this matters" layer so the vision reads as a business. The MVP demonstrates the patient experience; the data it structures is what the business is later built on.

## 7. MVP scope

**In scope (patient-facing, demonstrable):**

- Prescription-list parsing → item-by-item patient confirmation → schedule.
- Daily plan view and "what's today?" conversational query.
- Proactive daily summary / reminders.
- Grounded Q&A over a curated IVF knowledge base, with honest "I don't know — ask your clinic" fallback.
- Symptom logging with capture-and-escalate behavior and consistent "not medical advice" framing.

**Explicitly out of scope for the MVP:**

- Any clinic-facing dashboard or data-sharing surface (§6 — roadmap).
- Any dosage, symptom, or lab interpretation (§5 — permanent boundary, not a scope cut).
- Real-time clinician chat / telemedicine.

**Protocol handling:**
The vision is **protocol-agnostic** — the companion adapts to whatever a given prescription list contains, rather than hard-coding a single treatment path. For the build and demo, we anchor on **one representative stimulation protocol as the concrete worked example**: a controlled ovarian **superovulation-stimulation** cycle, drawn from a real clinic patient card (AVA-Peter, "Стимуляция суперовуляции"). It spans the three phases every stimulation cycle shares — daily stimulation, a precisely-timed trigger plus puncture, and post-transfer luteal support — which makes it a faithful stand-in for the general case rather than a narrow special case. Other protocols and clinic formats are roadmap. See the worked-example appendix.

**Surface:**
A self-contained web application. The conversational UX lives inside an interface fully under our control — simpler to build, show, and reason about than a third-party messenger bot, with no external platform approvals in the critical path.

## 8. Open questions to resolve next

- ~~Which single protocol becomes the demo instance?~~ **Decided** — the superovulation-stimulation cycle from the AVA-Peter patient card (see appendix). Remaining: confirm the card covers every state the parse-and-schedule flow must handle, and decide how blank fields (dates a clinician fills in by hand) are represented.
- What is the minimum viable curated knowledge base — sourcing, scope, and who validates it?
- Where is the parse-confirmation UX drawn so it feels effortless but never lets an unconfirmed item through — especially for the trigger, where exact timing is safety-critical?
- What is the data model that keeps capture "structured enough" to matter for §6 without over-engineering the MVP?

---

*This document is a living brief. It is intended to anchor the build and to serve as the first artifact of the rework's product narrative — showing not just what was built, but how a 2021 concept was re-evaluated against 2025 technology and constraints.*

---

## Appendix A — Worked example: the anchored protocol

The MVP is built and demoed against one concrete, real-world instance: a controlled ovarian **superovulation-stimulation** cycle, taken from a clinic patient card ("Стимуляция суперовуляции", AVA-Peter). It is not a hard-coded path — it is the first prescription list the companion is proven to parse and schedule end to end. It is chosen because it exercises every hard part of the general problem in a single document.

**Why this instance is representative.** Every stimulation cycle, regardless of clinic or drug brand, moves through the same three phases. This card contains all three, so modeling it convincingly covers the shape of the general case:

1. **Daily stimulation.** A procedural grid of up to four gonadotropin preparations (e.g. Menopur / Gonal / Puregon-class), dosed once or twice daily over roughly one to two weeks, tracked by cycle day. *Exercises: multi-drug daily schedules, dose parsing, meal- and time-of-day anchoring, day-by-day adherence capture.*
2. **Trigger and puncture.** A single **trigger injection** (Pregnyl 10 000 IU / Ovitrelle 250 mcg / a GnRH-agonist such as Decapeptyl or Diferelin) given at an **exact time**, followed by egg retrieval ~34–36 hours later, with a fasting-before rule and a no-alcohol / no-driving rule for 24 hours after. *Exercises: the safety-critical single-event reminder where timing precision matters most, and non-medication instructions the companion must surface but not interpret.*
3. **Post-transfer luteal support.** A regimen of progesterone and estrogen support (Progynova, Divigel, Crinone gel, Utrozhestan, plus vitamin E and folic acid) with different routes (oral, vaginal, intramuscular) and meal anchors, continued until the pregnancy test, with conditional add-on medications (Progesterone, Dicynone, Tranexam) triggered by specific symptoms. *Exercises: multi-route regimens, "continue until date/event" durations, and conditional/as-needed items.*

**What it forces the MVP to handle well:**

- Extracting structured medication items (name, dose, unit, route, frequency, meal/time anchor, duration) from a semi-structured clinic document — including brand-name variability and hand-filled blanks.
- A confirmation step that is effortless for routine daily meds yet unmistakably deliberate for the trigger, where an unconfirmed or mis-timed item is the highest-consequence error in the whole cycle.
- A schedule model expressive enough for: fixed daily doses, a single exactly-timed event, "until a date or event" durations, and conditional as-needed items — without becoming a general-purpose medical scheduler.
- Non-medication instructions (fasting, activity restrictions, "call the clinic if…") that are surfaced and escalated, never assessed.

**What it deliberately leaves out (roadmap):** other stimulation protocols (long / short / antagonist variants), other clinics' card formats, and any cross-protocol generalization. The vision stays protocol-agnostic; the *proof* starts with one protocol done properly.

*Note: the patient card is source material for modeling the data and flow. Drug names and doses appear here only to characterize the structure the parser must handle; nothing in this document or the product constitutes medical guidance (see §5).*
