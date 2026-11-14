# Centrifugal Pump and NPSH Analysis

Operating point and cavitation assessment of a centrifugal pump on a water-transfer line, using Excel and DWSIM.

Author: Dixon Godwin, Department of Mechanical Engineering, University of Lagos (October 2026)

## What this project does

A pump's flow and its cavitation risk both depend on the pipe system it serves, not on the pump alone. This project answers two questions for a water-transfer system built around a manufacturer's pump curve:

1. At what flow and head does the pump operate on the pipe system, and how does that change with a higher static head or a dirty pipe?
2. When is cavitation expected, and how does water temperature change that?

The Excel results are checked in a DWSIM simulation, and the main failure modes are ranked in a qualitative FMEA.

## System modelled

| Item | Value |
|---|---|
| Pump | HRZL IS100-80-125C, 2900 rpm (rated 80 m³/h at 12.5 m) |
| Pipes | Raw steel, 100 mm internal diameter, roughness 4.57×10⁻⁵ m |
| Suction pipe | 5 m long, pump sits 2 m above the supply level |
| Discharge pipe | 90 m long, rises 6 m to the delivery point |
| Static head | 8 m |
| Supply and delivery | Atmospheric pressure, water at 25 °C (base case) |

## Method

- **System curve:** static head plus friction head, H = Hs + kQ². The friction constant k = 0.001093 m/(m³/h)² comes from the Darcy-Weisbach and Colebrook equations, evaluated at a reference flow of 71 m³/h (f = 0.0181, Re = 2.8×10⁵).
- **Pump curve:** third-order polynomial fitted to points read from the datasheet, within 0.07 m of the datasheet points. It is not extrapolated beyond the datasheet flow range.
- **Operating point:** where the two curves meet, found with Goal Seek in Excel.
- **NPSH:** NPSHa is calculated at six water temperatures (25 to 90 °C) using vapour pressure and density from Çengel and Cimbala Table A-3. NPSHr is read from the manufacturer's curve at 8 m³/h intervals and linearly interpolated at the operating flow. The datasheet does not state its NPSHr criterion, so it is assumed to be NPSH3 (the NPSH at which head drops by 3%).
- **DWSIM:** the same system rebuilt with the IAPWS-IF97 steam tables property package, to check pump head, delivery pressure and NPSH available against Excel.
- **FMEA:** qualitative ratings (severity, likelihood, detectability) for the failures the simulation can say something about.

## Key results

### Operating point

| Case | Flow (m³/h) | Head (m) |
|---|---|---|
| Base case (k = 0.001093) | 70.6 | 13.45 |
| Static head raised to 9 m | 66.52 | 13.84 |
| Dirty pipe (k = 0.002, f ≈ 0.033) | about 57.2 | about 14.5 |

In the base case, 8 m of the head is static lift and 5.45 m is friction. The pump runs to the left of its rated point, at about 62% efficiency (peak is about 64%). The dirty pipe cuts flow by about 19%, against about 5.8% for the extra metre of static head, because friction grows with Q².

### DWSIM check (base case, 25 °C)

- Water reaches the delivery point at 102,033 Pa, about 0.7 kPa above atmospheric, so the pump head matches what the pipes need to within about 0.07 m.
- Pump head is about 13.5 m in DWSIM against 13.45 m in Excel.
- NPSH available is 7.749 m in DWSIM against 7.746 m in Excel.

### NPSH at the operating flow (NPSHr = 3.71 m)

| Water temp (°C) | NPSHa (m) | NPSHa / NPSHr | Verdict |
|---|---|---|---|
| 25 | 7.75 | 2.09 | Adequate margin |
| 60 | 6.16 | 1.66 | Adequate margin |
| 70 | 5.02 | 1.35 | Adequate margin |
| 75 | 4.27 | 1.15 | Marginal |
| 80 | 3.37 | 0.91 | Cavitation expected |
| 90 | 1.01 | 0.27 | Cavitation expected |

Verdict rule: Adequate margin if the ratio is at least 1.3, Marginal between 1.0 and 1.3, Cavitation expected below 1.0.

The 1.0 limit follows from the NPSHr definition. **The 1.3 value is an assumed screening criterion, not a manufacturer limit or a requirement of ANSI/HI 9.6.1.** Required margins depend on the application, so the report shows how the temperature limit changes with the chosen ratio:

| Required ratio | Max water temperature |
|---|---|
| 1.0 (NPSHa = NPSHr) | about 78 °C |
| 1.1 | about 76 °C |
| 1.3 (used here) | about 71 °C |
| 1.5 | about 65 °C |

Whatever criterion is chosen, the pump has adequate margin at 25 and 60 °C and is at or beyond its limit between roughly 75 and 80 °C.

- DWSIM at 80 °C gives NPSHa of 3.38 m against 3.37 m in Excel, so both agree that cavitation is expected there.
- The cause is vapour pressure: going from 25 to 80 °C removes about 4.4 m of NPSHa while the pipe, lift and pump stay the same.

### FMEA ranking

