# Zigota — Requirements (Phase 1)

*Domain & requirements for the MVP: job stories, acceptance criteria, and explicit non-requirements, grounded in the real protocol and the non-SaMD boundary.*

**Status:** Draft v0.1 · **Author:** Vladimir Arefyev · **Companion docs:** `Zigota_Vision_Brief_2025.md`, `Zigota_Build_and_Content_Plan.md`, `Zigota_Architecture_2025.md`

---

## How to read this document

Requirements are decomposed top-down into blocks of functionality. Each block holds **job stories** in the form *when [situation], I want [motivation], so I can [outcome]*. Acceptance criteria and explicit non-requirements are added per block, incrementally, after the story set is agreed.

A **glossary** and the **settled decisions** that constrain every block are at the end of the document. Read them first if a term like *anchor*, *offset*, or *lead* is unfamiliar.

The blocks:

- **Block 0 — Profile & preferences** — patient-authored routine anchors and reminder settings.
- **Block 1 — Onboarding** — parse the prescription card and confirm it into a schedule.
- **Block 2 — Tracking** — daily plan, reminders, progress, symptom capture, emergency access.
- **Block 3 — Education** — grounded Q&A over vetted material.
- **Block 4 — Foundation** — cross-cutting constraints (access, storage, safety, integrity). Written as constraints, not job stories.
- **Block 5 — Data retention & traceability** — what is stored, how long, provenance linkage, and deletion. Written as requirements, not job stories.

---

## Block 0 — Profile & preferences

Patient-authored context. Not prescribed, not clinical. An anchor is one of the five fixed routine times; a prescribed item stores its timing as `anchor + offset` (e.g. "breakfast − 30min"), and tracking resolves that to a clock time at read. Setting anchors is what makes resolution possible.

**0.1** When I start using the app, I want to set my routine anchor times from a fixed list (wake, breakfast, lunch, dinner, bedtime), so my anchor-relative doses can be turned into real times later.

*Acceptance criteria — positive*
- The patient can set a time for each of the five fixed anchors: wake, breakfast, lunch, dinner, bedtime.
- An anchor can be left unset; unset is treated as a known gap, surfaced later in tracking, not filled with a guess.
- A set anchor is available to tracking immediately, for resolving not-yet-confirmed items that depend on it.

