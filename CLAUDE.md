# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Calqulus Pipeline** development workspace for Qualisys motion capture biomechanics analysis. The primary work here involves writing and editing YAML pipeline files that instruct the Calqulus processing engine what to calculate from motion capture data.

The workspace is organized as:
- `calqulus_coding_kb/calqulus-pipelines/` — YAML pipeline definitions for sports/activity analyses (running, cycling, golf, baseball, functional assessment, etc.)
- `calqulus_coding_kb/calqulus-steps/` — Reference TypeScript source and documentation for all available steps
- `calqulus_coding_kb/CALQULUS_CODING_REFERENCE.md` — Auto-generated comprehensive reference (1.2MB)

## Architecture

Pipelines are YAML files that define a graph of **output nodes** and **step nodes**:

### Output Nodes (root-level)
- `parameter` — exports time-series or scalar data to the report JSON
- `event` — exports discrete event data
- `space` — defines a custom coordinate system for data normalization

### Step Nodes (within output nodes)
Steps process data sequentially. Each step takes inputs and produces an output. Categories:
- **Aggregation**: `min`, `max`, `mean`, `median`, `std`, `range`, `sum`
- **Algorithm**: `abs`, `convert`, `derivative`, `filter`, `gapFill`, `integral`, `negate`, `power`, `root`, `round`, `sort`
- **Angle**: `jointAngle`, `angle`, `planarity`
- **Arithmetic**: `add`, `subtract`, `multiply`, `divide`
- **Data structure**: `extract`, `merge`, `append`
- **Event generator**: `peakFinder`, `threshold`, `eventMask`, `eventDuration`
- **Event utility**: `eventTime`, `refineEvent`
- **Filter**: `lowpass`, `highpass`, `bandpass`
- **Geometry**: `distance`, `dot`, `cross`, `project`
- **Import**: `import`, `segment`, `marker`
- **Kinematic**: `segmentKinematics`, `jointKinematics`
- **Logic**: `if` (with `then`/`else`)
- **Trigonometry**: `sin`, `cos`, `tan`, etc.

### Key Pipeline Concepts

**Inputs:**
- Named signals: segment/marker/event names directly (e.g., `Hips`, `LeftFoot`)
- Component access: `Hips.x`, `Hips.y`, `Hips.z`
- At-event sampling: `Hips@LTO` (values at event frames), `Hips@1` (first frame), `Hips@-1` (last frame)
- Previous step output: `$prev` or `$prev(n)` (n steps back)
- Variables: `$framerate`, `$length`, `$bodyMass`, `$field(Field Name)`
- String literals: `$"value"` or `$'value'`
- Measurement filter: `Hips?name=static*`, `LeftFoot?force=left`
- Protocols for disambiguation: `segment://Hips`, `marker://MyMarker`, `event://MyEvent`

**Outputs:**
- `output: name` — local scope (same node only)
- `export: name` — global scope (accessible anywhere, included in JSON)
- Arrow shorthand: `- import: Hips.x => hips` (single named input only)
- Output immutability: existing signals are never overwritten; nodes are skipped if outputs already exist

**Measurement filtering** on output nodes via `where:`:
```yaml
- parameter: MyParam
  where:
    name: static*          # match by measurement name (supports wildcards, case-insensitive)
    fields:
      type: MyType         # match by field value
    force: left            # match by force assignment (any/both/left/right/none)
  index: last              # pick the nth match (1-based, or first/last)
```

**Templates** (`calqulus-pipelines/templates/`) provide reusable pipeline fragments for skeleton mapping, time series, events/kinetics, and sport-specific analyses.

## Key Reference Files

- **Step documentation**: `calqulus_coding_kb/calqulus-steps/docs/nodes/steps/` — one file per step category
- **Inputs/Outputs reference**: `calqulus_coding_kb/calqulus-steps/docs/inputs-and-outputs.md`
- **Skeleton/segment names**: `calqulus_coding_kb/calqulus-steps/docs/skeleton.md`
- **Body mass**: `calqulus_coding_kb/calqulus-steps/docs/body-mass.md`
- **Pipeline examples**: `calqulus_coding_kb/calqulus-pipelines/*.calqulus.yaml`

## Coding Conventions

- Use **camelCase** for variable names, output names, and parameter names
- Only use step nodes and base nodes documented in the Calqulus-Steps docs
- Use meaningful, descriptive names for outputs and parameters
- Add comments for complex logic

## Pipeline File Structure Example

```yaml
- space: MySpace
  alignWithSegment:
    segment: Hips

- parameter: HipAngle
  steps:
    - jointAngle: [Hips, LeftUpLeg]
      output: angle
    - lowpass: angle
      cutoff: 6
    - mean:

- event: LeftFootStrike
  steps:
    - peakFinder: LeftFoot.y
      height: 0
```
