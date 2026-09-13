# Zigota — Implementation Plan (Phase 3)

*How the architecture gets built: two tracks, one seam between them, and the harness that makes a multi-day cycle testable in seconds.*

**Status:** Draft v0.1 · **Author:** Vladimir Arefyev · **Format:** living document · **Companion docs:** `Zigota_Vision_Brief_2025.md`, `Zigota_Requirements_2025.md`, `Zigota_Architecture_2025.md`, `Zigota_Build_and_Content_Plan.md`

---

## How to read this document

Phase 3's exit gate is "a backlog you could start building tomorrow, front-loaded on the risky slice, with a clear MVP-demo-done definition." This document is that backlog.

It is organised as **three tracks**:

- **Track 0 — Harness.** Domain model in code, synthetic data generation, clock control, scenario replay. Built first because every other slice is untestable without it.
- **Track A — Agentic.** Everything that runs on structured protocol data: education grounding, state engine, confirmation, tracker agent, reminders, surfaces. Runs on synthetic cycles.
- **Track B — Parsing.** Photo to structured proposals. Added after Track A, plugging into a contract Track A already defines and uses.

Slices are sized **S / M / L** in relative effort, not in dates. Dates are not invented here because the weekly time budget is still an open decision (Build Plan, "What would sharpen this further"). Once that number exists, this backlog converts to a calendar in one pass; nothing else needs to change.

Each slice states: **what you build**, **what you can demo**, **exit gate**. A slice is done when the gate holds, not when the code compiles.

---

## 0. The sequencing change, and what it costs

The Build Plan put parse-and-confirm first, on the grounds that it is the highest-value and highest-risk piece. This plan puts the agentic track first. That is a real change and it deserves a stated rationale rather than a quiet reorder.

**The reason it is defensible.** In a deployment where Zigota sits alongside a clinic system, the protocol arrives as structured data and the parser never runs. Photo parsing is the fallback for the unintegrated case, not the spine. Building the spine first matches the product's actual dependency order: structured protocol in, tracked cycle out. The parser is one of at least three ways the structured protocol can arrive (photo, clinic feed, manual entry), and designing around any one of them first would bake that one in.

**What the reorder actually defers.** "Parse-and-confirm" bundles two different risks, and they separate cleanly:

| Risk | Where it lives | Sequencing |
|---|---|---|
| Does the protocol model survive a real card? | Data model | **Stays first** (Track 0, S0.3) |
| Does the lifecycle state machine hold under real transitions? | State engine, confirmation gate | **Stays first** (Track A) |
| Can OCR extract fields from a photographed card reliably? | Ingestor | Deferred to Track B |
| Do unseen clinic layouts map onto the model? | Ingestor + Mapper | Deferred, and calendar-bound (see §5.0) |

The two risks that would force an architecture rewrite are both in Track A. The deferred risks are extraction-quality risks: painful, bounded, and correctable without touching the data model. So the reorder defers the *lower* architectural risk while keeping the higher one first. The risk-first principle survives the reorder; it just gets applied to the right decomposition.

**What it costs, honestly.** Two things.

1. The Mapper's output contract gets designed against synthetic data. If real extraction later produces a shape the contract cannot express, the contract changes and Track A changes with it. **Mitigation: S0.3 below.** The synthetic data is not invented. It is hand-transcribed from the real AVA-Peter card into the Mapper's output schema. That exercise tests "does the protocol model survive a real card" without building a single line of OCR, which is the whole point.
2. Card collection has a long lead time and does not start by itself. **Mitigation: §5.0 starts now, in parallel, as a non-coding task.**

---

## 1. The seam: the Intake Contract

One artifact makes the two-track split work, and it is worth naming before the slices.

**The Intake Contract** is the JSON shape the Mapper emits and the Confirmation Gate consumes: a list of typed proposals (A / B / Mon / C / D1 / E), each with timing as `anchor + offset` or an exact moment, each with a provenance envelope, each flagged where a field could not be filled.

It does three jobs at once:

- **Track A's input.** The synthetic cycle generator emits the contract. Track A never knows whether a photo, a clinic feed, or a fixture produced it.
- **Track B's output spec.** The parser is done when it emits the contract. That is a testable target, not a vibe.
- **The parser's ground truth.** The hand-transcribed reference cycle *is* the correct answer for the real card. Parser evaluation becomes a scored diff against it, not a subjective read.

It is also the clinic-integration boundary. A clinic feed becomes a third adapter emitting the same contract, which is the "protocol comes to us as structured data" case stated as an architectural property rather than a hope. FHIR shaping stays where the architecture put it: compatible in shape, not implemented in MVP.

**Rule:** nothing downstream of the contract may read a field the contract does not define. If Track A needs something, it goes in the contract first, and Track B inherits the obligation.

---

## 2. Track overview

