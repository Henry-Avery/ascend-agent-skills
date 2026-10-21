---
name: ascend-profile-code-review
description: Review Ascend profiling against pinned framework code to identify implementation costs, active hot-path risks, and evidence for newly introduced latency. Use for profiling 对照代码、实现是否不合理、是否新增时延. Consumes existing captures; does not collect profiles or run benchmarks.
---

# Ascend profile code review

Bridge an existing profiling capture to the implementation that actually ran. Separate observed work, avoidable work, causal latency, and regression evidence. A single candidate capture can reveal costs; it cannot establish that a change introduced them or that no regression exists.

## Select the smallest useful scope

- Execution-proof summary: verify its claims and source identity, inspect fixed code, and report static hypotheses plus missing timing evidence.
- Analysis report: consume its compact summary and only the event windows needed to investigate findings.
- Raw DB/trace: if `ascend-profiling-analysis` is installed, use it for parsing, segmentation and rank analysis. Otherwise use only profiler data whose schema and provenance you can inspect, or limit the review to the available report and source. Keep large captures at their source; retrieve bounded evidence and reports.

Read [review-contract.md](references/review-contract.md) when reviewing findings. Use [report-template.md](references/report-template.md) for a saved report when useful. Reuse existing domain tools rather than building a second profiler parser.

## Establish the running implementation

Pin candidate and comparison SHAs, paired repositories, dirty patch identity, operator/native packages, actual arguments, topology, graph mode, speculative configuration and workload. Current checkout HEAD and installed version strings alone do not identify the capture.

Use exact tree diffs and call paths to distinguish upstream/base, dependency/reference, existing candidate changes, environment overlays, and new proposals. A local fork point is not automatically the PR's current upstream base. For tensor contracts or unfamiliar entrypoints, read `explain-code-path` if installed.

For each concern, record its guard and whether it was active in this capture. Distinguish initialization, metadata preparation, graph capture, graph replay and per-layer/per-step execution. A Python call used during capture need not execute during replay; its captured device work may recur. Configuration alone does not prove a particular branch executed.

## Connect code to time

1. Identify complete, comparable step windows and active/dummy ranks. Keep first request, compile/capture and capture-edge events separately identified.
2. Follow host/framework/device correlations or graph task mappings where present. Record IDs, stacks, row ranges or a reproducible DB query. Kernel names and temporal proximity alone establish a hypothesis, not a unique source-line attribution.
3. Review synchronization, copies, metadata construction, launches, allocation, collectives and stream dependencies using the contract checklist. Explain why the work exists before suggesting removal.
4. Quantify event counts, per-event distributions and observed costs for covered windows/ranks. Use interval unions for wall occupancy; preserve stream overlap and serial dependencies. Multi-rank or multi-stream duration sums are work volume, not request latency.
5. State the causal evidence needed to make the work avoidable and to place it on the request's critical path. Supply a minimal falsification plan for unresolved concerns.

When using `ascend-profiling-analysis`, its fast mode skips host-trace attribution. A fast report cannot exclude host blocking, D2H or launch overhead. Check actual host data and correlations; use full analysis or bounded host queries when appropriate. Missing host evidence remains a limitation, including after full analysis if stacks/correlations are absent.

## Classify conclusions

Use independent evidence labels:

- `static-risk`: code suggests a cost; activation or timing is unproven.
- `observed-cost`: covered events show work, with source association confidence stated.
- `causal-overhead`: evidence supports avoidable work on a serial dependency/critical path; quantify only the proven scope. This does not imply the candidate introduced it.
- `confirmed-regression`: comparable baseline/candidate evidence establishes degradation for a named metric, scope and noise threshold. Root-cause attribution may still be incomplete.

New code is not necessarily slow; existing code can be wasteful. An unobserved event, an inactive branch, a low kernel duration, or missing baseline does not prove no added latency. State “no issue observed in covered scope” only when that scope was actually reviewed, and list what it excludes.

For controlled before/after performance, read and route to `vllm-ascend-performance-regression` when installed. For required validation coverage, use `vllm-ascend-change-validation`. Read-only analysis does not authorize new workloads, profiling collection, service changes, instrumentation, or publishing. When raw data is unavailable, complete the supported static/report review and identify the exact gap without fabricating results.
