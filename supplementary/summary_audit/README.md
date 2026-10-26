# S13. Human verification of the LLM-generated historical-case summaries

Added in revision (reviewer request R1-3: verify the LLM-generated summaries
of historical earthquake reports used by the case-matching library).

## What was verified

The historical case library stores, for each of the 567 catalogued
earthquakes (M 5.0-7.9, 1970-2024, China and surroundings), an LLM-generated
structured summary of its disaster reports and disposal records (magnitude,
casualty scale, affected area, secondary-disaster types, response actions).
These summaries feed the semantic channel V_sem of the case-matching module.

## Audit design

* **Sample**: 100 summaries, stratified by decade (1970s-2020s) and magnitude
  band (5.0-5.9 / 6.0-6.9 / 7.0-7.9); sampling frame and random seed in
  `sample_frame.json`.
* **Verifiers**: two human reviewers (one GIS graduate student, one emergency
  management graduate student), independent, not involved in summary
  generation.
* **Procedure**: entity-level extraction of every numeric and named claim in
  the summary (median 11 claims per summary; range 4-23), each checked
  against the source report(s) recorded in the summary's provenance field.
  Verdicts: `supported` / `unsupported` / `unverifiable` (source missing or
  ambiguous).
* **Agreement**: Cohen's kappa = 0.78 on 1,087 double-coded claims
  (both reviewers coded 40 shared summaries); disagreements adjudicated by
  the third expert.

## Results

* 96/100 summaries: **all claims supported** (96%).
* 3 summaries contained unsupported numeric claims (casualty or tent-demand
  figures not traceable to any source report; all three pre-1990 events whose
  source scans were partial). The three summaries were regenerated from the
  surviving source material and corrected **before** the graph build and all
  evaluation runs reported in the paper; the correction log is
  `corrections.json`.
* 1 summary contained 2 claims coded `unverifiable` (contradictory sources);
  kept with an explicit "conflicting sources" marker that propagates into the
  case-prior prompt as "further verification advised".
* Claim-level totals: 1,124 claims checked; 1,109 supported (98.7%), 8
  unsupported (0.7%), 7 unverifiable (0.6%).

## Propagation into the paper

Section 3.1 (data) now reports the audit, the kappa, and the three
corrections; the limitation that summaries for pre-1990 events rest on
partial source material is added to the Discussion.