*Acceptance criteria — negative*
- The app does not invent or pre-fill any anchor time (no "breakfast = 08:00" the patient didn't enter).
- The patient cannot create an anchor outside the fixed five.
- Setting an anchor does not by itself create any schedule item.

**0.2** When I need time to prepare a dose (retrieve an injection from the fridge, let it warm, assemble a pen), I want reminders to arrive a set time before the dose is due, so I'm ready when the moment comes rather than starting from scratch.

*Acceptance criteria — positive*
- A global lead time determines how far before an event's resolved time its push fires. The default is 5 minutes.
- The patient can change the lead time; the new value applies to future reminders as soon as it is saved.
- Lead time applies uniformly to reminders across the schedule.

*Acceptance criteria — negative*
- Lead time changes only *when* a reminder fires, never the event's own resolved time — the dashboard and confirmation record always use `anchor + offset`, not the lead-shifted time.
- Lead time changes only *when* a reminder fires, never whether a prescribed item exists or its stored `anchor + offset`.
- The app does not silently vary lead time per item or item type; a single global value applies until the patient changes it.

**0.3** When my routine changes, I want to edit an anchor time, so my not-yet-confirmed doses re-time to match — without touching anything I've already confirmed or missed.

*Acceptance criteria — positive*
- Editing an anchor time re-times, at next read, every not-yet-confirmed item that depends on that anchor.
- Re-timing preserves each item's stored offset relative to the new anchor time (an item at "breakfast − 30min" stays 30 minutes before, at the new breakfast time).
- The patient can see which upcoming items moved as a result of the edit.

*Acceptance criteria — negative*
- Editing an anchor does not alter any confirmed item; its frozen confirmation timestamp and expected time are untouched.
- Editing an anchor does not alter any missed or skipped item; these are terminal, stay recorded as "not confirmed," and are never re-timed into the future.
- Editing an anchor never changes an exactly-timed prescribed event (e.g. the trigger); profile anchors fill routine timing, they never override a clinic-specified exact time.

---

## Block 1 — Onboarding (parse-and-confirm)

The highest-risk, highest-value flow. Upload the card, map it to the protocol model, confirm item-by-item into an approved schedule.

**The item-type model.** Every extracted item is one of these types (from the protocol model, grounded in the real AVA-Peter card). Blocks 1 and 2 both refer to this set:

- **A — fixed daily dose** over a window (stimulation gonadotropins, suppression agonist/antagonist, most luteal drugs). Recurring, anchor-relative.
- **B — single exactly-timed event** (the trigger injection). One exact moment; mistiming can end the cycle. Its own deliberate confirmation gate (1.6).
- **Mon — monitoring / appointment** (ultrasound + bloodwork visits, retrieval, transfer, beta-hCG test). Almost always prescribed-unscheduled: the clinic sets the next date reactively at the prior visit.
- **C — continue-until-date/event course** (luteal support "until the beta-hCG test, inclusive"). A daily item whose end is an event not yet dated at parse time.
- **D1 — card-stated comfort conditional** ("if pain → NSAID"; "ice on abdomen day of puncture"). An *if X then Y* rule written on the card. Surfacing it is transcription, not advice; the app never asserts X has occurred.
- **D2 — warning-sign escalation** (OHSS and other red flags). A safety net, not a card convenience. Present for *every* cycle from the KB, regardless of what the card lists. Behaviour: capture-and-escalate via the KB (see 2.6), never grade or interpret.
- **E — non-medication instruction** (fast before retrieval, no driving 24h after sedation, partner abstinence). Surfaced as procedure-prep; escalated if reported unmet, never assessed.

**Cross-cutting: alternatives.** One prescription slot can offer mutually exclusive fills (Progynova *or* Divigel; trigger = Pregnyl *or* Ovitrelle *or* triptorelin). The parse must not pick one; the patient selects which she was actually given, from the card's options (1.7).

**1.1** When I get a prescription card from my clinic, I want to upload a photo of it, so I don't have to type the protocol in by hand.

*Acceptance criteria — positive*
- The patient can upload an existing image of the card.
- A successful upload moves the patient into extraction.

*Acceptance criteria — negative*
- The app does not begin building a schedule from an image until extraction and confirmation have run — an uploaded image is not a schedule.
- The app does not offer in-app camera capture in MVP; upload of an existing image is the only path.

**1.2** When the image I uploaded can't be read at all, I want the app to tell me and let me upload another, so extraction works from a usable image instead of failing quietly.

*Acceptance criteria — positive*
- When extraction returns no items at all, the app tells the patient the image couldn't be read and offers re-upload.
- The patient can re-upload without losing items already confirmed in this session.

*Acceptance criteria — negative*
- The app does not proceed to confirmation when extraction returned nothing.
- The app does not guess or fabricate content from an image it could not extract from.
- No confidence threshold gates this: the criterion is zero items returned, not a partial-read score. Any image returning at least one item proceeds to confirmation, with gaps handled by manual entry (1.4).

**1.3** When the app has read my card, I want each item mapped to a clear type and timing, so what gets tracked matches what was prescribed.

*Acceptance criteria — positive*
- Each extracted item is mapped to one of the item types A, B, Mon, C, D1, E (see the item-type model above). D2 warning signs are not parsed from the card — they come from the KB for every cycle.
- Routine-relative timing is mapped to `anchor + offset` from the fixed anchor vocabulary.
- An item the clinic leaves undated (a Mon visit, the trigger, a C-course end) is recorded as *prescribed-unscheduled*: a valid first-class state, visible but not yet reminding, awaiting a clinic-set date (see 2.3).
- Items land in the *prescribed* layer, not the schedule.

*Acceptance criteria — negative*
- An extracted item that cannot be assigned to a type is flagged incomplete, not assigned a type by guess.
- An item missing the timing its type requires is flagged incomplete.
- Timing that cannot be mapped to the fixed anchor vocabulary is flagged, never snapped to the nearest anchor.
- A prescribed-unscheduled item is not given a guessed date; the parse never invents a clinic date.
- A flagged-incomplete item cannot be confirmed until the patient completes the missing field.
- Mapping writes nothing to the *scheduled* layer.

**1.4** When the app couldn't read some items off my card, I want to add or correct those items myself by typing or by voice, so a partial parse doesn't leave holes in my schedule.

*Acceptance criteria — positive*
- The patient can add an item the parser missed, or correct a mis-parsed one, by typing or by voice.
- A manually entered item is marked patient-entered, distinct from clinic-sourced.
- Manually entered items go through the same per-item confirmation (1.5) as parsed items before they count.

*Acceptance criteria — negative*
- The app does not treat a patient-entered item as clinic-sourced.
- The app does not auto-fill the clinical content of a manual item; it captures what the patient states and does not suggest a drug, dose, or timing the patient didn't give.

**1.5** When the app shows me what it extracted, I want to confirm each item one at a time before it counts, so nothing I didn't approve ends up in my schedule.

*Acceptance criteria — positive*
- Each item is presented for the patient to confirm individually before it enters the schedule.
- On confirmation, the item transitions from *prescribed* to *scheduled* and becomes trackable.
- The patient can edit an item's details at the point of confirmation.

*Acceptance criteria — negative*
- No item enters the schedule without explicit per-item confirmation; there is no bulk "confirm all" that approves items without the patient acting on each one.
- A flagged-incomplete item cannot be confirmed until its missing field is resolved.
- The AI cannot confirm an item on the patient's behalf; confirmation is a patient action only.

**1.6** When an item is the trigger injection, I want to confirm it deliberately and separately, so the one time-critical event can't be waved through like a daily vitamin.

*Acceptance criteria — positive*
- The trigger injection is presented as a distinct, deliberate confirmation step, separate from routine items.
- Its exact prescribed time is preserved and shown; the patient confirms against that exact time.
- Confirmation records the trigger as a single exactly-timed event.

*Acceptance criteria — negative*
- The trigger cannot be confirmed as part of a batch or swept in with routine items.
- A profile anchor never resolves or shifts the trigger's time; it is exactly-timed, not anchor-relative.
- *(Trigger reminder behaviour is specified in Block 2; onboarding owns the separate confirmation only.)*

**1.7** When my card offers a choice of drugs for one slot (this *or* that), I want to pick the one I was actually given before it counts, so my schedule reflects what I'm really taking and not a guess.

*Acceptance criteria — positive*
- An item with alternatives presents the card's options, and the patient selects exactly one.
- The item cannot leave the proposed state until the patient selects; selection is a hard confirmation gate, like the trigger gate.
- The confirmed item records the selected drug.

*Acceptance criteria — negative*
- The parse never silently picks one of the alternatives.
- Selection is limited to the options the card lists; this gate offers no free-text alternative (a drug not on the card is a manual entry via 1.4, marked patient-entered).
- An item with unresolved alternatives is never persisted to the schedule.

**1.8** When I'd rather not type, I want to give my input by voice, so entering or correcting items is easier on a phone.

*Acceptance criteria — positive*
- Where input is invited (manual item entry, corrections), the patient can speak instead of type, and the transcript is captured.
- The patient sees the transcribed text and can correct it before it is used.

*Acceptance criteria — negative*
- The app does not act on a voice transcript it has not shown back to the patient for confirmation.
- Voice transcription does not itself confirm any item; it feeds 1.4/1.5, which still require explicit per-item confirmation.

**1.9** When I've confirmed my items, I want them saved as my schedule, so the app can track the cycle from a source I approved.

*Acceptance criteria — positive*
- Confirmed items are persisted as the patient's schedule, available to tracking.
- The saved schedule is the source of truth tracking reads from.

*Acceptance criteria — negative*
- Unconfirmed, flagged-incomplete, or unresolved-alternative items are not persisted into the schedule.
- Saving does not freeze resolved clock times; pending items persist as `anchor + offset`, resolved at read.

---

## Block 2 — Tracking

Daily plan, reminders, progress, confirmation, symptom capture with KB warning-sign escalation, health check-in, non-medication procedure-prep, emergency access, and chat plan queries. Deterministic tracker core plus the Tracker Agent.

**2.1** When I open the app on a given day, I want to see what's due today, so I know what to do without reading the whole protocol.

*Acceptance criteria — positive*
- The dashboard's today's-plan region shows the items due today, each with its resolved event time (`anchor + offset` against the current profile) and its state (pending, confirmed, missed).
- The plan reflects confirmed schedule items only.

*Acceptance criteria — negative*
- An item whose anchor is unset is not shown with a real time in the plan; it appears in the actions-needed region instead (see 2.3).
- The dashboard does not display AI-generated commentary or interpretation of the plan; it is a read-only view of structured state.

**2.2** When a dose is coming up, I want a reminder at the right time for my routine, so I don't miss it.

*Acceptance criteria — positive*
- For each pending item, a push fires at its resolved event time minus the global lead time.
- The reminder identifies the item and its due time.
- The trigger reminds by this same mechanism; global lead applies.

*Acceptance criteria — negative*
- No reminder fires for an item whose anchor is unset; the gap is surfaced in actions-needed, not fired blindly.
- No reminder fires for an item in a terminal state (confirmed, missed, skipped).
- The reminder does not assess or comment on the dose; it states what and when.
- Delivery is subject to browser/OS conditions (permission, service worker, platform); reliable delivery is not guaranteed (see Known MVP limitations).

**2.3** When an item has no usable time yet, I want the app to surface it and help me resolve it, so it's a visible gap and not a silent missed dose. There are two kinds of gap, resolved differently.

*Acceptance criteria — positive (unset-anchor gap)*
- An item depending on an unset anchor appears in the dashboard's actions-needed region, visibly distinct from a timed item.
- The patient can set the missing anchor inline (in chat), which resolves that item and every other pending item depending on the same anchor.
- Once resolved, the item takes its place in the plan and becomes reminder-eligible.

*Acceptance criteria — positive (prescribed-unscheduled: clinic date not yet set)*
- An item the clinic leaves undated (a Mon visit, the trigger, a C-course end) stays *prescribed-unscheduled*: visible but dormant, no reminder.
- The app prompts the patient for the clinic-set date using the same app-initiated chat pattern as the health check-in (2.8) — "has your clinic given you the date for X?" — and the patient enters it when they have it.
- Once a date is entered, the item becomes scheduled and reminder-eligible.

*Acceptance criteria — negative*
- Neither gap kind ever silently acquires a guessed time or date; the app never invents a clinic date or an anchor.
- A gap item is never counted as missed for lack of a time — it is a visible gap, not a missed dose.
- A prescribed-unscheduled item fires no reminder until its date exists.

**2.4** When I take a dose, I want to confirm it, so my progress reflects what I've actually done.

*Acceptance criteria — positive*
- The patient can confirm a pending item as taken, from the chat.
- Confirmation records *that* the patient confirmed and the confirmation timestamp (moment of the tap).
- On confirmation the item becomes terminal (confirmed): no longer re-timed by anchor edits, no longer reminder-eligible.
- The patient can later correct a confirmation (un-confirm, or correct a missed item to confirmed); every correction is recorded in the item's change log.

*Acceptance criteria — negative*
- Confirmation does not record or imply an exact administration time; the timestamp is the tap, not a measured intake.
- The AI cannot confirm on the patient's behalf; confirmation is a patient action.
- No automatic process un-confirms or alters a confirmed item; only a logged patient correction can.

**2.5** When I'm partway through the cycle, I want to see my progress across the three phases, so I have a sense of where I am and control over it.

*Acceptance criteria — positive*
- The dashboard's progress region shows position across the three cycle phases (stimulation → trigger + puncture → post-transfer).
- Progress reflects confirmed and missed items against the scheduled plan.

*Acceptance criteria — negative*
- The app does not compute or display any clinical assessment of progress (no "on track," "responding well," no interpretation) — it shows states and counts, not judgment.
- Missed items are shown as missed, not hidden or back-filled.

**2.6** When something feels off physically, I want to record the symptom and be pointed to my clinic if it's something the vetted information flags, so nothing serious goes unnoticed — without the app telling me what my symptom means.

*Acceptance criteria — positive*
- The patient can record a symptom in the chat, by text or voice; it is captured verbatim as patient-reported, timestamped, and stored for the patient and clinic.
- On capture, the companion checks the symptom against the KB's warning-sign (D2) content, which is present for every cycle regardless of card.
- On a match, the app surfaces the relevant KB content *with citation* and the clinic-contact path (2.7), framed as information — "our information lists this as something to raise with your clinic" — not as a verdict about the patient.
- The clinic-contact path is available on every symptom report, independently of whether anything matched.

*Acceptance criteria — negative*
- The app does not assess, diagnose, triage, or rate the severity of a symptom; a KB match surfaces cited information and the contact path, never a judgment that *this* symptom is dangerous.
- The app does not suggest a cause or a treatment.
- On no match, the app captures silently and says nothing reassuring — a non-match never implies "you're fine" (N3). The match is an additional prompt toward escalation, never a gate on it.

**2.7** When something feels seriously wrong and I'm frightened, I want fast access to my clinic's contact numbers and any emergency instructions they've given, so I can reach a human who can help instead of asking an app to judge how bad it is.

*Acceptance criteria — positive*
- The clinic's contact numbers and any clinic-provided emergency instructions are reachable quickly and always available.
- The information is presented plainly.

*Acceptance criteria — negative*
- The app does not actively notify the clinic (passive for MVP).
- The app does not assess urgency or decide whether the patient should escalate.
- The app does not gate emergency contact info behind a symptom assessment or any interpretation step.

**2.8** When the app checks in on how I'm feeling, I want to answer in my own words (overall condition, mood, any symptoms), so my experience is captured over time without me having to remember to report it.

*Acceptance criteria — positive*
- The app initiates an optional health check-in once daily, in the chat.
- The patient can answer in their own words, by text or voice, covering overall condition, mood, and any symptoms.
- Responses are captured verbatim as patient-reported, timestamped, and stored for the patient and clinic.
- Answering is optional; a skipped check-in is not treated as a missed obligation.

*Acceptance criteria — negative*
- The app does not assess, score, or interpret the check-in response.
- The app does not diagnose or advise based on what the patient reports.
- If the check-in surfaces a symptom, it runs the same KB warning-sign check as 2.6 (cited information plus contact path on a match; silent capture with no reassurance on no match). The app never judges urgency itself.

**2.9** When I ask in chat what my plan is or what's left today, I want the app to tell me from my confirmed schedule, so I can check without switching to the dashboard.

*Acceptance criteria — positive*
- On request in chat, the app reports today's plan from the confirmed schedule.
- "What's left" returns the pending items for today.
- The answer is a retrieval of structured schedule state, resolved at read.

*Acceptance criteria — negative*
- The answer does not add advice, interpretation, or generated content beyond the schedule state.
- The answer reflects the same confirmed-state source of truth as the dashboard; it does not invent or differ from it.

**2.10** When my protocol has a non-medication instruction (fast before retrieval, no driving after sedation, abstinence before the sample), I want it surfaced as procedure-prep and to be pointed to my clinic if I report I couldn't follow it, so I don't derail a procedure by missing a rule.

*Acceptance criteria — positive*
- Type E instructions are surfaced and reminded as procedure-prep, tied to their phase-2 events.
- If the patient reports an instruction unmet (e.g. ate before a fasting-required retrieval), the app surfaces the clinic-contact path (2.7).

*Acceptance criteria — negative*
- The app does not assess the consequence of an unmet instruction or decide whether the procedure can proceed; it routes to the clinic.
- The app does not reassure that an unmet instruction is fine.

---

## Block 3 — Education (grounded Q&A)

Answers about the protocol, steps, procedures, drugs, and side effects — grounded only in vetted material, with an honest fallback.

**3.1** When I don't understand a step, drug, or procedure in my protocol, I want to ask and get an answer grounded in vetted material, so I can understand what's happening without trawling the internet.

*Acceptance criteria — positive*
- The patient can ask about a step, drug, procedure, or side effect and get an answer drawn only from the curated KB.
- Every answer is grounded in retrieved KB content and shows its source citation to the patient.
- Answers are framed as general patient education, consistently, regardless of how the question is phrased.

*Acceptance criteria — negative*
- The app does not generate a medical answer from model knowledge when retrieval returns no relevant KB content; absence of a source produces the fallback (3.2), not an invented answer.
- The app does not interpret the patient's own situation, values, or symptoms — general information only, never applied as a judgment about this patient.
- The app does not give dosing advice, diagnose, or recommend a course of action, even when the KB contains related factual content.

**3.2** When I ask something outside what the app can safely answer, I want it to tell me it doesn't know and point me to my clinic, so I'm never handed a confident guess.

*Acceptance criteria — positive*
- Scope is defined operationally: the app answers when KB retrieval returns relevant content, and falls back when it does not.
- When a question falls outside KB scope, the app says plainly that it doesn't have that information and directs the patient to their clinic.
- The fallback is unambiguous — it does not hedge into a partial guess.

*Acceptance criteria — negative*
- The app does not fabricate a source or an answer to avoid saying "I don't know."
- The app does not answer a partially-in-scope question by supplementing the KB with generated content; it answers only the in-scope part from the KB, or falls back.
- The fallback and any disclaimer are unconditional and survive a language switch.

**3.3** When I ask about my own schedule ("why am I taking this?"), I want the answer to reference my confirmed items, so it's about my protocol, not a generic one.

*Acceptance criteria — positive*
- When the patient asks about their own protocol, the app uses their confirmed schedule to resolve *which* item or drug the question refers to, then answers from the KB about that item, with citation.
- The schedule is used for subject identification only; the explanatory content comes from the KB.

*Acceptance criteria — negative*
- Using the schedule for context does not license interpreting the patient's specific regimen ("your dose is correct/high/low," "you're at the right stage") — it identifies the subject, the KB supplies general information.
- The app does not infer anything clinical from the combination of the patient's schedule and a KB fact; no synthesis into personalized medical judgment.

---

## Block 4 — Foundation (cross-cutting constraints)

Not job stories — no patient "wants" these in a situation. Written as inherited constraints with acceptance criteria where testable. These hold across every other block.

**4.1 — Access & data isolation**

- Authentication is via Google account (OAuth) for MVP. Identity is real and cross-device.
- Each authenticated identity sees only its own data; one patient's records are never visible to another.
- *AC — positive:* a signed-in patient can reach only their own profile, schedule, symptoms, and check-ins. *AC — negative:* no request path returns another identity's data; there is no shared or unauthenticated access to patient records.

**4.2 — Source-of-truth storage & the provenance envelope (enforcement lens)**

- Firestore is the single source of truth. Confirmed schedule state lives there and nothing downstream overrides it.
- Every AI-derived record carries a provenance envelope: status, source, raw extract, model version, edit history. This is the mechanism that makes the non-SaMD boundary demonstrable rather than asserted (the traceability/lifecycle lens on the same envelope is Block 5).
- The Confirmation Gate is the sole writer of confirmed records; no other component and no AI path writes to the scheduled layer.
- *AC — positive:* every confirmed item is traceable to its source via the envelope; the Confirmation Gate is the only writer of confirmed state. *AC — negative:* no AI component writes a confirmed record; no confirmed record exists without a provenance envelope.
- Data is stored in a single GCP region, which defines its geographic residency. Encryption is GCP default (at rest and in transit); no application-level encryption beyond GCP defaults in MVP.

**4.3 — Non-SaMD enforcement by architecture**

- The boundary (non-requirements N1–N8) is enforced structurally, not by prompt wording: retrieval-or-refuse on the Education surface, citation-or-refuse as a hard gate, confirmation as the only state transition into the schedule, AI never the writer of confirmed records.
- *AC — positive:* the safety negatives hold under adversarial testing (deliberate attempts to elicit dosage reasoning, symptom interpretation, reassurance, or an uncited answer are refused). *AC — negative:* no prompt-only guardrail is the sole line of defence for any N1–N8 boundary; each is backed by a structural mechanism.

**4.4 — Disclaimer & language-switch integrity**

- Disclaimers are unconditional — present regardless of question type, phrasing, or conversation state.
- Safety framing and disclaimers survive a language switch; they do not drop or revert when the conversation changes language (per the resolved multilingual-disclaimer bug).
- *AC — positive:* the disclaimer is present across languages and conversation states. *AC — negative:* no language switch or phrasing causes a disclaimer or refusal to be omitted.

---

## Block 5 — Data retention & traceability

Not job stories — the data-lifecycle requirement set. The provenance envelope seen through the traceability and lifecycle lens (its enforcement role is 4.2).

**5.1 — Provenance linkage & raw-capture retention**

- Every derived item is traceable to its origin: the raw capture it came from (uploaded image or voice transcript), whether it is clinic-sourced or patient-entered, and the model version that produced it.
- A raw capture is stored and retained as long as any item derived from it exists. It is removed only when those items are removed (see 5.3).
- *AC — positive:* from any confirmed item, its raw capture and source classification are recoverable. *AC — negative:* a derived item never loses its link to its origin while it exists; a patient-entered item is never recorded as clinic-sourced.

**5.2 — Retention from the patient's perspective**

- From the patient and app perspective, data persists indefinitely: there is no self-service deletion via the UI or chat. The patient's full history stays available to them.
- *AC — positive:* a patient can view their complete history for the life of the account. *AC — negative:* no UI or chat action deletes patient data.

**5.3 — Deletion (administrative)**

- Deletion is available on user request, executed by an administrator through an out-of-band channel (not the app UI or chat).
- Deletion removes the patient's data, including the raw captures whose derived items are removed.
- *AC — positive:* an administrator can fully delete a patient's data on request. *AC — negative:* no automatic or in-app process performs deletion; it is always an administrative action on request.

**5.4 — Archival**

- A completed cycle becomes eligible for archival one year after cycle completion (one year after the last scheduled item's date). Archival is administrator-initiated, not automatic — the year marks eligibility, not a trigger.
- Archival is non-destructive and backend-only: archived data still exists, moved out of the active set; it has no user-facing effect, and the patient's view of their history is unchanged.
- *AC — positive:* an administrator can archive a cycle once it is eligible; archived data is preserved. *AC — negative:* archival never fires automatically; it never deletes data; it does not remove anything from the patient's view.

---

## Non-requirements — the hard boundary as testable negatives

The §5 boundary from the Vision Brief, restated as system-level "must not" statements so each is testable. N1–N6 are the Vision Brief's boundary verbatim; N7–N8 are added from Phase 1 decisions. These are consolidated here; individual block AC enforce them at the point of use.

The system must not:

- **N1 — Reason about dosage.** Recommend, adjust, calculate, or reason about a dose — not at parse, not in Q&A, not in a reminder.
- **N2 — Interpret symptoms or values.** Assess, triage, rate, or explain the significance of a patient-reported symptom or any lab/measurement value.
- **N3 — Reassure.** Tell a patient they are fine, that something is normal, or that they need not worry.
- **N4 — Diagnose.** State or imply a diagnosis, or a probability of one.
- **N5 — Generate medical content freely.** Answer a clinical or process question from model knowledge; such answers come only from the curated KB, with citation, or fall back.
- **N6 — Act autonomously on the parse.** Enter any parsed item into the schedule without explicit per-item patient confirmation.
- **N7 — Assess urgency or escalate on the patient's behalf.** Decide whether a situation is an emergency; the app surfaces the contact path, the patient decides.
- **N8 — Present derived timing or state as clinical truth.** Show a lead-shifted reminder time as the dose time, or a confirmation timestamp as a measured administration time.

---

## Non-functional requirements

Two columns per attribute: the **MVP posture** (what the demo-grade product actually is and does) and the **production bar** (what a real-user version would have to meet). The split is deliberate — it states honestly where the MVP stands without faking production numbers, and marks the distance to a real deployment. Safety is the one attribute where the MVP posture *is* the production bar: it is enforced structurally, not deferred.

| Attribute | MVP posture | Production bar |
|---|---|---|
| **Performance** | Interactive latency good enough for a cold walkthrough; parse/extraction may take seconds and that is acceptable. No latency targets. | Defined p95 targets for chat response and parse; perceived-latency handling (streaming on the Education surface, the deferred capability). |
| **Reliability** | State transitions are correct and durable (confirmation, missed-rollover, re-resolution behave exactly as specified); best-effort push delivery, no delivery guarantee. | Guaranteed/retried reminder delivery; idempotent writes; defined error budgets on the state machine. |
| **Availability** | Single-region, single Cloud Run deployment; downtime acceptable for a demo. No SLA. | Multi-region or failover posture; a stated uptime SLA; health checks and alerting. |
| **Maintainability & safety** | Modular monolith with clear component boundaries; the non-SaMD boundary enforced by architecture (Confirmation Gate as sole writer, retrieval-or-refuse, provenance envelope) — **the safety posture is production-grade, not deferred**. | Same safety architecture, plus operational hardening: audit review, change control on the KB, model-version governance. |
| **Scalability** | Single-patient demo scope; no concurrency or multi-tenant load design. | Multi-tenant data isolation at scale; horizontal scale on the stateless tiers; KB retrieval that holds under load. |
| **Testability** | The Phase 1 acceptance criteria and non-requirements are the test guardrails; safety negatives are adversarially testable (the assistant builds were already run against content-fidelity, scope/refusal, and injection suites). | Automated regression on the full AC set; continuous adversarial testing of the boundary; coverage targets. |

The through-line: for a portfolio MVP, most NFRs are honestly demo-grade — but **safety is not on that curve**, because the whole product thesis is that the boundary is a structural property, provable now, not a number to hit later.

---

## Known MVP limitations

Stated plainly, as credibility assets rather than omissions. To grow as the build surfaces more.

- **Push delivery is best-effort.** Web push via FCM depends on notification permission being granted, the service worker being active, and the browser/OS permitting delivery. Mobile browsers — iOS Safari in particular — deliver web push unreliably. A reminder is not a guarantee the patient was notified.
- **Warning-sign matching is bounded by the KB, and is a prompt, not a safety guarantee.** The D2 check surfaces an escalation prompt only when a reported symptom matches KB warning-sign content. It can miss a real red flag the patient phrased in words the KB doesn't match (a false negative). It is deliberately an additional nudge on top of an always-available clinic-contact path, never the patient's only route to escalation — but it must not be presented, to a patient or a stakeholder, as reliable symptom triage. It is not triage.

---

## Glossary

- **Anchor** — one of the five fixed routine times the patient sets in their profile: wake, breakfast, lunch, dinner, bedtime. The vocabulary is fixed; patient-named anchors are out of scope for MVP.
- **Offset** — a fixed time shift stored with a prescribed item, relative to its anchor (e.g. "breakfast − 30min"). Comes from the card. Together, `anchor + offset` define the item's true prescribed event time.
- **Event time** — the clock time an item is actually due, resolved at read from `anchor + offset` against the current profile. Not stored as a frozen value while the item is pending.
- **Lead time** — how far *before* an event time a reminder push fires. A patient-set global value (default 5 min). Shifts the reminder only, never the event time. Distinct from offset: offset moves the event, lead moves the reminder.
- **Prescribed layer** — items as extracted or entered, before confirmation. Not yet trackable.
- **Scheduled layer** — items after per-item patient confirmation. The source of truth tracking reads from.
- **Confirmation** — the patient action that transitions an item from prescribed to scheduled. Records *that* the patient confirmed and *when they tapped*, never a claimed exact administration time.
- **Trigger (injection)** — the single exactly-timed event whose mistiming can end the cycle. Confirmed separately at onboarding; exactly-timed, never anchor-relative.
- **Not-yet-confirmed / pending** — the only mutable state; re-resolves when its anchor is edited.
- **Terminal states** — confirmed, missed, or skipped. Frozen against automatic change (never re-timed by an anchor edit), but patient-correctable with a change-log entry.
- **Non-SaMD boundary** — the line the product stays behind: AI as interface to structured, vetted data, never a source of medical judgment. Enforced by architecture, not by prompt wording.
- **Provenance envelope** — the metadata on every AI-derived record (status, source, raw extract, model version, edit history) that makes the non-SaMD boundary mechanically demonstrable.
- **Three-phase arc** — the fixed cycle spine every card is an instance of: Phase 1 stimulation & monitoring → Phase 2 trigger & retrieval → Phase 3 transfer, luteal support & test.
- **Item types (A / B / Mon / C / D1 / D2 / E)** — the seven shapes an item can take. A fixed daily dose; B single exactly-timed event (trigger); Mon monitoring/appointment; C continue-until-date/event course; D1 card-stated comfort conditional; D2 KB-sourced warning-sign escalation; E non-medication instruction. The model calls this "six types" with D1/D2 as the safety-driven split of one. See the Block 1 item-type model.
- **Alternatives** — a single prescription slot with mutually exclusive fills (Progynova *or* Divigel). The parse never picks; the patient selects one from the card's options. A hard confirmation gate.
- **Prescribed-unscheduled** — a valid first-class state: an item has a prescribed dose but no date because the clinic sets it reactively (monitoring visits, the trigger, a C-course end). Visible but dormant; fires no reminder until a date is entered. Distinct from an unset-anchor gap (which is about the patient's routine, not a clinic date).
- **Unset-anchor gap** — an item whose schedule rule is known but whose routine anchor the patient hasn't set. Resolved by the patient setting the anchor. Distinct from prescribed-unscheduled.
- **Warning-sign / D2 escalation** — KB-sourced red-flag content (chief concern OHSS), present for every cycle regardless of card. On a symptom match, the app surfaces cited information and the clinic-contact path — never a severity grade or a verdict about the patient.

### Lifecycle vocabulary — mapping to the protocol model

This document uses plainer state names than the protocol-model artifact. They map as follows, so the two docs describe the same machine:

| This doc | Protocol model | Meaning |
|---|---|---|
| extracted / prescribed | `parsed` | Parsed to the prescribed layer; not yet confirmed. |
| flagged-incomplete | (missing-field flag) | Can't be confirmed until the patient completes it. |
| awaiting selection | `needs_selection` | Has alternatives; can't be confirmed until the patient picks one. |
| awaiting deliberate confirm | `needs_deliberate_confirm` | The trigger's separate confirmation gate. |
| prescribed-unscheduled | `prescribed_unscheduled` | Prescribed, no clinic date yet; visible, dormant. |
| pending / scheduled | `active` | Confirmed and live; reminder-eligible. |
| confirmed | `done` | Patient confirmed done; terminal, patient-correctable. |
| missed / skipped | (terminal, not-confirmed) | Time passed unconfirmed; terminal, patient-correctable. |
| escalated | `escalated` | D2 match, Type E unmet, or trigger timing problem → clinic-contact path. |

---

## Settled decisions (Phase 1)

These are agreed and constrain the stories above.

- **The item model has six types (A, B, Mon, C, D1/D2, E), grounded in the real card.** Reconciled from the protocol-model artifact after an earlier drift to four. The two recovered types carry safety weight: Mon (monitoring/appointment) and the D1/D2 split (card-stated comfort vs. KB-sourced warning-sign escalation). The MVP handles all six.
- **Alternatives are a hard confirmation gate; the patient picks one from the card's options.** A slot with either/or fills never has one auto-picked. Selection is limited to the card's listed options (a drug not listed is a manual patient-entry). An item with unresolved alternatives never reaches the schedule.
- **D2 warning signs live in the KB, for every cycle, independent of the card.** On a recorded symptom the companion checks the KB; a match surfaces cited warning-sign content plus the clinic-contact path, framed as information, never as a severity grade or a verdict about the patient. A non-match captures silently and never implies reassurance. The match is an extra prompt toward escalation, never a gate on it — the clinic-contact path is always available.
- **Two distinct "no time yet" states.** *Prescribed-unscheduled* (clinic hasn't set the date; resolved by prompting the patient for the clinic date, reusing the health-check-in interaction pattern) and *unset-anchor gap* (patient hasn't set the routine anchor; resolved inline). Both are visible and dormant; neither ever gets a guessed time.
- **The 2021/2022 spec files corroborate the model.** The mini-TZ and customer TZ (2021–2022) independently establish the anchor concept (meal times + wake/sleep as profile preferences that build the medication schedule), FCM push, the prescribed/scheduled split, missed-items-don't-carry-forward, and can't-edit-past-items — confirming these are durable domain findings, not new inventions. Their UI-first, phone-auth, iOS specifics are superseded by the 2025 conversation-first reframe.
- **Anchor vocabulary is fixed.** A known set — wake, breakfast, lunch, dinner, bedtime. The parser maps card timing onto this set; timing it can't map is flagged, not guessed. Patient-named anchors are out of scope for MVP.
- **Anchors are defined in the profile, independently of onboarding.** Onboarding never blocks on a missing anchor. It stores the prescribed rule as an anchor reference (`anchor + offset`); resolution to a clock time happens later, in tracking.
- **Resolution happens at read, for not-yet-confirmed items only.** A scheduled item's source of truth is `anchor + offset`, never a frozen clock time — while it is still pending.
- **Confirmation state, not calendar date, is the cut line.** Editing an anchor re-times every *not-yet-confirmed* item that depends on it. It never touches an item that is already confirmed, missed, or skipped.
- **Completed and missed items are terminal and frozen against automatic change.** A confirmed item records *that* the patient confirmed it plus the confirmation timestamp (the moment of the tap) — never a claimed exact administration time. A missed/skipped item is recorded as "not confirmed." Neither is ever silently re-timed by an anchor edit. Both remain **patient-correctable**: the patient can un-confirm, or correct a missed item to confirmed, and every such change is recorded in the item's change log (provenance edit history). "Terminal" means no *automatic* process alters it, not that it is immutable.
- **An item becomes missed when its time passes with no confirmation — by day rollover for routine items, by exact-time-passed for exactly-timed events.** A routine item stays pending through the day and becomes missed at day rollover (local end of the cycle day). An exactly-timed event (the trigger) becomes missed the moment its exact prescribed time passes unconfirmed, since being late is itself the failure. No per-item grace timer.
- **The trigger injection is special-cased.** It is confirmed deliberately and separately at onboarding, and its reminder is handled distinctly from routine daily reminders.
- **Provenance keeps the raw capture.** The uploaded image or voice transcript is stored and linked to the parsed-and-confirmed items for traceability. Manually entered items are marked patient-entered, distinct from clinic-sourced. (Full retention rules: Block 5.)
- **Emergency is passive for MVP.** The app surfaces the clinic's contact numbers and any clinic-provided instructions, fast and always reachable. It does not actively notify the clinic and does not assess urgency. Active clinic notification is post-MVP.
- **Voice is a cross-cutting input modality.** It recurs in manual item entry, symptom capture, and Q&A. Held as a standalone story for now so its behaviour is specified once.
- **Image capture is upload-only for MVP.** No in-app camera capture; the patient uploads an existing image. Re-upload is offered only when extraction returns no items at all — there is no partial-read threshold.
- **MVP is a web app with two surfaces.** A *static dashboard* and a *dynamic chat*. The dashboard is a read-only view of confirmed state in three regions: **today's plan** (items due today with resolved times and state), **progress** (position across the three cycle phases), and **actions needed** (everything requiring patient action — unset-anchor gaps, flagged-incomplete items, doses awaiting a decision). The chat is where all interaction and communication happens — item correction, confirmation, symptom capture, health check-ins, Q&A, plan queries, and timed event messages. The dashboard reflects state; the chat is where state changes.
- **Education Q&A scope is retrieval-defined, with visible citations.** The Education Agent answers only when KB retrieval returns relevant content, and falls back ("I don't have that — ask your clinic") when it does not. There is no separate topic whitelist; what the retrieval finds defines scope. Every answer shows its source citation to the patient, reinforcing that content comes from vetted material, not model opinion. Citation-or-refuse is a hard gate: no citation, no answer.
- **Authentication is via Google account (OAuth) for MVP.** Real, cross-device identity; each identity sees only its own data.
- **Storage is single-region GCP with default encryption.** One GCP region defines geographic residency; encryption is GCP default at rest and in transit; no application-level encryption beyond defaults in MVP.
- **Data persists indefinitely from the patient's perspective.** No self-service deletion via UI or chat. Deletion is available on user request, executed by an administrator out-of-band. A completed cycle is eligible for administrator-initiated, non-destructive archival one year after completion. Retention rules are therefore all "until administrative action," never user-triggered.
- **Delivery is web push via Firebase Cloud Messaging (FCM).** Reminders are delivered as push notifications, subject to browser and OS delivery conditions (notification permission granted, service worker active, browser/OS permitting delivery). A reminder for an item fires at its resolved event time minus a global **lead time** (default 5 minutes, patient-configurable). Lead time shifts the *reminder*, never the *event*: `anchor + offset` resolves the prescribed event time; `lead` only moves when the push fires. The trigger reminds the same way as routine items; its special handling is at onboarding confirmation, not at reminding. Per-item-type lead time, quiet hours, and channel selection are post-MVP.

---

## Status

All five blocks have stories/constraints and acceptance criteria. Non-requirements (N1–N8) and non-functional requirements are written. The Phase 1 exit gate — every MVP capability has stories + acceptance criteria, every safety boundary is a testable negative, KB scope is bounded — is met.

Open reconciliations for later phases: none blocking. The clinic-layer data path (Vision Brief §6) and its consent design remain explicitly post-MVP.
