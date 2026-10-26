# S8 — Per-run provenance logs

One record per evaluation run (100 runs over seeds {42, 123, 2024, 7, 99} × schemes):

- configuration hash (α, σ, β, γ, δ, T);
- model identifier (Qwen2.5-7B-Instruct, Q4_K_M) and decoding settings;
- timestamp and wall-clock time;
- cache-hit / fresh-call split.

> Note: drop the per-run log files (e.g. `run_*.json`) here before uploading.
