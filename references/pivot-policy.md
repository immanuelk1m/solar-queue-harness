# Research pivot policy

Use this reference when the user asks to pivot the research axis, escape a failing chronology, or propose experiments that are meaningfully independent from the current lineage.

## Independence test

Treat a candidate as incremental and reject it from a pivot set when its main change is only one of these:

- hyperparameter tuning
- chronology robustness or fold reshuffling
- residual correction on the same target representation
- gating thresholds or gating architecture around the same signal
- ensemble weighting or member selection
- calibration-only changes
- another local feature variant that preserves the same information source and prediction object

A valid pivot should materially change at least one of:

- information representation available to the model
- prediction target or target decomposition
- physical problem definition
- source-to-observation mapping
- event representation, including arrival, duration, shape, or phase
- latent state being reconstructed before forecasting
- causal or physical intermediate quantity predicted before the final solar-wind target

## Candidate construction

Ground every pivot in the latest measured failure mode. For each candidate, specify:

1. Current failure it addresses.
2. New information representation or physical state introduced.
3. What is predicted first and how it maps to the official target.
4. Why the candidate is independent from recent failed experiments.
5. Minimal decisive experiment.
6. Official metric and success threshold.
7. Failure condition that should terminate the branch.

When the sheet shows that an oracle component is valuable but the real-time proxy cannot predict it, prefer pivots that reconstruct that component from information available before forecast issuance.

Source-side arrival and shape reconstruction for HSS/CIR events is an example of a valid pivot when it replaces direct same-lineage target correction with an upstream physical intermediate representation. Do not automatically select it; compare it against the latest evidence in the live sheet.

## Ranking

Rank candidates primarily by expected official-MSE impact and strength of evidence that the current bottleneck is addressable. Use independence from the failed lineage as a tie-breaker. Implementation cost may be reported but must not be used as an exclusion rule unless a required input is unavailable.

If the user asks for a single pivot, return one. If the user asks for multiple pivots, return no more than the requested count and avoid near-duplicates.