1. Cavitation (high priority)
2. Restricted suction pipe or strainer (high)
3. Loss of prime or dry running (high, not modelled)
4. Fouled discharge pipe (medium)

## Repository contents

```
pump-npsh-analysis/
├── README.md
├── report/
│   └── Centrifugal_Pump_and_NPSH_Analysis_Simulation_Report.pdf
├── excel/
│   └── pump_npsh_analysis.xlsx
├── dwsim/
│   ├── pump_system_25C.dwxmz
│   └── pump_system_80C.dwxmz
├── python_verification/
│   ├── README.md
│   └── Pump_Hydraulic_Model_Colebrook_Clean.ipynb
└── images/
    └── (figures from the report)
```

## How to reproduce

1. Open the Excel workbook. It contains the base case, the 9 m static head case, the dirty-pipe case, the NPSHr lookup table (8 m³/h intervals) and the NPSH temperature table. Find each operating point with Goal Seek: set the difference between the pump and system heads to zero by changing the flow.
2. Open the DWSIM files. The feed flow is the Excel operating flow of 70.59 m³/h. Read NPSH Available on the pump panel and the pressure at the delivery stream.
3. For the 80 °C case, set the feed temperature to 353.15 K and keep the same volumetric flow of 70.59 m³/h.

## Assumptions and limitations

- The suction side (pump 2 m above the water, 5 m suction pipe) is an assumed layout, not measured site data.
- Only straight-pipe friction is modelled. Valves, fittings, the tank exit and any strainer are not included, so real friction and k would be higher and the real operating flow lower. The velocity head at the delivery point (about 0.3 m) is also ignored.
- Pump curve and NPSHr points were read by eye from the datasheet graph. NPSHr carries an uncertainty of about ±0.1 m, and was held at 3.1 m below 32 m³/h, where the datasheet curve starts.
- The datasheet NPSHr is assumed to be NPSH3.
- k is treated as constant, although the Colebrook friction factor varies from about 0.01905 at 40 m³/h to 0.0177 at 96 m³/h. At the operating flow the error is under 0.1%.
- The 1.3 margin ratio is an assumed screening value, and the temperature limits are linear interpolations of a curved vapour-pressure relationship.
- Excel and DWSIM use the same pump curve, so their agreement checks the pipe side of the model, not the pump data.
- FMEA ratings are engineering judgement, and no failure-rate data was used.

## Tools

Microsoft Excel (Goal Seek), DWSIM (open-source chemical process simulator), Python (NumPy, SciPy, pandas, Matplotlib) in Google Colab, SOLIDWORKS 2026.

## References

The numbering matches the report.

1. Y. A. Çengel and J. M. Cimbala, *Fluid Mechanics: Fundamentals and Applications*, 4th ed., McGraw-Hill, 2018.
2. HRZL, "Performance curve: horizontal centrifugal pump, model IS100-80-125C," manufacturer datasheet, 2025.
3. I. J. Karassik, J. P. Messina, P. Cooper, and C. C. Heald, Eds., *Pump Handbook*, 4th ed., McGraw-Hill, 2008.
4. ANSI/HI 9.6.1-2017, *Rotodynamic Pumps: Guideline for NPSH Margin*, Hydraulic Institute, 2017.
5. D. Medeiros, "DWSIM: open source chemical process simulator," version 8.8.0. https://dwsim.org
6. Çengel and Cimbala, Table A-3, Properties of saturated water.
7. IEC 60812:2018, *Failure Modes and Effects Analysis (FMEA and FMECA)*.
8. Cover image: https://www.simscale.com/blog/what-is-centrifugal-pump/

The full reference list, with access dates, is in the report.


## Integrated Study (branch: integrated-study-v2)

This branch extends the original pump and NPSH analysis into a connected pump-system study. The original Excel, DWSIM and report files are unchanged.

### Components
1. **Original hydraulic and NPSH analysis** (`excel/`, `dwsim/`, `report/`): operating point of about 70.6 m³/h at 13.45 m head, NPSH versus water temperature, and an FMEA.
2. **Python hydraulic verification** (`python_verification/`): tests the constant friction-factor approximation against the Colebrook–White equation.
3. **Mechanical arrangement (SolidWorks 2026):** supply tank, suction pipe, pump casing, discharge pipe and delivery tank. The assembly is fully defined with no interferences, and the suction pipe ends 400 mm below the water surface. It is a geometric model, not a structural simulation.

### Python verification result

| Parameter | Original model | Colebrook model |
|---|---|---|
| Operating flow | 70.5972 m³/h | 70.5877 m³/h |
| Operating head | 13.4498 m | 13.4508 m |
| Friction factor at operating point | representative constant | 0.018062 |
| Reynolds number at operating point | not applicable | 279,353 |

The operating flow differs by 0.01346%. This supports the representative friction factor near the operating point.

### Scope and limitations
- The Python work checks the friction-factor treatment only. It does not validate the pump curve, the NPSH model or the full installation.
- The pump curve is a third-order polynomial fitted to datasheet readings.
- Water properties are fixed at 25°C. Only straight-pipe friction is included, with no minor losses.
