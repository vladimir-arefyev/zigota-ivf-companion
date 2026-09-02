# Self-Check (run on EVERY entry before emitting it)

For each entry, answer every question below. If any answer is "no" (or "yes" where marked ⛔),
FIX the entry or DROP it. Do not emit an entry that fails a check.

## Universal checks (all kinds)
1. Does every fact in `body` trace to a source in `source.cite`? (no → drop the unsourced fact)
2. Is `body` paraphrased, not copied verbatim, and <= ~60 words? (no → rewrite)
3. Is `scope` set correctly (in_scope vs out_of_scope)?
4. Is `confidence` set, and does it match the sources?
   - multiple sources agree → agreed
   - sources give different numbers → ranged (and the range is recorded, each attributed)
   - one source → single_source
   - out_of_scope → n/a
5. ⛔ Did you use any fact from your own knowledge that no approved source supports? (yes → remove it)
6. ⛔ Did you average, round, or collapse differing source numbers into one? (yes → restore the range)
7. Are `review.validated_by` and `review.reviewed_on` both null? (no → set them null)
8. If any URL failed to load, is `access: from_report` set on the affected cite? (no → fix)

## medication checks
9. ⛔ Is there ANY dose, unit, quantity, frequency, or schedule anywhere in the entry? (yes → remove)
10. Is `purpose` at the class level (what the class is for), not patient-specific? (no → rewrite)
11. If the drug has interchangeable forms/brands, is `alternatives_group` tagged? 
12. Are multilingual/brand names in `names` and `aliases` for retrieval?

## warning checks
13. Is `tier` set (contact_clinic or urgent_care)?
14. ⛔ Does the entry grade severity, reassure, or say "if mild/if severe" as advice? (yes → remove)
15. Does `verbatim_text` use the CLINIC'S OWN contact details (not 999/foreign numbers)?
16. Is `body` (symptom list) sourced to a guideline body, separate from the clinic redirect?

## instruction checks
17. Is `unmet_action` set (usually escalate_to_clinic)?
18. ⛔ Does it tell the patient how to self-correct a missed/mistimed item? (yes → replace with escalate)

## out_of_scope checks
19. Does `body` refuse warmly and route to the clinic, without giving the withheld guidance?
20. ⛔ Does the refusal accidentally leak the very thing it's refusing (a dose, an interpretation)? (yes → rewrite)

## stage checks
21. If sources differ on a number, is it in `value_range` with per-source attribution (not in body as one figure)?

## Final gate (whole output)
22. Are out_of_scope stubs present for every category in 03_OUT_OF_SCOPE.md section B?
23. Does the output cover all three phases and all protocol types in scope?
24. Is the output valid YAML, entries grouped by kind, and nothing else (no prose/commentary)?
