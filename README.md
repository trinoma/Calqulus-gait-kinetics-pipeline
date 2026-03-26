# Calqulus Gait Kinetics Pipeline

A [Calqulus](https://github.com/qualisys/calqulus-pipelines) pipeline for gait analysis using Qualisys motion capture and force plate data. Developed by **Trinoma R&D**.

## Repository Structure

```
grf-only-gait.calqulus.yaml     # Main pipeline file (see below)
calqulus_coding_kb/
  calqulus-pipelines/           # Reference pipelines (running, cycling, golf, etc.)
```

---

## Pipeline: `grf-only-gait.calqulus.yaml`

### What it does

Performs a complete **kinetic gait analysis using force plate data only** — no skeleton, no markers, no kinematics required. It detects stance phase events from vertical ground reaction force (GRF) signals and exports normalized GRF time series and temporal gait parameters.

### Assumptions

| Assumption | Detail |
|---|---|
| Force plate assignment | ForcePlate1 = **right foot**, ForcePlate2 = **left foot** |
| Event detection threshold | Fixed **20 N** on vertical GRF (fz) |
| Body mass | Must be set as a session field in QTM (`$bodyMass`) — required for BW normalization |

> **Warning:** If `$bodyMass` is not set in QTM before processing, GRF normalization will produce invalid (Inf/NaN) values.

### How event detection works

For each foot, the pipeline:
1. Imports the vertical force signal (`ForcePlate*.fz`)
2. Applies a **50 Hz low-pass filter** to reduce noise
3. Detects threshold crossings at **−20 N**:
   - **Foot On (LON / RON):** signal crosses below −20 N (`direction: down`)
   - **Foot Off (LOFF / ROFF):** signal crosses above −20 N (`direction: up`)
4. Masks events to the valid trial window (BOF → EOF)

### Outputs

#### Events

| Event | Description |
|---|---|
| `LON` / `RON` | Left / Right foot contact (foot on) |
| `LOFF` / `ROFF` | Left / Right foot off |
| `BOF` / `EOF` | Begin / End of file frame markers |

#### Parameters

| Parameter | Unit | Description |
|---|---|---|
| `Left GRF` / `Right GRF` | BW (body weights) | 3-component GRF (fx, fy, fz) normalized by body weight, segmented by stance cycles |
| `Left Stance Duration` / `Right Stance Duration` | seconds | Time from foot on to foot off per step |
| `Left to Right Step Time` / `Right to Left Step Time` | seconds | Time between contralateral foot strikes |
| `Left Cycle Time` / `Right Cycle Time` | seconds | Time from foot strike to next ipsilateral foot strike (full gait cycle) |

The GRF time series are annotated with stance cycle pairs (LON→LOFF / RON→ROFF), allowing the Qualisys web report to normalize the x-axis to **0–100% stance phase**.

### Compatibility

Parameter and event names are aligned with the **Qualisys web report template** for GRF-based gait analysis.

---

## Getting Started

1. Open your QTM project and ensure body mass is set as a session field.
2. Assign force plates in QTM: ForcePlate1 → right, ForcePlate2 → left.
3. Place `grf-only-gait.calqulus.yaml` in your QTM project's pipeline folder.
4. Run the Calqulus processing engine.

## References

- [Calqulus Pipelines](https://github.com/qualisys/calqulus-pipelines)
- [Calqulus Steps Documentation](calqulus_coding_kb/calqulus-steps/docs/index.md)
