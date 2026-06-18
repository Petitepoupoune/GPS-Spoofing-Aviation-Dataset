# GPS Spoofing Synthetic Dataset for Commercial Aviation

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/)
[![Institution](https://img.shields.io/badge/Institution-Polytechnique%20Montréal-003DA5)](https://www.polymtl.ca/)

This repository contains a labelled synthetic dataset of commercial aviation flight trajectories enriched with GPS spoofing scenarios, produced as part of a doctoral research project on real-time GPS spoofing detection at **Polytechnique Montréal**.

> **Citation:** If you use this dataset in your research, please cite:
> ```
> Sutre, N., Zrelli, R., Gohring de Magalhaes, F., & Nicolescu, G. (2025).
> LLM-Guided Synthetic Data Generation for GPS Spoofing Detection in Commercial Aviation.
> Polytechnique Montréal.
> ```

---

## Dataset Overview

| Property | Value |
|---|---|
| Total flights | 100 base trajectories |
| Aircraft types | A320, A321, B737, B737 (X-Plane), CRJ9, E190 |
| Normal instances | 150 (3 phases × 50 flights + 100 additional) |
| Spoofed instances | 1 350 (50 flights × 3 types × 3 severities × 3 phases) |
| **Total instances** | **1 500** |
| Spoofing types | LL (position), ZL (timing), GA (Doppler velocity) |
| Severity levels | Low, Medium, High |
| Flight phases | Takeoff, Cruise, Landing |
| Expert inter-rater agreement | κ = 0.81 |

---

## Repository Structure

```
GPS-Spoofing-Dataset/
│
├── A320/
│   ├── Data_A320_1_spoofed.xlsx       # Spoofed flight (sheets: 4 ref + 27 spoofed)
│   ├── ...
│   ├── Data_A320_10_spoofed.xlsx
│   ├── Data_A320_1.xlsx               # Normal flight (reference only)
│   ├── ...
│   └── Data_A320_20.xlsx
│
├── A321/
│   ├── Data_A321_1_spoofed.xlsx
│   ├── ...
│   └── Data_A321_20.xlsx
│
├── B737/
│   ├── Data_B737_1_spoofed.xlsx
│   ├── ...
│   └── Data_B737_20.xlsx
│
├── B737_xplane/
│   ├── Data_B737_xplane_1_spoofed.xlsx
│   ├── ...
│   └── Data_B737_xplane_10.xlsx
│
├── CRJ9/
│   ├── Data_CRJ9_1_spoofed.xlsx
│   ├── ...
│   └── Data_CRJ9_20.xlsx
│
├── E190/
│   ├── Data_E190_1_spoofed.xlsx
│   ├── ...
│   └── Data_E190_20.xlsx
│
└── README.md
```

**Naming convention:**
- `Data_<Aircraft>_<N>_spoofed.xlsx` — flight with synthetic spoofing scenarios (N = 1–10)
- `Data_<Aircraft>_<N>.xlsx` — normal reference flight (N = 1–20)

---

## File Structure (Excel Workbooks)

Each `_spoofed.xlsx` file contains **31 sheets**:

| Sheet name | Type | Content |
|---|---|---|
| `Entire flight` | Reference | Full trajectory, non-spoofed |
| `Takeoff` | Reference | Takeoff phase, non-spoofed |
| `Cruise` | Reference | Cruise phase, non-spoofed |
| `Landing` | Reference | Landing phase, non-spoofed |
| `Takeoff_LL_low/medium/high` | Spoofed | Position spoofing, 3 severity levels |
| `Takeoff_ZL_low/medium/high` | Spoofed | Timing spoofing, 3 severity levels |
| `Takeoff_GA_low/medium/high` | Spoofed | Doppler velocity spoofing, 3 severity levels |
| `Cruise_[type]_[level]` | Spoofed | Same 9 combinations for cruise phase |
| `Landing_[type]_[level]` | Spoofed | Same 9 combinations for landing phase |

Each normal `Data_<Aircraft>_<N>.xlsx` file contains **4 sheets**: `Entire flight`, `Takeoff`, `Cruise`, `Landing`.

---

## Columns

| Column | Unit | Source | Spoofable |
|---|---|---|---|
| `_zulu,_time` | Decimal hours | GPS | ✓ (ZL attacks) |
| `local,_time` | Decimal hours | FMS avionics | ✗ |
| `Vtrue,_ktas` | Knots | Pitot-static | ✗ |
| `Vtrue,_ktgs` | Knots | GPS Doppler | ✓ (GA attacks) |
| `__lat,__deg` | Decimal degrees | GPS | ✓ (LL attacks) |
| `__lon,__deg` | Decimal degrees | GPS | ✓ (LL attacks) |
| `__alt,ftmsl` | Feet MSL | Barometric | ✗ |
| `phase` | — | Synthetic | Attack phase: `alignment` / `replay` / `drift` |

> **Time format:** Decimal hours. Example: `15.51694` = 15 h 31 min 01 s.  
> Conversion: `H + M/60 + S/3600`

---

## Spoofing Types

### LL — Position Channel Spoofing (RF GPS)
Manipulates GPS latitude and longitude. The ground speed column (`Vtrue,_ktgs`) is systematically recomputed from the spoofed positions via the haversine formula to maintain internal signal consistency.  
**Detection indicator:** GS/TAS discrepancy; anomalous position rate-of-change.

### ZL — Timing Channel Spoofing (RF GPS)
Manipulates the GPS UTC clock (`_zulu,_time`) while the FMS local time (`local,_time`) remains intact.  
**Detection indicator:** Growing desynchronisation between zulu and local time; anomalous ground speed derived from compressed/dilated Δt.

### GA — Doppler Velocity Channel Spoofing
Models injection of a bias on the GPS Doppler velocity observable, corrupting `Vtrue,_ktgs` without necessarily altering the pseudorange position solution. Positions and UTC time are unchanged.  
**Detection indicator:** Sustained GS/TAS discrepancy inconsistent with normal wind conditions.

---

## Three-Phase Attack Model

Each spoofed scenario follows a structured temporal sequence:

1. **Alignment** — Injected signal synchronises with the legitimate GPS signal. No data modification. Marked `alignment` in the `phase` column.
2. **Replay** — Injection begins with a subtly offset signal. Marked `replay`.
3. **Drift** — Bias accumulates progressively, steering the corrupted parameter away from truth. Marked `drift`.

Rows outside the attack window are marked `normal` in the `phase` column.

---

## Data Sources

- **FlightRadar24** (Gold subscription): 90 real-world trajectories across A320, A321, B737, CRJ9, E190 aircraft types
- **X-Plane 12 simulation** ([Arts, 2024](https://zenodo.org/records/11126713)): 10 Boeing 737-800 trajectories (DLR dataset, Frankfurt, 1 Hz)
- **SRTM terrain data**: AGL → MSL altitude conversion
- **OurAirports**: Official airport elevations for ground phases
- **ERA5 (ECMWF)**: Wind field data for True Airspeed reconstruction
- **Spoofing scenarios**: Generated by ChatGPT (GPT-4.5) guided by physics-constrained prompts, curated after expert evaluation (κ = 0.81)

---

## Usage Example

```python
import pandas as pd

# Load a spoofed flight
xl = pd.ExcelFile("A320/Data_A320_1_spoofed.xlsx")
print(xl.sheet_names)  # list all 31 sheets

# Read cruise ZL medium spoofing
df = pd.read_excel("A320/Data_A320_1_spoofed.xlsx", sheet_name="Cruise_ZL_medium")
df.columns = df.columns.str.strip()

# Compare zulu vs local time offset (ZL detection signal)
df["zulu_local_diff_s"] = (df["_zulu,_time"] - df["local,_time"]) * 3600
print(df[["_zulu,_time", "local,_time", "zulu_local_diff_s", "phase"]].head(10))
```

---

## License

This dataset is released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).  
You are free to share and adapt the material for any purpose, provided appropriate credit is given.

---

## Contact

**Nina Sutre** — PhD candidate, Département de génie informatique et génie logiciel  
Polytechnique Montréal  
nina.sutre@etud.polymtl.ca

Supervisor: **Prof. Gabriela Nicolescu** — gabriela.nicolescu@polymtl.ca