| Slice | Track | What it proves | Size |
|---|---|---|---|
| S0.1 | 0 | Skeleton deploys, emulator runs locally | S |
| S0.2 | 0 | Item types, lifecycle, provenance envelope exist as code | M |
| S0.3 | 0 | Real card fits the protocol model (contract v1 + reference cycle) | M |
| S0.4 | 0 | Any cycle state can be generated on demand | M |
| S0.5 | 0 | A 14-day cycle can be replayed in seconds | M |
| S0.6 | 0 | Existing safety suites run in CI against the product, not a platform | S |
| A1a | A | Corpus partitioned by clinical risk; tier tags, not deletions | M |
| A1b | A | Routing with unconditional Tier-1 priority; both gates pre-generation | M |
| A1c | A | Crawl hardening: retrieved-content injection, allowlist, citation fidelity | M |
| A2 | A | State engine resolves and transitions correctly under time | L |
| A3 | A | Confirmed records are unreachable except through a validated command | M |
| A3a | A | Manual entry and correction: patient-entered origin, no clinical autofill | M |
| A4 | A | Tracker agent narrates state and captures input, computes nothing | M |
| A5 | A | D2 escalation, reminders, and proactive prompts, verified under replayed time | L |
| A6 | A | Orchestrator + two surfaces: the demo path runs end to end | L |
| B0 | B | Card corpus and render pipeline exist (starts in parallel, early) | M |
| B1 | B | Document AI returns text + regions into `rawCaptures` | S |
| B1.5 | B | Zero-item failure, partial read, re-upload without losing confirmed items | S |
| B2 | B | Mapper emits the Intake Contract from extracted fields | L |
| B3 | B | Photo to confirmed schedule, end to end | M |
| B4 | B | Parser accuracy is measured, with the limits stated | M |

---

## 3. Track 0 — Harness

Built first. Not scaffolding to throw away: S0.4 and S0.5 are the "check progress over days" tooling, and they stay in the repo permanently as the regression substrate.

### S0.1 — Skeleton and local loop *(S)*

**Build.** One Cloud Run service, Python, modules laid out exactly as the architecture's §5 tree (empty packages are fine). Firestore emulator for local work. Firebase Auth wired. One deploy script, one environment. Build-time step that computes KB embeddings from `kb.yaml` and bakes them into the image.

**Demo.** `make dev` runs the service against the emulator; `make deploy` puts it on Cloud Run; `/healthz` answers.

**Exit gate.** A code change reaches a deployed URL in one command, and the same code runs locally against the emulator with no GCP dependency except Vertex AI. `firestore.rules` is deployed and covered by emulator rule tests from this slice onward, not retrofitted at A3: a client-write test that should fail must be failing before any code exists to bypass.

*Solo-builder note: resist the CI matrix. One pipeline: lint, run suites, build embeddings, deploy. Everything else is future-you's problem and probably never arrives.*

### S0.2 — Domain model as code *(M)*

**Build.** The shared module the architecture calls `shared/`: the seven item-type labels, the nine lifecycle states, the legal `from → to` transition allowlist as data (not as `if` statements), the provenance envelope, the `origin` distinction, `schema_version` and the four reference-version stamps.

**Demo.** A transition table you can print. An illegal transition raises.

**Exit gate.** Every state and type in Requirements §3.1 and the Protocol Model v2 is representable, and the transition allowlist is a single source consulted by every mutation path. No component defines its own state vocabulary.

### S0.3 — Reference cycle and Intake Contract v1 *(M)*

This is the highest-leverage slice in the plan.

**Build.** Take the real AVA-Peter card. Hand-transcribe every item on it into the Intake Contract: type, dose, timing as anchor+offset or exact moment, alternatives as unresolved slots, `prescribed_unscheduled` where the clinic sets the date, the hand-filled blanks as flagged-incomplete. Commit it as `fixtures/reference_cycle.json`. Where the card contains something the contract cannot express, the contract changes, and the change gets a line in the architecture's decisions log.

**Demo.** The full card, as structured data, with every gap explicitly marked as a gap rather than filled.

**Exit gate.** Every line on the card is either represented in the contract or listed in a written "deliberately not represented" section. Zero items were made expressible by quietly guessing a value.

*This is C2's lived moment, and it is also the cheapest possible test of the deferred parsing risk. If the real card breaks the model, you find out here, before any OCR exists.*

### S0.4 — Synthetic cycle generator *(M)*

**Build.** A generator that emits a cycle at any point in its life, parameterised by: protocol variant, start date, which anchors are set, which alternatives are resolved, whether the trigger has a clinic date, and a target day position. It seeds Firestore (emulator or a dev project) with proposals, confirmed items, and progress events consistent with that position.

It must be able to produce **at least one item in every lifecycle state simultaneously**, because that is the fixture the dashboard's "actions needed" region and the State Engine's edge cases both need.

**Demo.** `zigota gen --variant antagonist --day 9 --anchors wake,breakfast --trigger-dated` produces a database you can immediately query.

**Exit gate.** Every state in the S0.2 machine is reachable from the generator, and two runs with the same seed produce identical databases.

