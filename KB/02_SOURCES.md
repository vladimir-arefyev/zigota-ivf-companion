# Approved Sources (the ONLY sources you may use)

You may extract ONLY from the sources below. Any fact not traceable to one of these must be
omitted. Do not use forums, clinic marketing/blogs, social media, or your own training knowledge.

## Primary sources — fetch and read these

### Patient-education & guideline bodies (anchor sources)
- ASRM / reproductivefacts.org — patient booklet:
  https://www.reproductivefacts.org/news-and-publications/fact-sheets-and-infographics/assisted-reproductive-technologies-booklet/
- ASRM / reproductivefacts.org — patient journey:
  https://www.reproductivefacts.org/patient-journeys/in-vitro-fertilization-treatment/
- SART — patient guide to ART:
  https://www.sart.org/patients/a-patients-guide-to-assisted-reproductive-technology/
- NHS — IVF:
  https://www.nhs.uk/tests-and-treatments/ivf/
- HFEA — https://www.hfea.gov.uk/  (IVF process, protocol names, luteal support wording)
- Mayo Clinic — IVF:
  https://www.mayoclinic.org/tests-procedures/in-vitro-fertilization/about/pac-20384716
- MedlinePlus — IVF:  https://medlineplus.gov/ency/article/007279.htm
- MedlinePlus — ART:  https://medlineplus.gov/assistedreproductivetechnology.html
- UpToDate — IVF (Beyond the Basics):
  https://www.uptodate.com/contents/in-vitro-fertilization-ivf-beyond-the-basics

### Clinic-specific source (for region-specific fields only)
- AVA-Peter / Scandinavia clinic ART patient card (provided as a project file).
  Use ONLY for: the clinic's own emergency/contact numbers (for `warning.verbatim_text`),
  and to confirm which drugs/instructions a real card lists. Do NOT treat it as a source of
  general medical fact — those come from the anchor sources above.

## Precedence & confidence rules
- When anchor sources AGREE on a fact → `confidence: agreed`; cite one or two representative sources.
- When they give DIFFERENT numbers/timings (e.g. trigger-to-retrieval window, beta-hCG timing,
  cycle length, stimulation duration) → `confidence: ranged`; record the full range and attribute
  EACH figure to its source. Never average.
- When only ONE source states a fact → `confidence: single_source`; cite it.
- For `warning.verbatim_text` (the redirect the patient is told to act on): use the AVA-Peter
  clinic's OWN contact details. Guideline sources back the SYMPTOM LIST (`body`, `match_patterns`);
  the clinic backs the REDIRECT. Do not tell the patient to call a foreign emergency number
  (e.g. 999) — use the clinic's numbers.

## Access fallback
If a URL is paywalled, blocked, errors, or will not load (UpToDate and SART have done this before):
do NOT fill the gap from your own knowledge. Fall back to the quotes for that source already
captured in the research report file, and mark `source.access: from_report` on the entry.
If neither the URL nor the report covers a needed fact — omit it.

## Source dates (note currency; record in `review.source_dated`)
Mayo reviewed 2023; NHS reviewed 2025; MedlinePlus 2026; HFEA 2016 (ongoing);
ASRM booklet is the 2015/2018 patient edition — flag figures that may be superseded.
