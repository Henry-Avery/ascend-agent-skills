# Review contract

## Evidence for a finding

Record only fields meaningful to the finding; retain unknown values explicitly:

| Field | Evidence expected |
|---|---|
| Source and ownership | Repository, SHA, path/symbol/line; comparison SHA and diff; upstream/dependency/candidate/overlay/proposal |
| Applicability | Guard, shape/dtype/device/topology conditions, actual branch evidence; init/build/capture/replay/layer/step frequency |
| Capture identity | Case, host/container, rank/role, file path/hash where practical, collection time and profiler settings |
| Event locator | DB query + parameters + row keys, or source event IDs/row ranges; time unit/domain, stream, step/window |
| Association | Framework/CANN/device correlations, call stack, graph mapping; name-only association marked inference |
| Cost | Count, distributions, sum of durations, interval union, overlap, host blocked time or device gap; explicit denominator |
| Causality | Dependencies/waits that place work on critical path; necessity, proposed alternative and counterevidence |
| Comparison | Same workload/environment/topology/mode/shape, warmup treatment, repetitions/noise; confounders and baseline provenance |
| Conclusion | Evidence label, association confidence, metric and covered scope, missing evidence, minimal falsification |

Prefer task identifiers over rowid when export stability is unknown. Check join cardinality before counting TASK records; one-to-many host/API relations can multiply durations. Read profiler DBs in read-only mode. SQL filters or row keys supplement rather than replace the source DB identity.

## Hot-path checks

- **Host/device boundaries:** `.cpu()`, `.item()`, `.tolist()`, Python truth or indexing of device results, blocking `.to()`, explicit synchronizations and host reads. Confirm tensor placement and actual blocking APIs; syntax alone is not proof. Moving a sync may only move the wait.
- **Allocation and layout:** `empty/zeros`, `clone`, `contiguous`, `repeat`, `cat`, dtype/device conversions, scratch growth and redundant packing. Check strides, aliases, padding and consumer requirements; a view is not automatically a copy, and a necessary copy is not automatically waste.
- **Metadata:** CPU vs device producers, once per batch vs layer, main/draft/history/current variants, refresh/reuse, graph updates and stable addresses. Count unique constructions rather than just Python call sites. Caching proposals must preserve shape/length dependence, concurrency and graph lifetime.
- **KV and speculation:** scatter/gather, history/current split, masked or padded work, accepted-token handling, rollback state copies and repeated Q packing. Verify semantic differences before merging calls or eliding writes.
- **Distributed work:** collective identity and group, useful vs dummy ranks, tensor size, barriers and stream waits. Slow communication may include waiting for upstream work. Work already present in the baseline is not a new collective introduced by the PR.
- **Graph and launches:** distinguish host wrapper launch, captured device nodes, graph update, first compilation and replay. Small kernels may amplify launch/scheduling cost; kernel duration alone misses that cost.

## Timing and comparison traps

Keep host wall time, device timestamps, kernel durations and summed work in distinct columns. Align cross-host clocks only with a documented calibration method; otherwise compare local durations or structural step IDs, not absolute timestamps across hosts. Never sum parallel ranks into end-to-end time.

Segment complete steps before dividing totals by token count or layer count. Generated tokens, scheduled tokens, padded tokens, speculative candidates and accepted tokens are different denominators. Rank skew and call-count differences require work/shape checks before inferring imbalance or missing execution.

A cold profiled request can expose costs but confounds steady-state acceptance. A single-state profile supports hypotheses or observed costs. Controlled baseline evidence supports a regression verdict; a causal source change additionally needs branch and event attribution or an isolated ablation. Profiling itself can perturb timings.

## Minimal behavioral acceptance cases

1. Only execution-proof summary: do not claim timing root cause or no regression; retain a useful static review.
2. Candidate contains a `.cpu()` under a disabled guard: list as outside captured coverage; do not blame it for this capture.
3. Overlapping streams/ranks: report work sums separately from wall occupancy; do not add them into request latency.
4. Fast report lacks host trace: do not exclude host synchronization.
5. Slow collective existed in base: attribute the existing work correctly and investigate changed prerequisites.
6. First request under profiler, no baseline: no steady-state or regression acceptance.
7. Measured A/B slowdown without stacks: regression may be confirmed while code root cause remains unknown.
