# Mott-Macdonald-Industry-Project
# UNSW Building R13: Intelligent Boat Hull Wash Facility

> A closed-loop boat hull washing system with computer vision-driven targeting, real-time water quality monitoring, and ML-assisted pump health prediction — designed for Mott MacDonald as a Group 5 capstone project.

**Client:** Mott MacDonald  
**Scope:** Concept design for a self-contained, sensor-monitored, water-recycling wash bay for marine research vessels

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Computer Vision — Hull Dirt Detection (YOLO)](#computer-vision--hull-dirt-detection-yolo)
4. [ML Pipeline — Water Quality & Pump Monitoring](#ml-pipeline--water-quality--pump-monitoring)
5. [Hardware Stack](#hardware-stack)
6. [Closed-Loop Water Recycling](#closed-loop-water-recycling)
7. [Sustainability Outcomes](#sustainability-outcomes)
8. [Cost Summary](#cost-summary)
9. [Known Limitations & Mitigations](#known-limitations--mitigations)

---

## Project Overview

Building R13 is a 177 m² dual-purpose facility housing a **closed-loop boat wash bay** and a **two-storey research fit-out**. The wash system is fully automated: a YOLO-based computer vision model identifies fouling on the hull, targeted spray bars respond in real time, and a sensor array ensures water quality before any recirculation occurs.

The facility replaces manual hose-down procedures with a touchless, data-driven wash cycle — cutting water use, preventing stormwater contamination, and producing an auditable environmental log for regulatory reporting.

---

## System Architecture

```
┌─────────────────────────────────────────────────────┐
│                  WASH BAY (WET SIDE)                │
│                                                     │
│  [IP67 Camera] ──► [YOLO Model] ──► [Spray Control]│
│                        │                            │
│                   Hull segmented                    │
│                   into dirty zones                  │
│                        │                            │
│               Pumps target stained                  │
│               sections only                         │
│                                                     │
│  [Water Quality Sensors] ──► [Quality Gate]         │
│  pH · Turbidity · Chlorine · TDS                    │
│        │                                            │
│        ▼                                            │
│  PASS ──► Recirculate to tank                       │
│  FAIL ──► Hold for controlled disposal              │
│                                                     │
│  [Flow + Pressure Sensors] ──► [Pump Health Monitor]│
│  Catch degradation before failure                   │
└─────────────────────────────────────────────────────┘
         │
         ▼
┌───────────────────────────────┐
│  RESEARCH SIDE (DRY)          │
│  Monitoring office (upper)    │
│  Sample lab / workbench (lower│
└───────────────────────────────┘
```

---

## Computer Vision: Hull Dirt Detection (YOLO)

### What it does

A self-trained **YOLO (You Only Look Once)** object detection model processes live camera frames of the boat hull. It segments the hull surface into discrete sections and classifies each section by fouling severity — biofouling, algae staining, or clean. The spray bar controller reads this output and activates pumps only where the model detects staining.

This eliminates blanket washing entirely: water pressure is applied where it is needed, not uniformly across the hull.

### Why YOLO

YOLO was selected for its real-time inference speed. A single-pass detection architecture processes each frame in milliseconds, which matches the latency requirements of live spray bar control. Frame-by-frame segmentation allows the system to update targeting dynamically as the spray bars traverse the hull vertically.

### Model training

The model was trained on annotated images of boat hulls with labelled fouling regions. Training categories included:

- Biofouling (barnacle and organism attachment)
- Algae and silt staining
- Clean hull sections

Output is a set of bounding regions mapped to hull position coordinates, which the orchestration layer translates into spray bar activation commands.

### Inference pipeline

```
Camera frame (LUCID Triton IP67)
        │
        ▼
YOLO inference (hull segmentation)
        │
        ▼
Fouling map (dirty zones identified)
        │
        ▼
Spray bar controller
  - Activates pumps at dirty sections
  - Skips clean sections entirely
        │
        ▼
Water savings logged
```

### Honest performance bounds

| Factor | Detail |
|---|---|
| Model accuracy | ~99% — roughly 1% of reads may be incorrect |
| Lens fogging | Wet environment causes condensation mid-wash |
| Glare interference | Reflective wet hull surface can confuse detection |
| Mitigation | Yearly hydrophobic lens coating; sensor cross-checking; scheduled inspections |

---

## ML Pipeline: Water Quality & Pump Monitoring

### Water quality gate

Four parameters are measured in real time by the Hanna HI98194 multiparameter probe:

| Parameter | Role |
|---|---|
| pH | Detects chemical contamination from hull coatings |
| Turbidity | Measures suspended solids — biofouling residue |
| Chlorine | Flags disinfection byproduct buildup |
| TDS (Total Dissolved Solids) | Overall dissolved contamination load |

Decision logic uses threshold comparisons against pre-set safe-reuse limits. Water that meets all four thresholds re-enters the supply tank. Water that fails any threshold is held for controlled disposal — it never re-enters the wash circuit or reaches stormwater.

### Pump health monitoring

The IFM SF5700 sensor tracks three pump parameters each wash cycle:

- **Pressure** — deviation from baseline indicates blockage or wear
- **Flow rate** — drop in flow flags impeller degradation
- **Energy draw** — rising current at constant output signals mechanical friction

These signals are compared against per-pump baseline profiles. The system flags anomalies early, before a pump fails mid-wash and causes an uncontrolled discharge event.

### Orchestration layer

The high-level controller that ties YOLO output, water quality decisions, and pump health alerts into a unified wash cycle is identified in the design as requiring further research and development. Current implementation uses rule-based logic (thresholds and comparisons) for water and pump decisions, with YOLO providing the targeting layer independently.

---

## Hardware Stack

| Component | Unit | Cost (AUD) | Replacement Cycle |
|---|---|---|---|
| Water quality probe | Hanna HI98194 (pH, turbidity, chlorine, TDS) | $2,300 – $3,000 | Electrode yearly; unit 2–3 yrs |
| Flow + pressure sensor | IFM SF5700 | ~$970 | Every 3–4 yrs |
| Hull camera | LUCID Triton (IP67, marine-rated) | $480 – $1,200 | Every 3–5 yrs |
| **Hardware total** | | **$3,750 – $5,170** | |

All sensors are rated for salt-air environments. Inspection intervals are monthly for the camera and quarterly for flow/pressure sensors. Spares are kept on site to minimise downtime.

---

## Closed-Loop Water Recycling

100% of wash water is recycled within the facility. The loop operates as follows:

1. **Wash runs** — recycled water feeds the spray bars; only stained hull sections receive full pressure
2. **Runoff captured** — graded concrete slab channels all runoff into the bunded drain; nothing escapes to stormwater
3. **Quality checked** — pH, turbidity, chlorine, and TDS measured in real time
4. **Recirculated or held** — clean water re-enters the supply tank; contaminated water is held for disposal

A passive filtered mesh layer (316 stainless steel, 0.5–1 mm aperture) sits recessed in the slab and intercepts biofouling, algae, silt, and hull debris before water reaches the drain or recirculation feed. The mesh is a pull-out tray design, inspected and cleared each wash cycle.

Water only leaves the system via evaporation or controlled discharge when sensors confirm quality thresholds have been exceeded.

---

## Sustainability Outcomes

| Metric | Value |
|---|---|
| Wash water recycled | 100% |
| AI targeting vs blanket hose-down | Significant reduction in water use |
| AI error rate | ~1% (cross-checked by sensor data) |
| Wall lifespan vs original lining | 2–3× longer (fibre cement vs original sheeting) |
| New lifting equipment required | 0 |
| New vehicles required | 0 |

Sensor logs provide a fully auditable record of water quality and reuse volumes, supporting environmental compliance reporting.

---

## Cost Summary

| Scope | Estimate (AUD ex GST) |
|---|---|
| Wash bay fit-out (Y's scope) | ~$44,000 (range $33k–$55k) |
| Mould remediation — wet half | ~$16,000 |
| Monitoring & ML hardware | $3,750 – $5,170 |
| Filtered mesh system | $280 – $500 |

---

## Known Limitations & Mitigations

**YOLO model — lens interference**  
Wet-environment fogging and hull glare degrade detection confidence. Mitigated by hydrophobic coating applied annually and cross-validation against sensor readings.

**Sensor drift in salt air**  
316 SS and IP67-rated hardware resist corrosion, but electrochemical sensors degrade over time. Mitigated by scheduled replacement cycles and on-site spares inventory.

**Mesh blockage mid-wash**  
Heavy fouling loads could back-flood the bunded bay. Mitigated by post-wash inspection protocol and a secondary overflow path to drain.

**Biofouling disposal**  
Captured solids may contain invasive marine species. Protocol: bag and bin as solid waste — never washed back to drain or sea.

**Orchestration layer maturity**  
The unified controller that coordinates YOLO output with water quality and pump health signals is at concept stage. Rule-based logic handles current decision paths; a learned orchestration model is identified for future development.

---

*Building R13 — Concept Design · Group 5 · Mott MacDonald Collaboration*
