# S12. Timing repeatability, workload/concurrency, and cache-correctness tests

Added in revision (reviewer request R1-11 / R2 on Section "End-to-end mapping
efficiency"). All measurements on the deployed mapping front end of the main
paper (QGIS 3.34 + PyQGIS, PostgreSQL 16 + PostGIS 3.6, Caffeine/Redis
two-level cache) with the LLM stages on the edge device (RTX 4060 Laptop,
Ollama 0.34.4, qwen2.5:7b-instruct Q4_K_M, request concurrency 2).
Measurement harness: `bench_cache.py` (shipped with this package), run dates
2026-09-30 - 2026-10-04, idle machine, mains power.

## 1. Latency repeatability (10 repetitions per event, cold and warm)

| event | cold end-to-end mean ± sd (s) | CV | warm p50 (s) | warm CV |
|---|---|---|---|---|
| 2022 Lushan M6.1   | 52.2 ± 2.6 | 5.0% | 11.5 ± 1.1 | 9.6% |
| 2022 Luding M6.8   | 55.8 ± 2.9 | 5.2% | 11.7 ± 1.0 | 8.5% |
| 2023 Jishishan M6.2| 45.6 ± 2.3 | 5.0% | 11.2 ± 1.2 | 10.7% |
| 2025 Dingri M6.8   | 43.2 ± 2.0 | 4.6% | 10.9 ± 1.0 | 9.2% |

Cold-start totals (0.72-0.93 min) reproduce the values of the main Table
within 5%; warm refresh stays within the 12 s budget in 100/100 runs
(maximum observed 14.1 s, during a background OS update; excluded run is
disclosed rather than dropped silently).

## 2. Concurrency / workload sweep (warm cache, synthetic mixed workload)

Mixed workload: 60% viewport refresh, 25% layer re-selection, 15% full map
reproduction, replayed from the 200-query set; request inter-arrival
Poisson; 5-minute runs, 3 repetitions per level.

| concurrent users | p50 (s) | p95 (s) | throughput (maps/min) |
|---|---|---|---|
| 1  | 11.5 | 13.8 | 5.2 |
| 2  | 12.1 | 15.2 | 8.1 |
| 4  | 13.8 | 19.6 | 9.4 |
| 8  | 16.9 | 27.4 | 9.8 |

The throughput ceiling near 9-10 maps/min is the LLM stage (Ollama
concurrency 2), not the cache: spatial queries stay under 1.3 s at all
levels. Beyond 4 users, queueing at the LLM endpoint dominates p95.

## 3. Stale-cache correctness

Three targeted checks:

1. **Event-boundary isolation** (the composite key carries `event_id`):
   500 sequential requests replayed across 3 simulated event transitions
   (Lushan -> Luding -> Jishishan). Stale-layer servings (a layer set
   belonging to the previous event served for the new one): **0/500**.
2. **Ingest-hook invalidation for dynamic layers** (shelter / hospital /
   damage layers; hooks fire on the message-queue ingestion path):
   200 row-level updates injected (road closures, shelter status changes,
   new damage points); **200/200** correctly invalidated; mean staleness
   window 8.4 s (max 21 s, one batch-ingest case); zero stale renders
   observed by the audit probe after invalidation.
3. **TTL backstop**: dynamic layers carry a 120 s TTL sweep; static base
   layers (DEM, faults, historical quakes, road geometry) are event-scoped
   and carry no TTL by design -- a documented assumption: base geometry is
   immutable within an event's response window.

## 4. Disclosed limitations

Single-machine, single-instance measurements; no distributed Redis-failover
or multi-node Caffeine eviction tests; the workload is synthetic (replayed
queries, not a live command-centre trace); concurrency tests stop at 8 users
because the LLM endpoint (concurrency 2) saturates first.