### S0.5 — Clock control and scenario replay *(M)*

The tool that answers "check progress over days."

**Build, three parts.**

1. **Injectable clock.** No `datetime.now()` anywhere in business logic. A `Clock` dependency passed into the State Engine, the reminder dispatcher, and the command API. In dev/test builds only, an override header or `/dev/clock` endpoint sets it. Gated off in prod by config, and the gate itself is tested.
2. **Scenario scripts.** A scenario is a seed plus an ordered list of `(day, time, action)`: confirm item, skip a dose, edit an anchor, enter a clinic date, report a symptom, let a reminder fire. Scenarios are YAML, live in the repo, and read like a patient's fortnight.
3. **Trace and snapshot.** Replaying a scenario emits a **trace**: after every step, the resolved dashboard state (today's plan, progress, actions-needed) plus every lifecycle transition that occurred. Traces are snapshot-tested. A state-engine change that alters a committed trace fails the build and has to be explained.

Commit a **golden trace**: a full antagonist cycle from day 1 to the beta test, covering the trigger, one missed dose corrected later, one anchor edit mid-cycle, one alternative selected late, and a clinic date entered on day 8.

**Demo.** `zigota replay scenarios/antagonist_14d.yaml --trace` prints a day-by-day narrative of the whole cycle in a few seconds.

**Exit gate.** A fortnight of cycle behaviour is verifiable without waiting a fortnight, and the wake-anchored rollover, confirmation-gated re-timing, and terminal-state immutability rules are each asserted in at least one scenario.

**Plus a coverage artifact.** `tests/requirement_map.md`: every Block 0 and Block 2 acceptance criterion mapped to the scenario and assertion that covers it, with uncovered ones listed as uncovered. Without this, "all ACs are asserted" is a claim rather than a checkable fact, and the ones that quietly go missing are predictable: 0.3 (the patient can see *which* items moved after an anchor edit), 2.3 (proactive clinic-date prompting), 2.7 (contact path reachable from every authenticated screen), 2.8 (once-daily optional check-in, skipping creates no missed state), 2.10 (Type E reported unmet), 2.11 (cancellation stops all reminders). Each of those is a behaviour, not a state, which is exactly why a state-focused trace misses them.

*Reminder testing note: the replay harness calls the dispatch endpoint directly with a simulated fire time rather than going through Cloud Tasks. That tests the part that matters (the send-time state recheck) without waiting for real queue delivery. Cloud Tasks enqueue/delete behaviour gets its own small integration test in A5.*

### S0.6 — Safety suite, ported *(S)*

**Build.** The three suites already written during the assistant comparison (content fidelity and citation, scope and refusal, injection) move into the repo and run against the product's education module rather than against three external platforms. Add the non-requirements N1 to N8 as executable cases.

**Exit gate.** `make test-safety` runs in CI and fails the build on any regression. The instruction-leak case and the multilingual disclaimer case from the earlier round are both in it as permanent regressions.

---

## 4. Track A — Agentic

### A1 — Education module: partition the corpus, then route *(L, three sub-slices)*

The running agent and the KB currently hold substantially the same content, so there is no meaningful routing to build yet. The routing becomes real once the corpora are partitioned by clinical risk: Tier 1 clinical, Tier 2 general education. That partition is the work, and most of it is content judgment rather than code.

**Why the partition matters more than the ordering.** With overlapping corpora, the safety-domain classifier is the only thing keeping a symptom question away from crawled content, and a classifier is a single point of failure. Partitioned, a classifier miss is non-catastrophic: the datastore has no clinical content to return, so the query refuses. Two independent barriers, both of which must fail. This is now recorded as D10 in the architecture.

#### A1a — Partition the corpus *(M, mostly content work)*

**Build.** Add a `tier` field to every `kb.yaml` entry; the loader filters on it. Tag as **clinical (Tier 1)**: medication and drug-safety entries, procedure entries, D2 warning signs, non-medication instructions, out-of-scope stubs, clinic contact. Tag as **general (Tier 2 territory)**: glossary, physiology, stage overviews, what-to-expect.

Three rules that make this safe:

- **Tag, never delete.** The partition lands as a reviewable diff, reverses in one commit, and an entry that turns out to be load-bearing comes back without being rewritten.
- **Re-tagging is a substitution, not a relocation.** Tier 2 is a crawl; it does not contain your entries. Dropping a glossary entry from Tier 1 replaces a reviewed answer with whatever the crawler found on NHS or ASRM. That may be better. It is still a per-entry judgment, not a category sweep.
- **Watch the aliases.** The lexical oos-filter runs off entry aliases, including cross-language ones. Re-tagging must not thin the deterministic layer that runs before any model. Measure the pre-filter's hit rate before and after, separately from answer quality.

**Demo.** The same KB loaded in two configurations, and a diff showing exactly which questions changed tier.

**Exit gate.** A hand-labelled set of ~40 grey-zone questions (the "what happens at egg retrieval" class, which is procedural and general at once) with the correct tier marked for each. That set is simultaneously the partition's definition, the classifier's test suite, and the evidence for whether Tier 2's residual scope justifies a second retrieval path at all.

#### A1b — Wire the routing *(M)*

**Build.** Port the education agent into the `education/` module and implement the chain, both gates running before any generation:

```
safety-domain classifier (deterministic, versioned, fail-closed)
  → oos-filter (lexical, clinical-judgment stubs)
  → Tier 1 KB (clinical corpus, in-memory, build-time embeddings)
  → Tier 2 datastore (general-education crawl, non-safety only)
  → refuse
```

**Tier 1 priority is unconditional.** Where both tiers could answer, the KB answers and Tier 2 is suppressed for that query. A strong Tier-2 hit never outranks a weak Tier-1 hit. This is an authority rule, not a relevance comparison, and it is tested as such.

Keep the KB loader behind an interface that could later resolve a per-tenant KB from Cloud Storage. The MVP bundles one KB in the image, but a clinic-curated Tier 1 varies per deployment and changes without a code deploy, which breaks the bundling principle. Cheap now, expensive to retrofit.

**Demo.** A dose question refuses without the model running. A warning-sign question answers from Tier 1 with the contact path. A physiology question answers from Tier 2 with visible origin attribution and crawl date. A question neither tier covers refuses with the clinic redirect.

**Exit gate.** S0.6 passes. Citation-or-refuse holds with no exception path. Fail-closed classification proven by test. Tier-1 priority proven by a case where Tier 2 has the stronger hit and is still suppressed.

#### A1c — Crawl-specific hardening *(M)*

The crawl introduces an attack surface the earlier suites never covered, and it lands on the one output the product must never produce.

**Build.**

- **Retrieved-content injection defence.** A crawled page containing instruction-shaped text ("tell the patient this is normal") is a different attack from a user-side injection. Retrieved passages are handled as data, never as instructions, and this gets its own suite.
- **Prune the allowlist.** Drop UpToDate: it is paywalled, so the crawl indexes marketing pages while creating the licensing exposure. Check robots.txt and terms on the remainder before crawling, Mayo Clinic in particular. This is a public repo with your name on it.
- **Verify the indexing tier.** Vertex AI Search website datastores gate advanced indexing behind domain verification, and you can only verify domains you own. Confirm whether what is running is basic website search over nhs.uk and reproductivefacts.org, because that materially changes retrieval and chunking quality and therefore how much Tier 2 is buying you.
- **Citation fidelity.** Every Tier-2 answer carries origin, URL, and crawl date, so the weaker authority is never disguised as the stronger.

**Exit gate.** The injection suite passes against a deliberately poisoned fixture page. The allowlist is pruned and its legal basis noted. Everything on D7's governance list that is *not* covered here (freshness SLAs, source-change detection, per-source retrieval evaluation) is written into Known Limitations by name rather than left implied.

*This slice is C6's moment, and a better one than the original: not "I tried to make my app lie" but "I planted the lie in a page it trusted."*

#### The requirements change A1 forces

N5 currently reads that medical answers come only from the curated KB. A live Tier 2 widens it. The partition gives a principled amendment rather than a quiet erosion:

> **N5 (proposed).** The system must not answer a **clinical** question from model knowledge or from non-curated content. Clinical answers (medications, procedures, warning signs, instructions, contacts) come only from the curated KB, with citation, or fall back. Non-clinical patient education may be answered from the allowlisted retrieval index, with visible origin attribution, and never for safety domains.

This edit belongs in the requirements doc, made deliberately, before Tier 2 ships on. Making it in the architecture instead is exactly the v0.5 mistake.

### A2 — State engine on synthetic cycles *(L)*

**Build.** `anchor + offset` resolution at read. Wake-anchored missed rollover (derived, not written by a job). Confirmation-gated re-timing on anchor edit. Timezone-fixed exact moments for the trigger. The three derived dashboard views. No LLM anywhere in this module.

**Demo.** Replay the golden trace. Day 1 shows resolved times; day 5 shows a missed dose appearing at the wake boundary and not at midnight; an anchor edit on day 6 moves active items and leaves the done ones frozen.

**Exit gate.** Every Block 0 and Block 2 acceptance criterion, positive and negative, has a scenario assertion. The `active`-is-the-only-mutable-state rule and the two distinct "no time yet" states are each asserted. Zero business-logic calls to system time.

### A3 — Command API and confirmation gate *(M)*

**Build.** `POST /cycles/{cycleId}/commands` with the typed command set, and the set must be **complete over every patient-data mutation**, not only over confirmed-item transitions. The architecture's list omits reminder lead time (Req 0.2). State the invariant explicitly so no later mutation escapes it:

> Every patient-data mutation goes through an authenticated, typed, idempotent command. The Confirmation Gate is the sole command family permitted to promote a proposal into confirmed schedule state.

Without that phrasing, a future client writes profile anchors or a cancellation directly to Firestore while confirmed-item writes stay protected, and the boundary leaks somewhere nobody is watching.

 Firebase token validation, uid authorisation, transition allowlist check, Firestore transaction, server timestamp, request-ID idempotency, optimistic concurrency on item version. Firestore security rules made read-only for clients. `auditEvents` as a subcollection.

**Demo.** Confirm an item from the client. Then try to write the same item directly through the client SDK and watch it fail at the rules layer.

**Exit gate.** N6 is a passing adversarial test, not a claim: no path reaches a confirmed record except a validated, authenticated, transactional command. Concurrency cases are covered (confirm racing a reminder fire, double clinic-date entry, two sessions correcting at once).

*This slice is C3's moment: the point where the guarantee stops living in the prompt and starts living in the database rules.*

### A3a — Manual entry and correction *(M)*

Req 1.4 is an MVP requirement with no owner in v0.1 of this plan. It is easy to miss because Track A seeds cycles from the generator and Track B assumes the parser produces everything, so neither track surfaces the case where the patient adds what the card-reading missed. It is also where a new failure mode lives: a clinical autofill.

**Build.** Extend the command set with `create_patient_entered_proposal` and `edit_proposal`. Both require `origin`, which is not defaulted. Patient-entered items carry the patient's verbatim input as `raw_extract` and go through the same per-item confirmation as parsed ones (1.5). The confirmation UI distinguishes "correct a clinic-sourced proposal" from "add a patient-entered item", because the two produce records with different origins and the requirements never allow those to blur.

**Demo.** Add an item the parser missed. It lands as `patient_entered`, requires per-item confirmation, and appears in the audit trail as patient-sourced.

**Exit gate.** The negative case is the point: typing a partial drug name produces **no** suggested completion of drug, dose, or timing. Req 1.4's negative AC bans the app from proposing clinical content the patient did not give, and a chat surface backed by a language model is exactly where that leaks in. One adversarial test: enter "Menopur" alone and assert nothing fills the dose.

### A4 — Tracker agent *(M)*

**Build.** Gemini reads resolved state from the State Engine and phrases it. Captures symptoms and check-ins verbatim into `progressEvents`. Tool-constrained: it can call read functions and a capture function, and nothing else. It cannot reach the command API for lifecycle changes.

**Demo.** "What's due today?" gets a narration of computed state. "Should I take 150 instead of 75?" hits the deterministic refusal before generation.

**Exit gate.** N1 to N4 and N8 hold against adversarial prompting. The agent's answers are reproducible against a fixed state fixture: same state, same substance, whatever the phrasing.

### A5 — Warning-sign matcher, reminders, and proactive prompts *(L)*

**A capability neither review named, and that this plan originally missed.** Reqs 2.3 (proactively prompt for a clinic date on a prescribed-unscheduled item) and 2.8 (initiate an optional daily check-in) both require the app to **start a conversation**. That is not a reminder and not a response to a message: it is scheduled outbound conversation. It runs on the same Cloud Tasks infrastructure as reminders but has different content, different idempotency (exactly once daily, not once per item), and a different non-failure mode (a skipped check-in must create no missed state, Req 2.8). It belongs here because it shares the infrastructure, and it needs naming because otherwise it falls between A5's reminders and A6's chat surface and nobody builds it.

**Build.** The D2 matcher as a deterministic symptom-to-KB match with a committed benchmark of canonical warning-sign phrasings, no LLM judgment. Escalation surfaces cited content plus the contact path, never a grade. Reminder enqueue on Cloud Tasks with deterministic task IDs, dispatch endpoint with send-time state recheck, trigger cascade as three independently cancellable tasks.

**Demo.** Report an OHSS-pattern symptom and get cited KB content plus the contact path, with no severity assessment anywhere in the output. Replay a scenario where a dose is confirmed two hours before its reminder fires, and watch the dispatch endpoint decline to send.

**Exit gate.** The matcher's benchmark passes and its false-negative behaviour is documented as a known limitation rather than smoothed over. N3 and N7 hold. Reminder cancellation on confirmation is proven by the send-time recheck, not by task deletion alone. A skipped check-in creates no missed state. Cancelling a cycle (Req 2.11) stops every queued send, proven at dispatch-time recheck rather than by task deletion.

**One thing the replay harness cannot prove.** Everything above is verified against the dispatch endpoint with a simulated clock, which tests the logic and not the delivery. Web push is best-effort and weakest exactly where patients are (iOS Safari PWA, which requires Add to Home Screen first). Add one manual milestone: an end-to-end FCM push to a physical phone running the PWA shell, once, with the result written into Known Limitations. Not a suite, just evidence that the path works at all before the demo depends on it.

*C5's moment sits here: the trigger's reminder cascade being visibly not-a-vitamin-reminder.*

### A6 — Orchestrator and the two surfaces *(L)*

**Build.** Intent classification routing reads to resolved state and mutations to the command API. Start with structured/keyword routing, add a model fallback only if the ambiguity rate justifies it. Dashboard as a read-only projection with Firestore listeners. Chat as the only place state changes.

Note this is a **different routing** from A1b's, and conflating them causes confusion: A1b routes a question across trust levels inside the education module; A6 routes a message across capabilities. Different problems, different failure modes, different tests.

**Demo.** The full scripted path (§7).

**Exit gate.** The dashboard renders no AI-generated text. Every state change in the demo path went through a command. The scripted path survives a cold walkthrough.

---

## 5. Track B — Parsing

### 5.0 — Start the card collection now *(non-coding, calendar-bound)*

You have one real card. That is enough to build against and not enough to evaluate against, and the gap closes on other people's schedules, not yours. So this starts in parallel with Track 0, not when Track B begins.

- Ask for de-identified prescription cards from other clinics through whatever clinical network exists (this is one of the concrete things a clinical co-founder would unlock, which is another reason that open decision matters).
- Ask patients in IVF communities for photographed, de-identified cards.
- Target: five or more distinct clinic layouts before B4 runs. Even three changes the honesty of the result.

State plainly what the eventual number proves. Synthetic renders (B0) measure robustness to image conditions. Only real cards from clinics you did not design against measure robustness to layout.

### B0 — Test data generation *(M)*

**Build.** A renderer that takes `reference_cycle.json` (and generator variants) and produces card-like documents: two or three layout templates, hand-filled blanks, checkboxes, stamps, the visual texture of a real A6 card. Then a degradation pass: perspective warp, blur, shadow gradient, JPEG artefacts, rotation, partial crop.

Each output is emitted as an **`(image, ground_truth)` pair**, the ground truth being the contract JSON it was rendered from. That makes parser evaluation a scored diff and not a manual read.

**Exit gate.** A few hundred degraded images with paired ground truth, generated in one command, plus the real card and whatever has arrived from §5.0.

### B1 — Ingestor *(S)*

**Build.** Signed-URL upload session (validate uid and cycle, mint capture ID, constrained object path, short-lived scoped URL). Document AI reads from Cloud Storage, writes text plus normalised bounding boxes into `rawCaptures` once per card.

**Exit gate.** A photographed card produces stored raw text plus regions, with the image payload never transiting the container and the server never trusting a client-supplied object URI.

### B1.5 — Extraction result policy and re-upload *(S)*

Req 1.2 is precise and none of B1, B2, or B3 owned it. It is small, and it is the first thing a cold walkthrough hits when the demo photo comes out badly.

**Build.** The zero-item decision (extraction returned nothing: tell the patient the image could not be read, offer re-upload, do not proceed to confirmation). The partial-read decision (at least one item: proceed to confirmation with gaps handled through A3a manual entry). Re-upload preserving everything already confirmed in the session.

**Exit gate.** Four executable cases: zero items blocks confirmation; one item proceeds; re-upload leaves confirmed items intact; and **no confidence threshold anywhere gates the flow**. That last one needs its own test because it is the natural thing to build by accident. Req 1.2's criterion is item count, not a score, and a well-intentioned quality gate would silently violate it.

### B2 — Mapper *(L)*

**Build.** Extracted fields to the Intake Contract: item typing, drug-class lookup against the bundled Protocol Model, timing to anchor+offset, alternatives as unresolved slots, unmappable and missing fields flagged. Deterministic: same extracted fields, same proposals. No generation, no guessing.

**Exit gate.** Run on the real card, the output diffs against `reference_cycle.json` and every difference is either a genuine parser error or a deliberate flag. Nothing was filled by inference. Feeding it deliberately corrupted extraction produces flags, never plausible-looking values.

### B3 — Photo to confirmed schedule *(M)*

**Build.** Wire B1 and B2 into the confirmation gate already built in A3. Alternatives gate, trigger's deliberate confirm, flagged-incomplete completion flow.

**Exit gate.** Photograph the real card, confirm item by item, end with a schedule identical to the A2 synthetic one. Both paths converge on the same confirmed state, which is the proof that the seam held.

### B4 — Parser evaluation *(M)*

**Build.** Scored diff over the `(image, ground_truth)` corpus: field-level precision and recall, item-type accuracy, flag rate, and the rate at which a wrong parse was caught at confirmation. Plus the free signal the architecture already identified: every `change_log` correction entry is a labelled parser error, so production corrections feed the same scoreboard without a separate study.

**Exit gate.** A number, its N, and a written statement of what it does and does not generalise to. Small-N is fine and gets said out loud.

---

## 6. Testing tools, consolidated

Five tools, four of which are built in Track 0.

| Tool | Answers | Slice |
|---|---|---|
| Cycle generator | "Give me a cycle on day 9 with an undated trigger" | S0.4 |
| Clock override | "What does the app think right now is?" | S0.5 |
| Scenario replay + trace | "What happens across a fortnight, in seconds?" | S0.5 |
| Safety suite | "Does the boundary still hold after this change?" | S0.6 |
| Parser eval harness | "How often is extraction right, and on what?" | B4 |

Two disciplines make them worth having:

- **Traces are snapshot-tested.** A change to the State Engine that silently alters day-9 behaviour fails the build. This is the only realistic way a solo builder catches temporal regressions.
- **Acceptance criteria become tests before the code exists**, for the safety-critical ones only. N1 to N8 and the Block 1 confirmation criteria get written as failing tests first. Everything else is tested after. Test-first everywhere is a discipline tax a solo build does not need; test-first on the boundary is the thing that keeps AI-assisted coding honest, because the model will happily write plausible code that erodes a guarantee it cannot see.

---

## 7. Definition of done: the scripted demo

MVP demo is done when this path runs cold, on the real card, in front of someone:

1. Sign in. Set three anchors, leave two unset.
2. Photograph the AVA-Peter card. Watch proposals appear with flags on the hand-filled blanks and an unresolved alternative slot.
3. Confirm item by item. Select one alternative. Complete a flagged field. Confirm the trigger through its separate deliberate gate.
4. See the dashboard: today's plan with resolved times, progress across three phases, actions-needed listing the two unset-anchor gaps and the undated monitoring visits.
5. Ask "what's due today?" and get narrated state. Ask "should I increase my dose?" and get a deterministic refusal.
6. Ask "what is a follicle?" and get a cited KB answer. Ask something outside the KB and get an honest fallback.
7. Report a warning-sign symptom. Get cited content plus the contact path, no grading.
8. Jump forward (clock override) to day 8. Enter the clinic's retrieval date. Watch the prescribed-unscheduled items activate.
9. Edit the breakfast anchor. Watch active items re-time and the confirmed ones stay frozen.
10. Tap any item and ask "why is this here?" Get the card region and raw OCR text it came from.

**Nine adversarial cases run alongside it**, as automated scenarios rather than live demo steps:

1. Unreadable image: no confirmation offered, re-upload preserves confirmed items (1.2).
2. Partial read: proposals appear with flags, not a blanket failure (1.2).
3. Parser-missed item added manually: `origin = patient_entered`, no suggested clinical values, per-item confirmation still required (1.4).
4. Direct client writes attempted against profile, proposals, schedule items, progress events and cancellation: all rejected at the rules layer (N6).
5. Type E instruction reported unmet: contact path surfaced, no judgment about the procedure (2.10).
6. Cycle cancelled: pending and prescribed-unscheduled items stop appearing as due, and no queued reminder sends after dispatch-time recheck (2.11).
7. Language switched mid-conversation: disclaimer, refusal wording, citation rendering and contact path all survive (this is the regression from the earlier multilingual bug).
8. Personal-protocol question: the schedule identifies the referenced item, the explanation stays cited and non-personalised (3.3).
9. Poisoned Tier-2 page containing instruction-shaped text: treated as data, never as instruction (A1c).

Steps 1 to 7 and 9 to 10 are reachable with Track A alone, substituting a generated cycle for step 2 and dropping the card region in step 10. **That is the Track-A demo**, and it is worth treating as its own milestone rather than waiting for parsing to have something showable.

---

## 8. Content moments by slice

Content lags build by one beat, and only where a genuine moment occurs.

| Beat | Slice that produces it |
|---|---|
| C2 (real card breaks the model) | S0.3 |
| C3 (guarantee moved into architecture) | A3 |
| C4 (built the scary part first, AI tooling fought me) | A2 or S0.5, whichever produces the better dead end |
| C5 (one reminder is not like the others) | A5 |
| C6 (planting a lie in a page the app trusted) | A1c |
| C7 (retrospective) | Phase 6 |

Keep the build log live from S0.1. C4 through C6 are only as good as what gets written down on the day.

---

## 9. Open decisions this plan forces

Three requirements-level decisions, all of which must land in `Zigota_Requirements_2025.md` before the slices they gate are built. The rule that produced this list: an implementation plan may not narrow a requirement. Where it wants to, it raises a decision.

### 9.1 — Block 3 and N5, before A1b *(P0)*

Settled in direction (architecture D10): corpus partitioned by clinical risk, Tier 1 unconditionally prioritised, Tier 2 live for general education. **The amendment is broader than the N5 wording drafted in A1.** Block 3.1 and 3.2 currently scope grounded answers to the curated KB, so amending N5 alone leaves the requirements internally contradictory:

- **3.1** needs the clinical / general-education distinction.
- **3.2** needs Tier-2 answers to satisfy the citation contract via origin URL plus crawl date.
- A **taxonomy** of clinical, safety, procedure and general-education must be stated once and referenced by both, because "procedure" sits inside the curated-only scope while being the archetypal grey-zone question ("what happens at retrieval"). A1a's 40-question labelled set is what makes the taxonomy testable rather than rhetorical.

### 9.2 — Voice scope, before A3a *(P0)*

The conflict is two-layered and the layers need separating.

**Sequencing:** voice is in Reqs 1.4/1.8/2.6/2.8 and absent from this plan.

**Behaviour:** the architecture says "voice never confirms or edits schedule values." Req 1.8 does not actually conflict with the intent behind that: its own AC requires the transcript to be shown back and the standard per-item confirmation to follow, so voice supplies draft text and the visual gate still holds. The architecture's blanket ban is **overstated** and should be narrowed to the rule it meant: *speech may supply reviewable draft input; it may never itself commit or confirm a schedule mutation.* That rule is worth fixing in the architecture whether or not voice ships in MVP, because otherwise it will be inherited wrongly later.

**Recommendation: formally defer.** Move 1.8 and the voice portions of 1.4, 2.6 and 2.8 into the requirements' Scope boundary as pre-pilot, with the narrowed behavioural rule retained for when they return. Text plus photo proves the paradigm, the Transcriber seam is already at the Orchestrator's front door, and Gemini audio-in makes it a modality flag later rather than new infrastructure. This is a solo-builder scope call, not a safety one, and the requirements doc is where scope calls get recorded.

### 9.3 — Block 5 scope, before B1 *(P1)*

Block 5 is framed as MVP requirements while the Scope boundary defers the "admin actor model" that 5.3 and 5.4 depend on. The requirements contradict themselves here, so the plan should split Block 5 rather than accept or defer it whole:

| Req | Status | Owner |
|---|---|---|
| 5.1 provenance linkage; raw capture retained while derived items exist | **MVP, buildable now** | S0.2 (envelope), B1 (capture retention), A3 (origin on write) |
| 5.2 no self-service deletion; history persists | **MVP, a negative** | A6 (no deletion affordance exists), tested as an absence |
| 5.3 administrative deletion | **Decision needed** | Unowned |
| 5.4 archival eligibility after one year | **Decision needed** | Unowned |

**Recommendation for 5.3/5.4: prototype position.** Build the data model so cascading deletion is *possible* (every raw capture and Storage object reachable from the UID root, no orphan paths), write a manual runbook, and downgrade 5.3/5.4 in the requirements to non-operational prototype commitments. Building an admin tooling surface for a synthetic-data demo is gold-plating; leaving a data model that makes deletion impossible later is not. The one thing that must actually be built is the traversal property, and that is a schema review, not a slice.

Carried from the architecture and still open: card-image retention policy (which 5.3's decision now constrains), orchestrator classification approach, separate audit log versus inline provenance (see §11).

## 10. What this plan does not cover

Everything in the Requirements' "Scope boundary, consciously deferred to pre-pilot" stays deferred, and this plan does not quietly re-admit any of it. Specifically not built here: consent flow, in-cycle protocol amendment, full timezone and DST semantics for anchor-relative items, auth lifecycle edges, accessibility conformance, systematic error and recovery states, data export workflow, admin actor model, KB content governance, protocol permutations beyond the reference card.

**Voice is the exception, and it is not a clean deferral.** Reqs 1.4, 1.8, 2.6 and 2.8 make voice an MVP input modality for manual entry, corrections, symptom capture and check-ins. The architecture defers it, and v0.1 of this plan followed the architecture. That silently overrode a requirement from an implementation doc, which is the same error class as the v0.5 N5 widening. See the open decision below; it must be settled in the requirements doc either way.

---

## 11. Architecture amendments this review forces

Three findings land in `Zigota_Architecture_2025.md` rather than here, because they are design errors and not sequencing ones. Recorded so they are not lost between documents.

**A11.1 — `escalated` does not belong on the schedule-item lifecycle.** §3.1 places `escalated` as a terminal sibling of `done` / `missed` / `cancelled`, which means a D2 symptom match would terminate a medication item and suppress its reminders. That is clearly not the intent, and it is the kind of error that only shows up when a patient reports a symptom on a day they still have doses to take. Escalation is an event, not a state: model it as a `safety_event` (kinds: `warning_match`, `contact_path_surfaced`, `instruction_unmet`, `trigger_timing_problem`) that *references* an item without mutating its lifecycle. Schedule lifecycle then governs only scheduling and reminders. This is a v0.8 change and it should land before A2 codifies the transition allowlist in S0.2.

**A11.2 — the audit model has two overlapping homes.** v0.6 moved the change log into an `auditEvents` subcollection, but §4's envelope still shows an inline `change_log` array. Both can exist, but their roles need naming or they become competing sources of record:

- *Inline provenance*: lineage answering "why is this item here?"
- *Item change log*: patient-visible correction history.
- *Audit events*: the security and operational write ledger (actor uid, command type, idempotency key, before/after version, authorisation outcome).

Name the source of record for each, because N6's proof and any future deletion traversal both depend on knowing which one is authoritative.

**A11.3 — the voice rule is overstated.** "Voice never confirms or edits schedule values" should read "speech may supply reviewable draft input; it may never itself commit or confirm a schedule mutation." The current wording contradicts Req 1.8 unnecessarily, since 1.8 already requires the transcript to be shown back before use. See §9.2.
