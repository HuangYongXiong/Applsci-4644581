# S9. Spatially filtered hybrid baseline (PureVec + GeoFilter + fusion)

Added in revision (reviewer request R1-9: include a hybrid RAG baseline with
strong spatial filtering). Run date: 2026-10-05, edge device of the main
evaluation (RTX 4060 Laptop, Ollama 0.34.4, qwen2.5:7b-instruct Q4_K_M).

## Design

1. **Retrieval**: identical to PureVec (dense bge-base-zh-v1.5 recall over the
   full node-text store, top-10), plus a hard spatial filter that keeps only
   recalled units located inside the officially announced affected zone of the
   query's event (implemented as a 75 km disk around the official
   intensity-zone centroid; the zone is published in the official bulletin, so
   the filter uses no oracle information unavailable at deployment time).
2. **Narrative**: the surviving evidence is passed through the *same* fusion
   stage and the *same* PROMPT_FUSION template as GraphRAG (five seeds,
   temperature 0.3), so that semantic accuracy is comparable across schemes.
3. **Judgment**: the 1,000 generated narratives (200 queries x 5 seeds) were
   appended to the blind annotation sheet and judged by the same two
   annotators under the same protocol (E3 adjudication; kappa = 0.55 on the
   new rows; overall sheet kappa 0.53).

## Results (mean +/- std over 5 seeds; populations as in Table 4/5)

| metric | PureVec | **VecFilter (hybrid)** | GraphRAG |
|---|---|---|---|
| HR@1 (78 gradable) | 0.041±0.006 | **0.041±0.006** | 0.021±0.015 |
| HR@5 (78 gradable) | 0.097±0.007 | **0.113±0.011** | 0.097±0.011 |
| MRR (78 gradable)  | 0.072±0.002 | **0.073±0.002** | 0.057±0.005 |
| SR (78 gradable)   | 0.014±0.003 | **0.008±0.003** | 0.065±0.003 |
| LMR (78 gradable)  | 0.295±0.008 | **0.295±0.008** | 0.791±0.007 |
| LP (78 gradable)   | 0.230±0.005 | **0.230±0.005** | 0.153±0.002 |
| LF1 (78 gradable)  | 0.237±0.006 | **0.237±0.006** | 0.252±0.003 |
| VCD (94 in-window, km) | 35.0±0.5 | **17.3±0.7** | 21.2±0.5 |
| IoU@25km (94 in-window) | 0.304±0.005 | **0.396±0.003** | 0.393±0.009 |
| SA (200 full)      | 0.000±0.000 | **0.068±0.008** | 0.429±0.011 |

SA (full set) = 68/1,000 blind judgments fully correct; in-window subset
0.145 (68/470); out-of-window subset 0.000 (0/530). 95% bootstrap CI for the
full-set SA: [0.054, 0.083].

## Reading

* The hard filter largely repairs viewport placement (VCD 35.0 -> 17.3 km)
  and slightly improves ranking precision (HR@5 0.097 -> 0.113).
* It cannot repair layer composition (LMR stays at the PureVec level 0.295,
  2.7x below GraphRAG) or narrative correctness (SA 0.068 vs 0.429): after
  filtering, the surviving evidence still lacks the multi-hop causal chains
  and historical case priors that the narrative requires.
* 106.8/200 queries return an empty evidence set (their events lie outside
  the materialised graph; every recalled unit falls outside the zone disk),
  mirroring the disclosed coverage boundary of all in-graph schemes.
* SR drops from 0.014 to 0.008: the filter also removes a handful of
  accidental cross-region "lucky hits" that PureVec scored on out-of-window
  events (e.g., a same-named village 596 km from the reference centre).

Files: `vecfilter_summary.json` (machine-readable), per-seed records
`vecfilter_seed{42,123,2024,7,99}.jsonl` (200 rows each; provenance fields
identical to the S5 schema; `system = "vecfilter"`).
