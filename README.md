# Calqulus Gait Kinetics Pipeline

A [Calqulus](https://github.com/qualisys/calqulus-pipelines) pipeline for gait analysis using Qualisys motion capture and force plate data.

## Repository Structure

```text
grf-only-gait.calqulus.yaml     # Main pipeline file (see below)
```

---

## Pipeline: `grf-only-gait.calqulus.yaml`

### What it does

Performs a complete **kinetic gait analysis using force plate data only** — no skeleton, no markers, no kinematics required. It detects stance phase events from vertical ground reaction force (GRF) signals and exports normalized GRF time series and temporal gait parameters.

### Assumptions

| Assumption | Detail |
| --- | --- |
| Event detection threshold | Fixed **20 N** on vertical GRF (fz) |
| Body mass | Must be set as a session field in QTM (`$bodyMass`) — required for BW normalization |

> **Warning:** If `$bodyMass` is not set in QTM before processing, GRF normalization will produce invalid (Inf/NaN) values.

### How event detection works

For each foot, the pipeline:

1. Imports the vertical force signal (`ForcePlate*.fz`)
2. Applies a **50 Hz low-pass filter** to reduce noise
3. Detects threshold crossings at **−20 N** → stored as `LON_force` / `LOFF_force` / `RON_force` / `ROFF_force`:
   - **Foot On:** signal crosses below −20 N (`direction: down`)
   - **Foot Off:** signal crosses above −20 N (`direction: up`)
4. Masks events to the valid trial window (BOF → EOF)
5. `LON` / `RON` / `LOFF` / `ROFF` are re-exports of the `_force` events for report compatibility

### Outputs

#### Events

| Event | Description |
| --- | --- |
| `LON_force` / `RON_force` | Left / Right foot contact detected from force threshold |
| `LOFF_force` / `ROFF_force` | Left / Right foot off detected from force threshold |
| `LON` / `RON` | Re-export of `LON_force` / `RON_force` |
| `LOFF` / `ROFF` | Re-export of `LOFF_force` / `ROFF_force` |
| `BOF` / `EOF` | Begin / End of file frame markers |

#### Parameters

| Parameter | Unit | Description |
| --- | --- | --- |
| `Left GRF` / `Right GRF` | BW (body weights) | 3-component GRF (fx, fy, fz) normalized by body weight, segmented by stance cycles |

The GRF time series are annotated with stance cycle pairs (LON_force→LOFF_force / RON_force→ROFF_force), allowing the Qualisys web report to normalize the x-axis to **0–100% stance phase**.

### Compatibility

Parameter and event names are aligned with the **Qualisys web report template** for GRF-based gait analysis.

## References

- [Calqulus Pipelines](https://github.com/qualisys/calqulus-pipelines)
