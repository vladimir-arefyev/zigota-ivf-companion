# Zigota — Architecture (2025 MVP)

*How the conversation-first IVF companion is built: components, boundaries, and their mapping to GCP. The design exists to make one promise mechanically true — AI as an interface to structured data, never a source of medical judgment.*

**Status:** Draft v0.4 · **Author:** Vladimir Arefyev · **Format:** living document · **Companion docs:** `Zigota_Vision_Brief_2025.md`, `Zigota_Build_and_Content_Plan.md`, `Zigota_Requirements_2025.md`

*v0.8 — aligned to Requirements v0.3, and three corrections. (1) **`escalated` removed from the schedule-item lifecycle**: escalation is a safety event referencing an item, never a state of it, because a D2 match must not terminate a medication item or suppress its reminders (§3.1, D11). (2) **Audit model reconciled**: inline provenance, item change log, and `auditEvents` given distinct roles and a named source of record each (§4, D12). (3) **The voice rule narrowed** from "voice never confirms or edits schedule values" to "speech may supply reviewable draft input; it may never itself commit or confirm a schedule mutation" (§2.6) — the blanket ban contradicted Req 1.8 unnecessarily. Also added the **scheduled-conversation** component (§2), which Reqs 2.3 and 2.8 require and which no prior version named, and recorded Tier 2 as sanctioned by the amended Block 3 / N5 rather than pending them (D10).*
*v0.7 — tiering reframed around a **content partition**. The two grounding tiers no longer hold overlapping corpora: Tier 1 is narrowed to clinical content (medications, procedures, warning signs, instructions, contacts), Tier 2 crawls general patient education. The partition is a second, independent barrier alongside the safety-domain classifier (§2.7), which is what changes the Tier-2 risk calculus. Entries carry a `tier` tag rather than being deleted, so the partition is reviewable and reversible (D10). N5 amendment proposed to the requirements doc; Tier-2 enablement remains gated on it.*
*v0.6 — second architecture review (requirements-led). Six findings folded in: reminder execution now on Cloud Tasks + Scheduler with send-time state recheck (§2, D8); the "sole writer" claim made database-enforceable via a command API and Firestore read-only rules (§4, D9); transaction/idempotency/concurrency rules for lifecycle transitions (§3.2); OCR reframed as a non-generative extraction dependency whose output is untrusted (§2, D3); Firestore physical model specified — hierarchy, change_log as a subcollection, Storage refs not payloads, version fields (§3); Tier 2 deferred behind a feature flag, seam kept, requirements-drift acknowledged (§2.7, D7). Added §11 Review dispositions.*
*v0.5 — reworked the Education Agent into a two-tier grounding pipeline: the curated KB (authoritative, exclusive for safety domains) and a Vertex AI Search datastore over curated web sources (general education only, KB-suppressed). Added D7 (tiered grounding), a data-placement principle (patient state → Firestore; curated reference → bundled), and a `web_datastore` provenance source. Protocol Model confirmed as bundled reference data, not Firestore.*
*v0.4 — reconciled against Phase 1 Requirements. Added the item-type & lifecycle state machine (§3.1), expanded the State Engine into a resolution-logic spec (§3.2), added the Reminder/FCM subsystem and the D2 warning-sign matcher as a deterministic gate (§2, Component overview), and made the dashboard/chat two-surface model explicit (§2, Surfaces). Reconciled auth (Firebase Auth as the Google-OAuth mechanism) and realigned encryption to the requirements' MVP posture — GCP-default encryption in MVP; CMEK moved to the production bar (§9, D6).*
*v0.3 — incorporated GCP architecture review. Added §9 Security & compliance and §10 Decisions log; folded build-time embeddings, signed-URL image upload, and identity binding into the design; recorded the ingress/streaming and Document-AI-vs-Gemini-vision decisions with rationale.*
*v0.2 — added voice as an input modality (§2 Transcriber, §2.6 Input modalities); updated deployment tree, GCP mapping, and open decisions.*

---

## TL;DR

Zigota is a **modular monolith on Cloud Run** with **Firestore as the single source of truth**, AI supplied by **Vertex AI (Gemini + embeddings + Search)** and **Document AI**, and curated reference data bundled in the container. English is the baseline language.

The architecture is organized around one load-bearing idea: **the conversation is the interface, the structured records are the data, and every AI-derived record carries its provenance inline.** Nothing an AI produces becomes truth until a human confirms it, and anything in the app can answer "why is this here?" by pointing back to either the patient's own prescription card or a knowledge-base passage — never to model-generated clinical reasoning.

Three deterministic cores do the load-bearing work (a scheduling state engine, a protocol mapper, and an out-of-scope refusal gate); the language models sit on top as thin narration and grounded Q&A. That split is what keeps the product out of medical-device territory and makes the non-SaMD boundary demonstrable rather than merely asserted.

---

## 1. Design principles

The whole architecture follows from four commitments, each stated as something the system will *not* do.

- **The AI never computes the plan.** Scheduling is a deterministic state engine over confirmed data. Language models read that state and phrase it; they do not decide what is due or when.
- **No AI-derived data becomes truth without human confirmation.** The prescription parser produces *proposals*. Only a patient's explicit confirm/edit promotes a proposal to a confirmed record.
- **Every AI-derived record is traceable to a non-AI source.** A schedule event traces to a region of the patient's card; an education answer traces to knowledge-base passages. Provenance is stored inline on the record, not reconstructed from chat history.
- **Clinical judgment is refused deterministically, before generation.** Dose, protocol-choice, and "is my case…" questions hit a lexical out-of-scope gate that returns a fixed redirect to the clinic — the refusal is not a model decision.

These are the same principles as the Vision Brief's non-SaMD boundary, expressed as mechanisms rather than intentions.

---

## 2. Component overview

Eighteen components, grouped by role (adding the Command API and Upload-session modules from the v0.6 review). Each is a module inside a single deployable service (see §5); the grouping is conceptual, not a deployment topology.

### Input modalities — how a message enters

**Transcriber.** Converts spoken input to text before it reaches the router. A modality adapter, not a reasoning component: audio in, text out, then the existing intent-classification path runs unchanged. It adds no new logic downstream — voice is a peer of typed text and card photo as ways a message enters, and all three converge on the Orchestrator. The audio itself is not persisted (see §2.6 and §8); only the resulting transcript, tagged as its provenance source, survives.

### Intake pipeline — photo → confirmed schedule

**Ingestor.** Takes a card photo, runs layout-aware OCR/vision extraction, returns raw text and field candidates with region references (bounding boxes). It is a **non-generative extraction dependency** — not a source of confirmed clinical truth — and its output is always treated as an untrusted *proposal*. It carries no protocol knowledge and no schedule logic. (Note the precise claim: the safety property is not that OCR is deterministic — it isn't; Document AI output can vary by processor version, image quality, and configuration — but that its output never becomes truth without passing the deterministic mapper and, above all, patient confirmation. See §10 D3.)

**Protocol Mapper.** Maps raw extracted fields onto the protocol model — drug to class, dose, timing to `anchor + offset` — producing `proposed` records with provenance attached, and *flagging* fields it cannot map or that are missing rather than guessing them. This is the deterministic, non-generative step: given the same extracted fields it produces the same proposals, and it validates against the protocol model rather than inventing. This is where extraction becomes a candidate schedule the patient will confirm.

**Confirmation Gate.** The human-in-the-loop checkpoint and the **sole writer of confirmed records** — enforced at the database layer, not merely by convention (§4, §10 D9). It presents proposals, captures confirm / edit / reject / select, records the `change_log` entry on any correction, and is the only path by which AI-derived data enters the confirmed store. All writes are server-side through the command API, bound to the authenticated `auth.uid`, and executed as Firestore transactions against an explicit state-transition allowlist.

### Tracker data core — deterministic, no AI

**State Engine.** Given a confirmed schedule, the progress log, and today's date, it computes what is due, what has been logged, and what is next. The single source of schedule truth. No LLM, no persistence of its own — a pure function over Firestore state. "Current progress" is the diff between the confirmed schedule (intent) and the progress log (actual).

**Protocol Model.** Static reference data: phases, drug classes, typical events, expected sequences. The knowledge the Mapper maps *against*. Bundled in the container image; sourced from the knowledge-base work.

### Conversational agents

**Tracker Agent.** A thin conversational surface over the State Engine. Reads computed state, phrases it for the patient, and collects symptom and adherence input into the progress log. It narrates and captures; it never computes the plan. Vertex AI (Gemini), low reasoning load.

**Education Agent.** Retrieval-grounded Q&A over **two tiered sources** (§2.7). A question first passes a deterministic **out-of-scope filter**; if it lexically matches a clinical-judgment stub (dose, protocol choice, personal-case), the agent short-circuits to a fixed refusal *before any generation*. What passes is answered from retrieved passages — never free generation — with a visible citation. Citation-or-refuse is a hard gate: no source, no answer (Req 3.1–3.2). The two sources are **tiered, not merged**: the curated KB answers first and exclusively for safety domains; the web datastore serves only general education the KB doesn't cover (full ordering in §2.7). Vertex AI (Gemini) for generation.

**Warning-sign matcher (D2).** A deterministic gate — architecturally a sibling of the oos-filter — that checks a patient-reported symptom against the KB's warning-sign (D2) content, which is present for every cycle regardless of card. On a match it surfaces the cited KB content plus the clinic-contact path, framed as information ("our information lists this as something to raise with your clinic"), never as a severity grade or verdict. Its match rule is reproducible against a fixed benchmark of canonical warning-sign phrasings — a build-time choice of lexicon / embedding threshold / hybrid, but *not* a free LLM judgment (Req 2.6, 2.70). It never gates escalation: the clinic-contact path is always available; a non-match captures silently and implies nothing reassuring (N3).

**Knowledge Base (Tier 1 — authoritative).** ~60 curated patient-education entries (`kb.yaml`) grouped by kind — out-of-scope, glossary, stage, medication, instruction, warning. Each entry carries a body, aliases (for the lexical filter), and a source citation. It also holds the D2 warning-sign content and the per-deployment clinic-contact / emergency information (vetted, cited KB content, not patient-entered — Req 2.7). Small enough for in-memory matching with no vector database; embeddings are pre-computed at build time and bundled in the image (§7, §10 D4). Entries carry a `review.validated_by` field that is null pending human validation — an honest-limits property the agent can surface. **This is the source Zigota authors and controls; it is authoritative and exclusive for the safety domains (warning signs, emergencies, drug safety, contacts).**

**Curated Web Datastore (Tier 2 — general education). Deferred behind a feature flag; the seam is built, the capability is off in the MVP.** When enabled, it is a **Vertex AI Search** datastore indexing content crawled from a fixed allowlist of authoritative reproductive-medicine and health sources: SART (sart.org), ASRM (reproductivefacts.org), FSANZ (fertilitysociety.com.au), Mayo Clinic (mayoclinic.org), NHS (nhs.uk), HFEA (hfea.gov.uk), and UpToDate (uptodate.com). It gives the breadth the 60-entry KB can't — "what is a follicle," "what happens at retrieval" — but it is content Zigota does *not* author and that changes when those sites change. It is therefore **general-education only, never consulted for safety domains, and only reached after the KB misses** (§2.7). **In the MVP the flag is off: the Education Agent answers from the KB or refuses, matching the Phase 1 requirement that medical answers come only from the curated KB (N5).** The seam is retained so the capability isn't designed out, but enabling it is gated on the content-governance operating model it requires (§8, §10 D7). This is the only place the architecture would use Vertex AI Search; the small KB stays in-memory regardless.

### Reminders

**Reminder execution — durable, not in-process.** A reminder must fire at a future moment, but Cloud Run scales to zero and cannot itself hold a timer — so the schedule cannot live in the request handler. Execution is therefore **Cloud Tasks** for per-item delayed delivery, with **Cloud Scheduler** as an optional periodic reconciliation poll:

- When the Confirmation Gate commits an item, or an anchor edit re-times active items, the service **enqueues a Cloud Task** targeting a dispatch endpoint at the item's resolved `event_time − lead`. On re-timing or cancellation it deletes/replaces the outstanding tasks (deterministic task IDs — `cycleId:itemId:reminderKind:scheduledAt` — make this idempotent).
- The task, when it fires, calls back into a Cloud Run **dispatch endpoint** which **re-checks the item's current state at send time** and sends the FCM web push only if the item is still `active` and unconfirmed. This is what makes "reminders stop after confirmation or cancellation" (Req 2.2) actually hold: the task may have been created hours before the patient confirmed, so the send-time recheck — not task deletion alone — is the real guarantee.
- The **trigger cascade** is three independently-cancellable tasks (−2h, −lead, +1h-if-unconfirmed), each with its own idempotency key, each re-checking state on fire.
- Cloud Scheduler optionally polls at a coarse interval to reconcile anything a task dropped — belt-and-braces for the mission-critical trigger.

The push itself identifies the item and its due time and never assesses the dose. Delivery is best-effort, subject to browser/OS conditions (notably weak on iOS Safari PWA) — a known MVP limitation, and the reason the trigger both cascades *and* is backstopped by reconciliation rather than relying on a single push (Req 2.2, settled decisions, Known limitations; §10 D8).

**Scheduled conversation (app-initiated messages).** A component no prior version named, and one the requirements need: Req 2.3 has the app prompt for a clinic date on a prescribed-unscheduled item, and Req 2.8 has it initiate an optional daily check-in. Neither is a reminder and neither is a reply, so both fell between the Reminder subsystem and the chat surface with no owner.

It shares the Reminder subsystem's infrastructure (Cloud Tasks for delayed delivery, a dispatch endpoint that re-checks state at send time) and differs in three ways that make it a distinct module rather than a reminder variant:

- **Idempotency is per-day, not per-item.** The check-in fires once daily regardless of how many items exist; a clinic-date prompt fires per undated item but must not repeat once the date is entered.
- **Non-response is not a failure.** A skipped check-in creates no missed state and no follow-up (Req 2.8). A reminder's unconfirmed item becomes `missed`; an unanswered check-in becomes nothing at all. Reusing reminder logic here would manufacture an obligation the requirements explicitly deny.
- **It writes a message, not a push.** The output lands in chat as an app-initiated turn, which the Orchestrator must distinguish from a patient-initiated one when classifying the reply.

### Coordination

**Orchestrator (Router).** The entry point for every patient message. Classifies intent — parse card / schedule query / log symptom / ask question / confirm item / health check-in — and dispatches to the right capability. State-changing intents become **commands** routed through the command API (§4); read intents are served from resolved state. The router itself holds no truth. Lightweight classification may suffice, with a model as fallback for ambiguous input.

### Surfaces

**Dashboard (static, read-only).** Reflects confirmed state in three regions: **today's plan** (items due today with resolved times and state), **progress** (position across the three cycle phases), and **actions needed** (unset-anchor gaps, flagged-incomplete items, items awaiting a decision). It is a projection of structured state and displays no AI commentary (Req 2.1, 2.5, settled decision on the two surfaces).

**Chat (dynamic).** Where all interaction and state change happens — item correction and confirmation, symptom capture, health check-ins, Q&A, plan queries, and timed event messages. **The dashboard reflects state; the chat is where state changes.** This split is a first-class architectural commitment, not a UI detail.

### Cross-cutting

**Provenance envelope.** Not a component but a schema convention: the metadata shape every AI-derived record carries inline. See §4.

---

## 2.6 Input modalities

A patient message enters through one of three modalities, all converging on the Orchestrator:

```
Patient ──┬── typed text ──────────────┐
          ├── card photo ── Ingestor ───┤
          └── voice ─────── Transcriber ─┴──▶ Orchestrator ──▶ dispatch
```

Voice is deliberately modeled as an **input modality, not a new capability.** Transcription sits *before* intent classification; everything downstream — routing, the tracker data core, the confirmation gate, provenance — is identical regardless of how the message arrived. This is why voice adds no new architectural risk: the seam already exists at the Orchestrator's front door.

**Transcription options.** Either Cloud Speech-to-Text as a discrete step, or Gemini's native audio input, which lets the conversational agents accept audio directly and skips a separate STT hop. For a Gemini-centric stack the latter is fewer moving parts — audio-in becomes a modality flag rather than new infrastructure.

**Two testable negatives govern voice:**

- **Speech may supply reviewable draft input; it may never itself commit or confirm a schedule mutation.** Earlier drafts stated this as "voice never confirms or edits schedule values," which was overstated and contradicted Req 1.8 unnecessarily. The risk is not speech as a medium: it is an *unreviewed transcript becoming truth*. Req 1.8's own criteria already close that, requiring the transcript to be displayed back, corrected if wrong, and then passed through the standard per-item confirmation. So the narrowed rule is the one worth holding: a patient may speak an entry or a correction, but the commit is always a visual confirmation against displayed text, and for parsed items against the source card region. Spoken-and-re-transcribed values ("Menopur 150, not 75") add a lossy layer exactly where the boundary requires verification against ground truth, and the transcript-review step is what removes it.
- **Audio is not persisted.** Voice is biometric data, carrying privacy weight at least as heavy as card images. Only the transcript is retained, and it becomes part of the provenance chain: a voice-originated symptom record carries `source: voice_transcript` with the transcript as its `raw_extract`. Transcription error is thereby a recognized error surface, the audio-input equivalent of an OCR miss.

Voice is **deferred to pre-pilot**, now formally in Requirements v0.3 rather than only in this document. Story 1.8 in full, plus the voice alternative within 1.4, 2.6 and 2.8, moved to the requirements' Scope boundary. Text plus photo proves the paradigm in the first MVP pass; the seam above is why voice returns as a modality flag rather than as new infrastructure. The narrowed behavioural rule travels with the deferral so it is inherited correctly when the capability lands.

---

## 2.7 Education grounding — two tiers, in order

The Education Agent grounds on two sources with different trust properties, and the discipline that keeps the non-SaMD boundary intact is that they are **tiered, not blended**. **In the MVP, Tier 2 is disabled by a feature flag** — the agent answers from Tier 1 or refuses, satisfying N5. The Tier-2 path below is the enabled-state design; the seam exists so the capability isn't designed out, but it ships off. The order is fixed:

```
question
   │
   ▼
safety-domain classifier ──(safety: D2, dose, emergency, contacts)──▶ Tier 1 KB only
   │ (deterministic, versioned, FAIL-CLOSED: ambiguous → Tier 1 or refuse, never Tier 2)
   ▼
oos-filter ──(clinical-judgment match: dose, protocol choice, personal case)──▶ REFUSE
   │ passes
   ▼
Tier 1 — KB retrieval (clinical corpus) ──(relevant hit)──▶ answer from KB, cite the vetted source
   │ no relevant KB hit
   │ AND question is not a safety domain
   ▼
Tier 2 — Web datastore (general-education crawl) ──(hit ≥ threshold)──▶ answer, attribute origin + URL + crawl date
   │ no hit / below threshold
   ▼
REFUSE ("I don't have that, ask your clinic")
```

**Tier 1 wins any contested question.** Where both tiers could plausibly answer, the KB answers and Tier 2 is suppressed for that query. This is unconditional and is not a relevance comparison: a strong Tier-2 hit never outranks a weak Tier-1 hit. The ordering is an authority statement, not a retrieval optimisation.

**Why tiered and not merged.** The two sources are not interchangeable. The KB is content Zigota authored and vetted; the web datastore is authoritative but externally controlled and changes on re-crawl. Blending them into one retrieval pool would implicitly claim equal authority for both and let a crawled page answer a question the KB should own. The tiering prevents that structurally:

- **The KB is authoritative and exclusive for safety domains** — warning signs, emergencies, drug safety, and contact information answer from the KB *only*. The web datastore is never consulted for these, even if it has matching content, because these are exactly the domains where a stale or subtly-off answer is dangerous and where Zigota curated content specifically so the model can't drift. A symptom question in D2 territory routes to the KB and the deterministic warning-sign matcher, never to a crawled page.
- **The safety-domain classifier fails closed.** It is deterministic, versioned, and testable; when its confidence is ambiguous it routes to Tier 1 or refuses — never to Tier 2. A taxonomy failure can only ever be *more* conservative, never leak a safety question to crawled content.
- **A KB hit suppresses Tier 2 for that query.** When the KB answers, the web datastore is not consulted — one answer, one source, unambiguous provenance. Tier 2 (when enabled) is reached only when the KB has no relevant hit *and* the oos-filter permits *and* the question is not a safety domain *and* the retrieval clears a confidence threshold.
- **The web datastore never answers what the oos-filter or KB would refuse or escalate.** Safety-critical questions never reach Tier 2 at all; the ordering guarantees it.

**The content partition is a second, independent barrier.** The ordering above is enforced by the safety-domain classifier, and a classifier is a single point of failure: one misclassification and a symptom question reaches crawled content. The partition removes that dependency. Tier 1 holds the **clinical** corpus (medications and drug safety, procedures, D2 warning signs, non-medication instructions, out-of-scope stubs, clinic contact); Tier 2 crawls **general patient education** (physiology, glossary, stage overviews, what-to-expect). Because the corpora do not overlap, a classifier miss on a safety question does not produce a wrong-authority answer: the datastore has no relevant clinical content to return, so the query falls through to a refusal. Classifier and partition fail in the same direction, and both must fail simultaneously for a safety question to be answered from crawled content.

This is what changes the Tier-2 risk calculus relative to v0.6, where both stores held substantially the same corpus and the classifier carried the whole load alone.

**The partition line is also the clinic hand-off line.** The long-term intent is that Tier 1 becomes content a clinic curates and signs off. A clinical-only KB is exactly the artifact a clinic would own; a KB that also holds general physiology explainers is not. So narrowing Tier 1 by risk domain is not only a safety scoping move, it builds the boundary the clinic-curated version needs anyway. The corollary for §3's placement principle: per-deployment clinic-owned content varies by tenant and changes outside a code deploy, so the KB loader is kept behind an interface that could later resolve a per-tenant KB from Cloud Storage, even though the MVP bundles a single KB in the image.

**Trust properties, stated plainly.** Both tiers are retrieval-grounded — the model answers from retrieved passages, not model knowledge, so N5 (no free medical generation) holds for both. The difference is governance: a Tier-1 citation points to content Zigota vetted; a Tier-2 citation points to content a third party controls and may have changed since crawl. That is an acceptable trade for general education and an unacceptable one for safety content — which is the whole reason for the split. Tier-2 answers are visibly attributed to their origin ("According to NHS…") so the weaker authority is never disguised as the stronger.

**Why Tier 2 is deferred (not just designed).** Enabling Tier 2 pulls in a whole operating model the MVP doesn't need to prove the core product: crawl scheduling and freshness SLAs, source-change detection, per-page allow/block patterns, snapshot/version metadata and citation fidelity, prompt-injection filtering on crawled pages, retrieval-quality evaluation per source, and — pointedly — licensing validation (UpToDate is a paid product; indexing it requires explicit licensing). The 60-entry KB proves the safety-critical paths without any of that. So the architecture keeps the seam and defers the capability behind a flag until the governance exists and the requirements sanction it (§10 D7). This also corrects a process error: Tier 2 was introduced in v0.5 as an architecture decision that quietly widened N5's "curated KB only" scope; the flag returns the MVP to the requirement and moves the scope question back where it belongs — the requirements doc.

---

## 3. Data model

Four separate concerns, all in Firestore, deliberately kept apart so the intent/actual split and the confirmation boundary stay legible.

- **Profile** — patient-authored routine anchors (the five fixed times: wake, breakfast, lunch, dinner, bedtime) and reminder settings (global lead time, default 5 min). Not clinical, not prescribed. Anchors are what let anchor-relative items resolve to clock times (Req Block 0). An unset anchor is a known gap, never a guessed default.
- **Proposals** — pre-confirmation staging (the requirements' *prescribed layer*). Records with a lifecycle status short of confirmed (§3.1). The Mapper writes here; the State Engine never reads here.
- **Patient Instance** — confirmed schedule records (the requirements' *scheduled layer*), the source of truth for what the patient is supposed to do. Written only by the Confirmation Gate. Each record carries inline provenance and an append-only `change_log` history.
- **Progress Log** — an append-only record of what actually happened: confirmations (with the tap timestamp, never a claimed administration time), symptom captures, health check-ins, all verbatim and timestamped.

The separation matters: **the schedule is intent, the progress log is actual, and current progress is the diff.** Keeping proposals out of the confirmed store means the State Engine can trust everything it reads without status-checking.

**Placement principle: patient state → Firestore; curated reference → bundled.** Firestore holds only the four *patient-data* concerns above — data written per-patient at runtime, queried per-patient, and mutated by patient action. Everything that is *curated reference data* — the Protocol Model (item types, phases, anchor vocabulary), the KB (`kb.yaml`) and its pre-computed vectors — ships **in the container image**, not Firestore. The deciding question is *who writes it and how often*: reference data changes at the cadence of code (authored by the team, versioned in git, reviewed in a PR, deployed atomically with the code that depends on it), while patient data changes at the cadence of use. Putting reference data in the image gives it versioning, review, and atomic deployment alongside its consumers, and removes a runtime read dependency and failure mode for something that never changes at runtime. This is why the Protocol Model is bundled, not in Firestore: no runtime process writes it, and a change to it is a change you would want tested and shipped with code. The one condition that would flip it — editing the Protocol Model without a redeploy, or selecting per-clinic protocol variants at runtime — is out of MVP scope (one anchor protocol, team-authored); the GCP table records Cloud Storage as the externalization path if that condition ever arrives. (The Tier-2 web datastore is neither patient state nor bundled reference — it is externally-governed content indexed in Vertex AI Search, which is exactly why §2.7 tiers it below the KB.)

**Physical model — hierarchy, tenancy, and growth.** The four logical stores map onto a strict per-patient document hierarchy, which is also the tenancy boundary (every path is UID-rooted, and Firestore security rules enforce `request.auth.uid == uid` at the rule level):

```
/users/{uid}
   profile
/users/{uid}/cycles/{cycleId}
   metadata
   proposals/{proposalId}          # prescribed layer, pre-confirmation
   scheduleItems/{itemId}          # scheduled layer, confirmed
   progressEvents/{eventId}        # append-only: confirmations, symptoms, check-ins
   safetyEvents/{eventId}          # append-only: D2 match, contact surfaced, E unmet
   rawCaptures/{captureId}         # OCR text + Storage ref + region coords, once per card
   auditEvents/{eventId}           # append-only change/transition log (was inline change_log)
   conversations/{cid}/messages/{mid}   # ops transcript, separate from audit
```

Three growth/tenancy rules follow, and they correct a v0.5 over-simplification:

- **The `change_log` is a subcollection (`auditEvents`), not an array embedded in the item.** An unbounded array inside a schedule document would eventually hit Firestore's 1 MB document ceiling; an append-only subcollection of immutable transition events does not, and it is a cleaner audit substrate (actor, server timestamp, from→to, reason, command ID per event).
- **Firestore stores a Storage *reference*, never an image payload** — object path, content hash, upload metadata, and the scoped provenance pointer. Raw OCR text is stored **once** per `rawCapture` and referenced from derived proposals and confirmed items, rather than copied into every derived record. Card regions are stored as normalized coordinates plus an image-version id.
- **Every durable record carries version stamps** — `schema_version` on the record, and `kb_version` / `protocol_model_version` / `parser_version` / `policy_version` in provenance — so an answer or a parse can always be tied to the exact reference data and code that produced it. Composite indexes are defined explicitly for the hot reads: today's plan, active items by date, unresolved actions, progress views, and audit export.

---

## 3.1 Item types and the lifecycle state machine

Every schedule item is one of seven types and moves through a defined lifecycle. Both are load-bearing: the Mapper assigns type, the Confirmation Gate enforces the type-specific confirmation rules, and the State Engine drives the state transitions. This is the shared vocabulary the intake pipeline and tracker core both operate on (Req Block 1 item-type model, lifecycle table).

**Seven item-type labels (six conceptual types; D splits for safety):**

- **A — fixed daily dose** over a window. Recurring, anchor-relative.
- **B — single exactly-timed event** (the trigger injection). One exact moment; its own deliberate confirmation gate.
- **Mon — monitoring / appointment** (scans, retrieval, transfer, beta-hCG). Usually *prescribed-unscheduled*: the clinic sets the date reactively.
- **C — continue-until-date/event course** (luteal support until the beta test). Daily item whose end is an event not yet dated at parse.
- **D1 — card-stated comfort conditional** (*if X then Y* written on the card). Surfacing it is transcription, not advice.
- **D2 — warning-sign escalation** (OHSS and red flags). *Not parsed from the card* — sourced from the KB for every cycle regardless of what the card lists. Behaviour: capture-and-escalate via the Warning-sign matcher (§2), never grade or interpret.
- **E — non-medication instruction** (fast before retrieval, no driving after sedation). Surfaced as procedure-prep; escalated if reported unmet, never assessed.

**Alternatives** are a cross-cutting property, not a type: one slot can offer mutually exclusive fills (Progynova *or* Divigel; trigger = Pregnyl *or* Ovitrelle *or* triptorelin). The parse never picks; the patient selects one from the card's options — a hard confirmation gate.

**Lifecycle states** (the same machine the requirements' lifecycle table defines; component that drives each transition in brackets):

```
                 [Mapper]                    [Confirmation Gate]
   raw parse ──▶ parsed ──┬─▶ needs_selection ─────────┐   (alternatives: patient picks one)
                          ├─▶ needs_deliberate_confirm ─┤   (trigger: separate gate)
                          └─▶ (missing field) ──────────┤   (flagged-incomplete: patient completes)
                                                         ▼
                                    prescribed_unscheduled   (no clinic date yet: visible, dormant)
                                                         │   [State Engine: date entered]
                                                         ▼
                                                      active   ◀── the only mutable state
                                       [State Engine / patient]  │  (re-times on anchor edit)
                          ┌──────────────────────────────────────┼
                          ▼                    ▼                  ▼
                        done                 missed          cancelled
                   (patient confirmed)   (time passed,    (cycle ended: pending +
                    tap timestamp)        unconfirmed)     unscheduled stopped,
                          │                    │            history preserved)
                          └── patient-correctable ──┘

   safety events (D2 match · contact path surfaced · Type-E unmet · trigger
   timing problem) reference an item but never change its lifecycle. See below.
```

**Escalation is an event, not a state.** Earlier drafts placed `escalated` as a terminal sibling of `done` / `missed` / `cancelled`, which is wrong and would have been a live defect: a patient reporting a concerning symptom on a day she still has doses to take would have had the referenced medication item terminated and its reminders suppressed. Escalation is recorded as a **`safetyEvent`** (kinds: `warning_match`, `contact_path_surfaced`, `instruction_unmet`, `trigger_timing_problem`) in the progress store, carrying a reference to the item or symptom report that occasioned it. The schedule item's lifecycle is untouched. The division of labour:

- **`scheduleItem.lifecycle`** — states that govern scheduling and reminding, and nothing else.
- **`progressEvent.kind`** — what the patient did or reported (confirmation, symptom capture, check-in).
- **`safetyEvent.kind`** — what the safety layer surfaced in response, with the item or report it references.

This also prevents a subtler distortion: with escalation as a state, the count of escalated items would double as a symptom-severity tally, which is precisely the interpretation N2 forbids. As events, escalations are a log of what the app showed, not a grading of what the patient has.

Two rules the State Engine enforces and that the architecture must not violate:

- **`active` is the only mutable state.** `done`, `missed`, and `cancelled` are terminal — never re-timed by an anchor edit. `done` and `missed` remain *patient-correctable* (un-confirm, or correct a missed item to done), and every correction is a `change_log` entry on the record (the append-only history in §4). "Terminal" means no *automatic* process alters it, not immutable.
- **Two distinct "no time yet" states, never conflated.** `prescribed_unscheduled` (clinic hasn't set a date — resolved by prompting the patient for the clinic date) and an *unset-anchor gap* (patient hasn't set a routine anchor — resolved inline). Neither ever acquires a guessed time or date.

---

## 3.2 The State Engine — resolution logic

The State Engine is the deterministic heart of the tracker, and it carries more than "due/next/logged." It owns the temporal rules that make the schedule correct, and it contains no LLM. Its inputs are the confirmed schedule (Patient Instance), the Profile anchors, the Progress Log, and the current time; its output is the resolved plan the dashboard and Tracker Agent read.

**Resolution at read (`anchor + offset`).** A pending item stores its timing as `anchor + offset` (e.g. "breakfast − 30min"), *never* a frozen clock time. The engine resolves it to an event time at read against the current Profile. This is why editing an anchor re-times items without rewriting them: the offset is the source of truth, the clock time is derived (Req 0.1, 0.3, glossary).

**Re-timing is confirmation-gated, not date-gated.** An anchor edit re-times every `active` (not-yet-confirmed) item depending on that anchor, preserving each offset. It touches nothing terminal — a `done` item keeps its frozen confirmation timestamp; a `missed` item stays missed (Req 0.3, settled decisions).

**Missed is a wake-anchored rollover, not midnight.** A routine (Type A) item becomes `missed` at the start of the next *waking* day — the wake anchor — not at 23:59. This deliberately prevents a post-midnight bedtime dose (e.g. progesterone at 01:00) from being wrongly marked missed while the patient still considers it part of the same day. An exactly-timed event (the trigger) becomes `missed` the moment its exact time passes unconfirmed — being late is itself the failure. No per-item grace timer (Req settled decision on missed-item boundary).

**Exact moments are timezone-fixed.** The trigger's confirmed date/time is an absolute moment that does not drift on travel or DST. General timezone/DST handling for anchor-relative items is deferred to pre-pilot; the trigger's fixed-moment rule holds in MVP because mistiming it can end the cycle (Req settled decision).

**The dashboard reads three derived views** from the engine (the two surfaces in §2): today's plan (resolved times + state), progress (position across the three phases from confirmed/missed counts), and actions-needed (unset-anchor gaps, flagged-incomplete items, items awaiting a decision). All three are read-only projections of structured state — never AI commentary.

**Correctness under concurrency.** Because the requirements place state-transition correctness above general NFRs, the engine's transitions are protected against the race conditions a real cycle produces — a patient confirming a dose while its reminder task fires, an anchor edit landing while reminders are already scheduled, a clinic date entered twice from two devices, a correction and a missed-rollover crossing the wake boundary, two sessions submitting corrections at once. The controls:

- **Every lifecycle transition runs in a Firestore transaction** through a single state-transition function with an explicit `from → to` allowlist — an illegal transition is rejected, not applied.
- **Optimistic concurrency** via a monotonic `version` (or precondition) on each schedule item, so a stale write loses rather than clobbers.
- **Commands are idempotent**, keyed by a request ID, so a retried or duplicated command applies once.
- **Server timestamps** are authoritative for confirmation and audit events, never client-supplied clocks; provenance fields are immutable after creation, amended only by append.
- **`missed` is preferably a *derived* view** — computed from "event time passed with no confirmation" rather than written by a background job — which sidesteps the wake-boundary race entirely. If a persisted `missed` event is ever needed, the worker transactionally re-checks the latest confirmation state before writing.

These are what make the state machine in §3.1 hold under real use, not just on paper.

---

## 4. The confirmation & audit layer

This layer is what converts the non-SaMD boundary from a claim into a demonstrable property. It has two distinct jobs.

### Job 1 — Confirmation (forward-looking, gates writes)

The rule: *the AI proposes, the patient confirms, only confirmed data becomes truth.* The parser does not write a schedule; it writes a proposal with a status flag. The State Engine reads only confirmed records. This yields a clean boundary the non-SaMD claim rests on, an edit path that is itself captured, and a confirmation screen that *is* the trust-building interaction rather than friction to minimize.

**"Sole writer" is enforced at the database, not by convention.** A design that merely *says* the Confirmation Gate is the only writer is unenforceable if the client SDK can write the confirmed collection directly — a bug or a malicious client would bypass it. The claim is made structurally demonstrable:

- **Clients get read-only access** to their UID-scoped documents via Firestore security rules. They cannot write `scheduleItems`, `progressEvents`, `safetyEvents`, `proposals`, or any lifecycle field.
- **All mutations go through an authenticated command API** on Cloud Run — `POST /cycles/{cycleId}/commands` with a typed command (`confirm_item`, `correct_item`, `select_alternative`, `set_anchor`, `enter_clinic_date`, `cancel_cycle`). The backend validates the Firebase ID token, authorizes the `uid` against the cycle, checks the transition against the allowlist, and writes in a transaction with a server timestamp.
- **The command pattern makes the boundary testable**: "no path exists for AI output or a client to reach the confirmed store except a validated, authenticated, transactional command" is a property you can write adversarial tests against (N6), not an intention you assert.

This upgrades the non-SaMD boundary from organizationally-intended to structurally-enforced — which matters, because that boundary is the whole product.

### Job 2 — Audit / provenance (backward-looking, explains any item)

Every AI-derived item carries a trail back to its non-AI source, so any element in the app can answer "why is this here?" independent of chat history:

- A **schedule event** links to the source card region it was parsed from (bounding box + raw OCR text).
- An **education answer** links to the KB passages it was grounded in (entry IDs + retrieved text).
- A **parser mapping** records the model version, confidence, and the extracted-field → protocol-field mapping.

Provenance lives on the **record**, not on the chat message. The chat is ephemeral, linear, and one-turn-produces-many-records; the records are durable and independently inspectable. A patient tapping an event on day nine to ask "why is this here?" gets an answer from the record, not from scrollback.

### The envelope

Every AI-derived record carries this shape inline:

```
{
  item_id,
  record_kind: schedule_item | answer | symptom_capture,
  item_type: A | B | Mon | C | D1 | D2 | E,     // schedule_item only
  lifecycle: parsed | needs_selection | needs_deliberate_confirm
           | prescribed_unscheduled | active | done | missed
           | cancelled,                         // the §3.1 state machine
                                                // note: no `escalated` — see §3.1
  origin: clinic_sourced | patient_entered,      // Req 1.4 / 5.1; never conflated
  produced_by: { component, model_version, timestamp },
  provenance: {
    source: card_region | kb_entry_ids | web_datastore | voice_transcript | user_input,
    raw_extract,        // OCR text, retrieved passage, or voice transcript
    citation,           // KB entry source, or web origin + URL + crawl date (Tier 2)
    confidence          // where meaningful
  },
  confirmed_by_user: bool,
  change_log: [ { before, after, timestamp } ]   // append-only (edit + correction history)
}
```

The `lifecycle` field is the same state machine §3.1 defines; the `change_log` is the append-only correction history that keeps `done`/`missed` items patient-correctable without an automatic process ever altering them (Req 2.4, 5.1). `origin` carries the clinic-sourced vs patient-entered distinction the requirements never allow to blur. For an education answer, `source` records *which grounding tier* answered — `kb_entry_ids` (Tier 1, vetted) or `web_datastore` (Tier 2, externally-governed, with origin + URL + crawl date) — so the audit layer can always say whether an answer came from curated or crawled content (§2.7).

**Three concerns, three homes, one source of record each.** v0.6 introduced `auditEvents` as a subcollection while §4's envelope still showed an inline `change_log`, leaving two partially-overlapping audit stores with no stated division. They are not redundant; they answer different questions and have different readers, retention, and mutability. Named explicitly:

| Concern | Answers | Home | Source of record |
|---|---|---|---|
| **Inline provenance** | "Why is this item here?" Lineage back to a card region, KB entry, or patient statement. | Fields on the record (`origin`, `produced_by`, `provenance`) | The record itself. Immutable after creation; amended only by append. |
| **Item change log** | "What did I correct, and when?" Patient-visible correction and lifecycle history. | Derived view over `auditEvents` filtered to that item | `auditEvents`. Not a separate array. |
| **Audit events** | "Who wrote this, under what authority?" Actor uid, command type, idempotency key, before/after version, transition, authorization outcome. | `auditEvents` subcollection, append-only | Itself. The ledger N6's proof rests on. |

The rule that removes the ambiguity: **`auditEvents` is the single write ledger; the patient-facing change log is a projection of it, never a parallel store.** The inline `change_log` array shown in the envelope above is therefore a *rendering* of the relevant `auditEvents`, not a second copy, and nothing writes to both. This matters beyond tidiness: N6's "no path reaches the confirmed store except a validated command" is provable only if every write appears in exactly one ledger, and a future deletion traversal (Req 5.2a) needs to know which store is authoritative before it can guarantee no orphan survives.

A further immutable, exportable per-cycle "everything the AI did" report remains deferred; it becomes worthwhile only if immutability guarantees or a clinic-value/regulatory-readiness export is required.

### A property, not a byproduct

Three things fall out of this layer for free:

1. **It is the mechanism of non-SaMD.** Any item can be shown to trace to the patient's own card or a KB passage, never to model-generated clinical reasoning.
2. **It is the parser evaluation harness.** The `change_log` on every confirmation is labeled error data — exactly where the parser was wrong — without a separate accuracy study.
3. **The out-of-scope filter is a deterministic gate,** not a model judgment, so the refusal boundary is inspectable.

---

## 5. Deployment shape — modular monolith

One Cloud Run service, components as internal modules, one deploy.

```
zigota-service (Cloud Run, Python)
├── orchestrator/     router — intent → read (served) or command (mutating)
├── command_api/      authenticated mutations: confirm/correct/select/set_anchor/…
│                     token-validate → authorize uid → transition allowlist → txn write
├── transcriber/      voice → text (Speech-to-Text or Gemini audio-in)
├── ingestor/         non-generative extraction; output is untrusted proposal
├── mapper/           raw fields → typed proposed records (A/B/Mon/C/D1/E), flags gaps
├── protocol_model/   static reference: types, phases, anchor vocabulary (bundled)
├── state_engine/     anchor+offset resolution, wake-anchored rollover, re-timing; txns
├── confirmation/     sole writer of confirmed records (DB-enforced, §4) + auditEvents
├── reminder/         enqueues Cloud Tasks; dispatch endpoint re-checks state → FCM
├── scheduled_conv/   app-initiated chat turns: clinic-date prompts, daily check-in
│                     per-day idempotency; non-response is not a missed state
├── education/        safety-classifier → oos-filter → Tier-1 KB → [Tier-2 flag] → refuse
│   └── kb_vectors    Tier-1 embeddings pre-computed at build time, bundled in image
├── warning_sign/     deterministic D2 symptom→KB matcher (benchmark-tested, no LLM)
├── tracker_agent/    Gemini — narrate state, capture symptoms / check-ins
├── upload_session/   issues scoped, short-lived signed URLs; derives object identity
└── shared/           Firestore client, provenance envelope, item type/state model

frontend (web, two surfaces)
├── dashboard/        read-only projection: today's plan · progress · actions-needed
└── chat/             all interaction and state change
```

The choice is deliberate: fewer moving parts, one deploy, scales to zero — a fit for a solo builder. Component boundaries live as **module boundaries**, so if a later constraint demands it (Document AI latency, independent agent scaling), the Ingestor or the agents can be peeled into their own services along seams that already exist. Deployment topology can change without the component design changing.

Two edge patterns keep the monolith from becoming an operational bottleneck (see §9 and §10): card images are uploaded by the client **directly to Cloud Storage via a signed URL**, so heavy image bandwidth never transits the container; and knowledge-base **embeddings are pre-computed at build time** and bundled in the image, so a scale-from-zero event incurs no embedding latency or Vertex AI rate-limit contention.

---

## 6. GCP mapping

| Component | GCP service | Notes |
|---|---|---|
| Command API | **Cloud Run** (module) | Sole mutation path: Firebase token validation → uid authorization → transition allowlist → Firestore transaction. Clients have no direct write access (§4). |
| Transcriber | **Cloud Speech-to-Text** or **Gemini audio-in** | Modality adapter before the router. Gemini native audio removes the separate STT hop. Transcript persisted, audio not. Route through HIPAA-BAA / ZDR-configured endpoints pre-pilot. |
| Ingestor | **Document AI** | Layout-aware OCR with bounding boxes for region provenance. Reads the image from Cloud Storage, not through the container. **Non-generative extraction dependency; output is an untrusted proposal, not deterministic clinical truth** (§10 D3). |
| Protocol Mapper | **Cloud Run** (module) | Deterministic Python. Assigns item type (A/B/Mon/C/D1/E), maps timing to `anchor + offset`, flags unmappable/missing fields, writes typed proposals. |
| Protocol Model | Bundled static / **Cloud Storage** | Item types, three-phase spine, fixed anchor vocabulary. Ships in the image. |
| Profile | **Firestore** | Patient anchors (five fixed times) and reminder lead time. Read by the State Engine at resolution. |
| Patient Instance | **Firestore** | Confirmed schedule (`scheduleItems`). Storage refs + provenance pointers, not payloads; `auditEvents` subcollection, not an embedded array (§3). |
| Progress Log | **Firestore** | Append-only `progressEvents` (confirmations, symptoms, check-ins) and `safetyEvents` (D2 match, contact-path surfaced, Type-E unmet, trigger timing), server-timestamped. Safety events reference items; they never mutate a lifecycle (D11). |
| State Engine | **Cloud Run** (module) | Pure function over Firestore. `anchor + offset` at read, wake-anchored missed (preferably derived), confirmation-gated re-timing, transactional transitions. No LLM. |
| Confirmation Gate | **Cloud Run** (via Command API) | The sole *server-side* writer of confirmed records, DB-enforced by read-only client rules (§4). Per-item, trigger, and alternatives gates; transactional; `auth.uid`-bound. |
| Reminder execution | **Cloud Tasks** (+ **Cloud Scheduler**) → **FCM** | Per-item delayed task at `event_time − lead`; dispatch endpoint re-checks item state at send time before pushing. Trigger = 3 idempotent cancellable tasks. Scheduler reconciles. Web push best-effort (§10 D8). |
| Scheduled conversation | **Cloud Tasks** / **Cloud Scheduler** → Firestore chat | App-initiated turns (Req 2.3 clinic-date prompt, 2.8 daily check-in). Per-day idempotency; a skipped check-in creates no missed state. |
| Education Agent | **Vertex AI (Gemini)** | safety-classifier (fail-closed) → oos-filter → Tier-1 KB → [Tier-2 flag, off in MVP] → refuse; citation-or-refuse, tiers never merged (§2.7). |
| Warning-sign matcher (D2) | **Cloud Run** (module) + KB | Deterministic symptom→KB match, benchmark-tested, no LLM. Surfaces cited content + contact path; never grades. |
| Knowledge Base (Tier 1) | **Cloud Storage** (source) + **build-time embeddings** | ~60 entries incl. D2 warning-signs and clinic-contact content; embeddings pre-computed in CI/CD, bundled. In-memory, no Vector Search. Authoritative + exclusive for safety domains. |
| Web Datastore (Tier 2) | **Vertex AI Search** *(flag off in MVP)* | Crawled index over a fixed source allowlist. General education only; reached only after a KB miss; never for safety domains. Disabled until content-governance model exists (§10 D7). |
| Tracker Agent | **Vertex AI (Gemini)** | Thin narration + symptom/check-in capture. |
| Orchestrator | **Cloud Run** (module) | Routes reads to resolved state, mutations to the Command API. Lightweight intent classification; model fallback for ambiguity. |
| Upload session | **Cloud Run** (module) + **Cloud Storage** signed URL | Validates uid + cycle ownership, mints a random capture ID and constrained object path, issues a short-lived MIME/size-scoped signed URL; server derives object identity, never trusts a client URI (§9). |
| Dashboard / Chat | **Firebase Hosting** (web) | Two surfaces: dashboard is a read-only projection of confirmed state; chat is where state changes. Firestore real-time listeners drive the dashboard. |
| Card image upload | **Cloud Storage** via **signed URL** | Client uploads directly; offloads image bandwidth. GCP-default encryption in MVP; CMEK is the production bar (§10 D6). |

**Cross-cutting managed services:**

- **Firebase Auth (Google OAuth)** — patient identity. The requirements specify Google-account sign-in (Req 4.1); Firebase Auth is the mechanism that provides it, giving real cross-device identity and the `auth.uid` that write-binding and Firestore rules key on.
- **Ingress** — **Cloud Load Balancing (HTTPS) + Cloud Armor**, with Firebase Auth validated at the backend/middleware. Chosen over API Gateway; rationale and the streaming implication are in §10.
- **Secret Manager** — keys.
- **Cloud Logging** — ops/debug transcript of chat, kept **separate from the audit layer**. Conflating the two is how the non-SaMD boundary gets muddy: the transcript is observability, the audit layer is data. PHI-scrubbing controls are specified in §9.
- **Cloud Storage** — `kb.yaml` source of record and card-image blobs, GCP-default encryption (see §8, §9).

---

## 7. Why in-memory retrieval, not a vector database

The knowledge base is ~55 KB — about 60 entries. At that scale a vector database is overhead, not leverage. Two properties of the entries make in-memory retrieval clean:

- Each entry carries an **aliases** list (including cross-language synonyms), so a deterministic lexical pre-filter can route obvious matches and, critically, catch out-of-scope questions *before* any model runs.
- ~60 embedding vectors held in memory make paraphrase-robust retrieval trivial. The vectors are **pre-computed at build time and bundled in the image** (§10, D4), not embedded at startup, so retrieval has no cold-start or external-API cost; they are recomputed only when the KB changes, which is a build-time event anyway.

The pattern is **lexical oos-filter first, then in-memory embedding match.** The filter makes the refusal boundary deterministic; embeddings handle everything that passes it. Vector Search is not used *for the KB* — and won't be until the KB grows by an order of magnitude. It does appear elsewhere in the stack as the Tier-2 web datastore (§2.7), which is a different problem: indexing a large, externally-crawled corpus, where a managed search index is the right tool. The two are not in tension — the small curated KB stays in-memory; the large crawled corpus uses Vertex AI Search.

---

## 8. Open decisions

Deliberately unresolved, flagged for a later pass. (Decisions that the architecture review *closed* are recorded in §10, not here.)

- **Card-image retention policy.** The *mechanism* is now decided — if retained, images live in Cloud Storage uploaded via signed URL, GCP-default encryption in MVP (§9). What remains open is the *policy*: whether to retain at all past confirmation, and for how long. Retention strengthens provenance (the region crop stays inspectable); deletion-after-confirmation strengthens the privacy posture. This is a product/compliance call, not a technical one.
- **Orchestrator classification.** Whether four intents justify a model-based classifier at all, or whether structured/keyword routing suffices with a model only as an ambiguity fallback.
- **Separate audit log.** Deferred in favor of inline provenance; revisited if immutability guarantees or an exportable per-cycle AI-activity report become requirements.
- **Language beyond English baseline.** The source card and KB aliases are bilingual (the anchor protocol is from a Russian-language clinic card); cross-language retrieval and a non-English patient surface are out of scope for the MVP but structurally anticipated.
- **Voice sequencing and transcription choice.** Voice is architecturally accommodated (§2.6) but deferred past the first MVP pass. When built, the Speech-to-Text-vs-Gemini-audio choice is open — and the interaction between transcription confidence and the symptom-capture provenance chain will need the same edit-path treatment the parser gets.
- **Streaming for the education surface.** Deferred for the MVP (non-streamed responses), but the ingress was chosen so it stays a config-level change rather than a re-architecture (§10).
- **Web-datastore governance (Tier 2).** Tier 2 ships **disabled by a feature flag** in the MVP (D7); the questions that must be answered before the flag can flip are content-governance ones — crawl cadence and freshness SLA, source-ownership approval, page-level allow/block, snapshot/version metadata and citation fidelity, prompt-injection filtering on crawled pages, per-source retrieval-quality evaluation, a source-change/removal/reindex process, and explicit UpToDate licensing validation. These sit with the pre-pilot KB/content-governance workstream, not the MVP.
- **Reminder execution tier for the demo.** D8 fixes the architecture (Cloud Tasks + Scheduler + send-time recheck). What's left to *pick* is the MVP dial: a minute-level Cloud Scheduler poll may be enough to demo, with item-level Cloud Tasks reserved for pilot-grade timing. A sizing call, not a design gap.

---

## 9. Security & compliance

The product handles medical data (prescription cards) and, once voice ships, biometric data (audio → transcripts). The following controls are part of the architecture, not afterthoughts. None of them alter the component design; they harden its edges.

**PHI in logs — the largest default risk.** Cloud Run, the ingress layer, and Vertex AI can, by default, capture raw request/response payloads — which for this app means card text and conversation transcripts landing unmasked in centralized logging. The primary control is *not logging them in the first place* — exclusion filters don't retroactively purge what's already written:
- **Never log raw request/response bodies** for chat, OCR, image, symptom, or education endpoints. Log structured events only: correlation ID, event type, component, latency, non-sensitive status.
- **Cloud Logging exclusion filters** as a backstop, and **Cloud DLP** inspection on any trace that *is* persisted — defense-in-depth, not the primary privacy control.
- **Disable model prompt/response logging** where configurable (Vertex AI), and ensure error reporting doesn't serialize request context.
- This reinforces the §4 boundary between the **ops transcript** (observability, must be scrubbed) and the **audit layer** (data, deliberately retained with provenance). The two must not share a store.

**Encryption at rest.** MVP uses **GCP-default encryption** at rest and in transit, single-region, with no application-level encryption beyond defaults — the posture the requirements set for a portfolio prototype not processing real patient data (Req 4.2, 5.x). **Customer-managed encryption keys (Cloud KMS CMEK)** are the production bar, not an MVP control (§10, D6); they belong to the pre-pilot hardening pass alongside consent, KB governance, and the admin actor model, when real patient data enters the system.

**Identity binding and write enforcement.** Firebase ID tokens (Google OAuth) are validated in the **application/command-API middleware** — not merely "at ingress," since the load balancer terminates TLS and applies Cloud Armor but token validation is an application concern. Clients hold **read-only** Firestore access to their own UID-scoped documents; every mutation goes through the authenticated command API bound to `auth.uid` (§4, D9). Firestore security rules enforce `request.auth.uid == uid` at the database layer as defense in depth, so no confirmed medical record can be written except by an authenticated patient acting on their own data through a validated command.

**Signed-upload session (secure object lifecycle).** Direct-to-Storage upload is the right pattern, but the object identity is server-derived, never client-supplied: the client requests an upload session for a specific cycle; the backend validates the token and cycle ownership, mints a random capture ID and a constrained object path, and issues a short-lived signed URL scoped by MIME type and size; after upload the backend verifies existence, size, content type, and ownership before OCR, and associates OCR output only with that authorized capture. The service never trusts a client-submitted Storage URI.

**Service-account least privilege.** The container's service account is scoped to exactly the APIs it uses (Document AI, Vertex AI, Firestore, the specific Storage buckets, Cloud Tasks) — not a broad project-editor role. Application, scheduler/task-dispatch, and CI/CD run as separate identities. Each externalized service later inherits only its own subset.

**Identity lifecycle (pre-pilot).** Token refresh, session expiry, logout, revoked tokens, account switching, UID-enumeration prevention, and download authorization (no long-lived public URLs for card images) are lightweight in the demo but become mandatory before real users, since the system stores medication and symptom data. Tracked with the pre-pilot gates (§11).

**Data-flow summary (trust boundaries):**

```
Patient device ──[HTTPS, Cloud Armor]──▶ Command API (Firebase token validated, uid authorized)
   │                                          │  reads: served from resolved state
   │                                          │  mutations: transition allowlist → Firestore txn
   └─[scoped signed URL]─▶ Cloud Storage         ▼
        (server-derived object id)  │      Cloud Run service
                                    └─read─▶ Ingestor ─▶ Mapper ─▶ [Confirmation Gate = sole writer]
                                                                          │
                          Firestore (read-only client rules, uid-scoped) ◀┘
   Cloud Tasks ─(delayed, idempotent)─▶ dispatch endpoint ─(recheck state)─▶ FCM push
   Logs ──[no payload bodies; structured only; DLP backstop]──▶ Cloud Logging (separate from audit)
```

---

## 10. Decisions log

Decisions made deliberately, with the reasoning preserved so they are not silently relitigated. D1–D2 came out of the first GCP review; D3, D7–D9 were shaped or corrected by the second, requirements-led review (§11).

**D1 — Ingress: Cloud Load Balancing + Cloud Armor, not API Gateway.**
*Decision:* Front the service with an HTTPS Load Balancer and Cloud Armor, validating Firebase Auth at the backend, rather than GCP API Gateway.
*Why:* Three independent reasons point the same way. Cloud Armor gives WAF/DDoS protection appropriate for a health product; the Load Balancer path avoids API Gateway's rigid 32 MB request ceiling; and — the forcing function — API Gateway does not support Server-Sent Events, which would foreclose token streaming on the conversational surface. Choosing the Load Balancer now costs little and keeps all three options open.
*Status:* Adopted for the MVP.

**D2 — Streaming: none in the MVP, but keep it a config-level change.**
*Decision:* The MVP serves **non-streamed** (unary) responses everywhere. Streaming is not built, but the ingress (D1) is chosen so that adding it later does not require re-architecture.
*Why:* Streaming only meaningfully benefits one of the four interaction surfaces — **education Q&A**, where progressive rendering of a multi-sentence grounded answer improves *perceived* latency (it does not reduce real latency). The other surfaces don't need it: prescription parsing returns structured records for visual confirmation, not prose (its latency is Document AI OCR, best handled with a progress indicator and the async upload pattern, not streaming); tracker narration is one or two sentences where streaming is imperceptible; the oos-filter refusal is a fixed string returned instantly. For an MVP with short, grounded answers, spinner-then-answer is acceptable, and non-streamed is materially simpler (a plain unary Gemini call and JSON response).
*Rework minimization — what keeps streaming cheap to add later:*
- **Ingress already supports SSE** (D1), so no edge replacement is needed — the single most expensive part of a late switch is pre-paid.
- **Streaming is a delivery concern only; it does not touch the audit layer.** A streamed education answer still attaches its `kb_entry_ids` to the *record* when generation completes, not to the streamed chunks. Provenance is unaffected, so enabling streaming cannot compromise the non-SaMD boundary.
- **The Education Agent's internal contract stays the same** — oos-filter → retrieve → ground. Whether the grounded tokens are buffered into one response or forwarded incrementally is a property of the response handler, not the agent logic. The seam is at the transport, and it is isolated.
The residual work when streaming is wanted: switch the education endpoint's response handler from buffered to SSE, and the client from awaiting a payload to consuming an event stream. That is bounded, edge-local work — the same "keep the seam, defer the build" pattern applied to voice and to service decomposition.
*Status:* Deferred; enabling path documented.

**D3 — Extraction stays on Document AI (non-generative), not Gemini vision.**
*Decision:* Keep prescription extraction on **Document AI**, declining the earlier suggestion to switch to Gemini multimodal vision for direct structured-JSON extraction. And correct the framing: Document AI is **not** "deterministic" in the strict sense.
*Why:* Two things here. First, the choice: the non-SaMD argument rests on keeping generative models out of the extraction path — Document AI is non-generative; Gemini vision is generative, and moving extraction onto it would relocate the parse across the boundary the product works hardest to keep clean. Second, the correction the second review rightly pressed: OCR/layout output *varies* by processor version, image quality, and configuration, so calling it "deterministic" overstates the property. The accurate claim is narrower and stronger: **Document AI is a non-generative extraction dependency whose outputs are benchmarked and always treated as untrusted proposals; it is not a source of confirmed clinical truth.** The safety property was never that OCR is deterministic — it's that OCR output becomes truth only through the deterministic mapper and patient confirmation. This wording is more defensible under scrutiny, and it comes with a parser-evaluation pipeline (golden de-identified corpus; field-level precision/recall/correction-rate; regression by processor version; negative fixtures for ambiguous timing, alternatives, handwriting, mixed languages, partial images; explicit fallback states for zero-items / partial / unsupported / OCR-unavailable).
*Status:* Adopted (Document AI retained; framing corrected).

**D4 — Build-time embeddings, not startup embeddings.**
*Decision:* Pre-compute knowledge-base embeddings during the CI/CD build and bundle the vectors in the container image, rather than embedding `kb.yaml` at container startup.
*Why:* Embedding at startup adds latency to every scale-from-zero event and risks Vertex AI rate-limit contention during autoscaling. The KB changes only at build time anyway, so computing vectors once in CI and shipping them in the image removes a startup dependency entirely and lets the container boot cold with no external AI call. Straight improvement, no trade-off.
*Status:* Adopted.

**D5 — Signed-URL image upload, not upload-through-container.**
*Decision:* The client uploads card images directly to a Cloud Storage bucket via a short-lived signed URL issued by the service; the Ingestor reads from Storage. Images do not transit the Cloud Run container. (Bucket uses GCP-default encryption in MVP; CMEK is D6, production bar.)
*Why:* Routing large image uploads through the container risks exhausting request concurrency and client timeouts, and forces the container to handle heavy bandwidth it doesn't need to. Signed-URL upload offloads that entirely. The full async processing pipeline (Eventarc/Tasks) that a production system would add on top of this is deferred — for a single-protocol MVP, synchronous processing behind the signed-URL upload is sufficient. The upload pattern is adopted now; the async-processing elaboration is a documented scale path.
*Status:* Upload pattern adopted; async processing deferred.

**D6 — MVP uses GCP-default encryption; CMEK is the production bar.**
*Decision:* The MVP stores patient data and card images with GCP-default encryption at rest and in transit, single-region, no application-level encryption beyond defaults. Customer-managed encryption keys (CMEK) are explicitly *not* an MVP control.
*Why:* The v0.3 GCP review recommended CMEK, and it is the right control for real patient data. But the Phase 1 requirements make a deliberate, defensible scope call: the MVP is a portfolio prototype not processing real patient data (no consent flow, no real cycle), and its stated storage posture is GCP-default encryption (Req 4.2, 5.x). Adding CMEK would push the MVP past its own declared scope while leaving the *actually* load-bearing pre-pilot control — consent — still absent, which would be incoherent. CMEK belongs with the other pre-pilot hardening (consent, KB governance, admin actor model, third-party data-flow governance) that all switch on together when real data arrives. This reverses the v0.3 addition on purpose: the review was right in general and wrong for *this* scope, and the requirements are the yardstick.
*Status:* GCP-default encryption for MVP; CMEK recorded on the NFR production bar.

**D7 — Two-tier education grounding; Tier 2 deferred behind a feature flag.**
*Decision:* The Education Agent is *designed* for two tiers — curated KB (Tier 1) and a Vertex AI Search datastore over an allowlist (Tier 2) — ordered, never merged, KB exclusive for safety domains. **In the MVP, Tier 2 ships disabled by a feature flag; the agent answers from the KB or refuses**, matching Req N5. The seam is kept so the capability isn't designed out.
*Why:* The tiering is the right *design* — it makes the authority hierarchy explicit in retrieval order rather than blending sources. But enabling Tier 2 was a scope change the requirements didn't sanction: v0.5 widened N5's "curated KB only" inside an architecture doc, which is backwards. Deferring behind a flag returns the MVP to the requirement and sends the scope question back to the requirements doc where it belongs. It also avoids taking on Tier 2's whole operating model before it's needed: crawl scheduling and freshness SLAs, source-change detection, page-level allow/block, snapshot/version metadata, citation fidelity, prompt-injection filtering on crawled pages, per-source retrieval-quality evaluation, and licensing validation (UpToDate is paid — indexing needs explicit licensing). The 60-entry KB proves the safety-critical product without any of it. When Tier 2 is enabled, a deterministic, versioned, **fail-closed** safety-domain classifier gates it: ambiguous classification routes to Tier 1 or refuses, never Tier 2, and a confidence threshold below which the agent refuses rather than synthesizes.
*Status:* Design adopted; Tier 2 disabled by flag in MVP, enabling gated on governance + requirements sign-off.

**D10 — The tiers are partitioned by clinical risk, and the partition is expressed as a tag, not a deletion.**
*Decision:* Tier 1 is narrowed to clinical content; Tier 2 crawls general patient education; the two corpora do not overlap. Tier 1 takes unconditional priority on any question both could answer. Each `kb.yaml` entry carries a `tier` field and the loader filters on it; entries leaving Tier 1 are re-tagged, never deleted.
*Why:* Two reasons, one safety and one operational. **Safety:** with overlapping corpora (the v0.6 state), the safety-domain classifier was the only thing keeping a symptom question away from crawled content, and a classifier is a single point of failure. Partitioned corpora make a classifier miss non-catastrophic, because the datastore has nothing relevant to return and the query refuses instead. Two independent barriers must both fail, not one. **Operational:** narrowing Tier 1 does not relocate a vetted entry to Tier 2, because Tier 2 is a crawl and does not contain Zigota's entries. It *substitutes* a reviewed answer with an unreviewed one. That makes each re-tagging a per-entry judgment rather than a category sweep, which is why the change must be a reviewable diff and reversible in one commit. Tagging also preserves each entry's `aliases`, which the deterministic lexical pre-filter depends on: deleting general-education entries would silently thin the pre-filter that is supposed to catch questions before any model runs.
*Carried risk:* the partition widens N5 ("medical answers come only from the curated KB"). Enablement is therefore gated on an N5 amendment in the requirements doc, scoping N5 to clinical content and Tier 2 to non-clinical education. The scope change goes through the requirements doc, not through this one. The same discipline that produced D7.
*Status:* Adopted. **Requirements v0.3 amended Block 3 (content taxonomy, 3.1, 3.2) and N5 accordingly**, so Tier 2 is now requirement-sanctioned rather than pending. The taxonomy lives in the requirements doc and this document references it; enablement remains gated on the bounded governance set in D7 (pruned allowlist, retrieved-content injection defence, origin + URL + crawl date on every Tier-2 citation).

**D11 — Escalation is an event, not a schedule-item lifecycle state.**
*Decision:* `escalated` is removed from the schedule-item state machine. A D2 match, a surfaced contact path, an unmet Type E instruction, or a trigger-timing problem records a `safetyEvent` referencing the item; the item's lifecycle is unchanged.
*Why:* As a terminal state it was a live defect, not a modelling preference. A patient reporting a concerning symptom mid-cycle would have had the referenced medication item terminated and its reminders suppressed, which is the opposite of safe. It also created an N2 problem by the back door: a count of `escalated` items is a symptom-severity tally, and the system is forbidden from grading symptoms. Splitting lifecycle (scheduling and reminding) from events (what the patient reported, what the safety layer surfaced) removes both. Requirements v0.3 records the same split in its glossary, so the two documents now share one lifecycle vocabulary.
*Status:* Adopted. Must land before the transition allowlist is codified in the implementation plan's S0.2.

**D12 — One write ledger; the patient-visible change log is a projection of it.**
*Decision:* `auditEvents` is the single append-only write ledger. Inline provenance answers lineage; the patient-facing change log is a filtered view over `auditEvents`, not a parallel array. Nothing writes to both.
*Why:* v0.6 introduced the subcollection without retiring the inline array, leaving two overlapping stores and no stated source of record. That is how audit systems quietly diverge. It also has two concrete dependents: N6's proof ("no path reaches the confirmed store except a validated command") is only checkable if every write appears in exactly one ledger, and Req 5.2a's deletion-reachability property needs a single authoritative store to traverse.
*Status:* Adopted.

**D8 — Reminders execute on Cloud Tasks + Scheduler, not in the request handler.**
*Decision:* Reminder delivery is durable infrastructure — **Cloud Tasks** for per-item delayed dispatch, **Cloud Scheduler** for periodic reconciliation — with a Cloud Run dispatch endpoint that **re-checks item state at send time** before sending the FCM push.
*Why:* The v0.4 "Reminder Scheduler reads the plan and fires FCM" was not deployable: Cloud Run scales to zero and cannot hold a timer, so the schedule cannot live in the request handler. Both reviews caught this. Cloud Tasks holds the future delivery; deterministic task IDs (`cycleId:itemId:reminderKind:scheduledAt`) make enqueue/cancel idempotent on re-timing and cancellation. Crucially, task deletion alone can't guarantee "reminders stop after confirmation" (a task may fire before deletion propagates), so the dispatch endpoint re-checks the item is still `active` and unconfirmed at send time — that recheck, not deletion, is the real guarantee (Req 2.2). The trigger cascade is three independently-cancellable tasks, each re-checking on fire; Scheduler reconciles as belt-and-braces for the mission-critical trigger. For the demo a minute-level Scheduler poll may suffice; item-level Cloud Tasks is the pilot-grade path.
*Status:* Adopted.

**D9 — "Sole writer" enforced at the database via a command API, not by convention.**
*Decision:* Clients get **read-only** Firestore access to their UID-scoped documents; **all mutations route through an authenticated Cloud Run command API** (`confirm_item`, `correct_item`, `select_alternative`, `set_anchor`, `enter_clinic_date`, `cancel_cycle`) that validates the token, authorizes the uid, checks an explicit state-transition allowlist, and writes in a transaction with server timestamps.
*Why:* "The Confirmation Gate is the sole writer" is only true if the client *cannot* write the confirmed collection directly — otherwise a bug or malicious client bypasses the gate and the non-SaMD boundary is unenforceable. The second review was right that this was still narrative. The command pattern makes it structural and *testable*: there is provably no path to the confirmed store except a validated, authenticated, transactional command, which is exactly the property N6 needs. Immutable provenance fields after creation, append-only audit events, and server-side timestamps close the loop.
*Status:* Adopted.

---

## 11. Review dispositions

Two external architecture reviews (a GCP review at v0.3, a requirements-led GCP + healthcare review at v0.5) informed the current draft. This records the triage so the doc shows judgment, not silent absorption.

**Accepted as architecture changes (folded into this draft):**
- Reminder execution on Cloud Tasks + Scheduler with send-time state recheck (D8).
- "Sole writer" enforced at the database via a command API and read-only client rules (D9).
- Transaction, idempotency, and concurrency rules for lifecycle transitions (§3.2).
- OCR reframed as a non-generative, benchmarked, untrusted-proposal dependency (D3).
- Explicit Firestore physical model: hierarchy, `auditEvents` subcollection over embedded array, Storage refs not payloads, version stamps, composite indexes (§3).
- Tier 2 deferred behind a feature flag; MVP answers from the KB or refuses (D7).
- Signed-upload hardened into a scoped session with server-derived object identity (§9).

**Accepted earlier (still standing):** build-time embeddings (D4), signed-URL upload pattern (D5), Load Balancer + Cloud Armor ingress (D1), non-streamed MVP with ingress chosen to keep streaming a config change (D2), GCP-default encryption with CMEK on the production bar (D6), Document AI over Gemini vision (D3).

**Acknowledged but held as pre-pilot gates (not MVP work), consistent with §8 and the NFR production bar:** consent flow and lawful-basis handling; CMEK; DPIA/privacy review; data-retention/deletion/backup/restore procedures and administrative (Req 5.3) deletion workflows; regional residency and BAA/ZDR vendor coverage for Vertex AI, Document AI, Speech-to-Text, FCM; formal KB medical-review governance; Tier-2 content governance; timezone/DST for routine items; accessibility and localization; admin actor model. The reviews agree these are correctly deferred for a synthetic-data portfolio prototype; the one sequencing change adopted is promoting **third-party data-flow / vendor governance to a hard go-live gate** rather than a soft pre-pilot note.

**Consciously not adopted for the MVP (scope discipline):** production-grade operational hardening the reviews raised but that would gold-plate a demo on synthetic cards — malware scanning and magic-byte validation on uploads, dead-letter queues, break-glass operator processes, formal recovery objectives. These are correct for a real system and are recorded here so their omission is a decision, not an oversight; they join the pre-pilot set when real data arrives.

**One PHI-handling principle adopted now (shapes every handler, cheap to hold from day one):** never log raw request/response bodies for chat, OCR, image, symptom, or education endpoints; use structured logs with correlation IDs and non-sensitive status only; keep operational telemetry physically separate from patient/audit data; DLP is defense-in-depth, not the primary control. This is a build-time discipline, not a deferred control.

---

## Appendix — component / GCP quick reference

```
INPUT MODALITIES       typed text ─┐
(→ Orchestrator)       card photo ──┤ (via Ingestor)
                       voice ───────┘ (via Transcriber; transcript kept, audio not)

INTAKE PIPELINE        Ingestor ──▶ Mapper ──▶ [Confirmation Gate] ──▶ confirmed
(photo→schedule)       Document AI  types+timing  sole writer         Firestore
                       item types A/B/Mon/C/D1/E · alternatives + trigger gates

TRACKER DATA CORE      State Engine (deterministic, no LLM) ◀── confirmed schedule
(no AI)                anchor+offset @ read · wake-anchored missed · re-time active
                       Protocol Model (types, phases, anchors)  + progress log
                       lifecycle: parsed → active → done/missed/cancelled
                       safety events reference items; never mutate lifecycle

REMINDERS              Cloud Tasks (delayed, idempotent) ─▶ dispatch endpoint
                       ─(recheck item state at send)─▶ FCM web push · Scheduler reconciles
                       trigger cascade: 3 cancellable tasks −2h / −lead / +1h-if-unconfirmed

CONVERSATIONAL         Tracker Agent (Gemini)   — narrate state, capture symptoms/check-ins
                       Education Agent (Gemini)  — safety-classifier → oos → Tier1 KB → refuse
                       Warning-sign matcher (D2) — deterministic symptom→KB, no LLM
    grounding tiers:   Tier 1 KB (vetted, exclusive for safety, in-MVP)
                       Tier 2 Vertex AI Search (general-ed, FLAG OFF in MVP; governance-gated)

SURFACES               Dashboard (read-only: today's plan · progress · actions-needed)
                       Chat (all interaction + state change)

WRITE ENFORCEMENT      clients read-only → Command API (token → uid → transition allowlist → txn)
                       Confirmation Gate = sole writer, DB-enforced · idempotent · server timestamps

COORDINATION           Orchestrator (Router)     — reads served · mutations → Command API

DATA PLACEMENT         patient state → Firestore (/users/{uid}/cycles/{cid}/…; auditEvents subcol)
                       curated reference → bundled (Protocol Model · KB · KB vectors)
                       externally-governed → Vertex AI Search (Tier-2, flag off)

SECURITY / EDGE        LB + Cloud Armor · Firebase Auth (Google OAuth, validated in app middleware)
                       scoped signed-upload session · no payload logging · SA least privilege
                       GCP-default encryption (CMEK = production bar)

CROSS-CUTTING          Provenance envelope (inline on every AI-derived record)
                       Secret Manager · Cloud Logging (scrubbed) · Cloud Storage (default enc)
```
