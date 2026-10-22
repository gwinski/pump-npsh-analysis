# Centrifugal Pump and NPSH Analysis

Operating point and cavitation assessment of a centrifugal pump on a water-transfer line, using Excel and DWSIM.

Author: Dixon Godwin, Department of Mechanical Engineering, University of Lagos (October 2026)

## What this project does

A pump's flow and its cavitation risk both depend on the pipe system it serves, not on the pump alone. This project answers two questions for a water-transfer system built around a manufacturer's pump curve:

1. At what flow and head does the pump operate on the pipe system?
2. Under what conditions does it cavitate, and how does water temperature change that?

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

- **Excel:** system curve (static head plus friction head, with the friction constant k calculated from the Darcy-Weisbach and Colebrook equations), a third-order fit of the datasheet pump curve, and the operating point found with Goal Seek. NPSHa is calculated against the datasheet NPSHr at six water temperatures.
- **DWSIM:** the same system rebuilt (IAPWS-IF97 steam tables property package) to check the pump head, the delivery pressure and NPSH available against Excel.
- **FMEA:** a qualitative failure mode and effects analysis for the failures the simulation can say something about.

## Key results

**Operating point**

| Case | Flow (m³/h) | Head (m) |
|---|---|---|
| Base case (k = 0.001093) | 70.6 | 13.45 |
| Static head raised to 9 m | 66.52 | 13.84 |
| Dirty pipe (k = 0.002) | about 57.2 | about 14.5 |

**DWSIM check (base case, 25 °C)**

- Water reaches the delivery point at 102,033 Pa, about 0.7 kPa above atmospheric, so the pump head matches what the pipes need to within about 0.07 m.
- Pump head is about 13.5 m against 13.45 m in Excel.
- NPSH available is 7.749 m in DWSIM against 7.746 m in Excel.

**NPSH at the operating flow (NPSHr = 3.71 m)**

| Water temp (°C) | NPSHa (m) | NPSHa / NPSHr | Verdict |
|---|---|---|---|
| 25 | 7.75 | 2.09 | Safe |
| 60 | 6.16 | 1.66 | Safe |
| 70 | 5.02 | 1.35 | Safe |
| 75 | 4.27 | 1.15 | Marginal |
| 80 | 3.37 | 0.91 | Cavitates |
| 90 | 1.01 | 0.27 | Cavitates |

Verdict rule: Safe if the ratio is at least 1.3, Marginal between 1 and 1.3, Cavitates below 1.

- At the operating flow, NPSHa reaches NPSHr at about 78 °C, and a ratio of 1.3 is reached at about 71 °C.
- DWSIM at 80 °C gives NPSHa of 3.38 m against 3.37 m in Excel, so both agree the pump cavitates there.
- The cause is vapour pressure: going from 25 to 80 °C removes about 4.4 m of NPSHa.

**FMEA ranking:** cavitation, then restricted suction pipe, then loss of prime (all high priority), then a fouled discharge pipe (medium).

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
└── images/
    └── (figures from the report)
```

## How to reproduce

1. Open the Excel workbook. The sheets contain the base case, the 9 m static head case, the dirty-pipe case and the NPSH temperature table. The operating point is found with Goal Seek (set the difference between the pump and system heads to zero by changing the flow).
2. Open the DWSIM files. The feed flow is set to the Excel operating flow of 70.59 m³/h. Read NPSH Available on the pump panel, and the pressure at the delivery stream.
3. For the 80 °C case, change the feed temperature to 353.15 K and keep the same volumetric flow.

## Assumptions and limitations

- The suction side (pump 2 m above the water, 5 m suction pipe) is an assumed layout, not measured site data. It has no foot valve, strainer or bends, so real suction losses would be higher.
- Pump curve and NPSHr points were read by eye from the manufacturer's datasheet graph, and NPSHr was held at 3.1 m below 32 m³/h, where the datasheet curve starts.
- The friction constant k is treated as constant, although the Colebrook friction factor varies slightly with flow.
- Excel and DWSIM use the same pump curve, so their agreement checks the pipe side of the model, not the pump data.
- FMEA ratings are engineering judgement, and no failure-rate data was used.

## Tools

Microsoft Excel (Goal Seek), DWSIM (open-source chemical process simulator).

## References

1. Y. A. Çengel and J. M. Cimbala, *Fluid Mechanics: Fundamentals and Applications*, 4th ed., McGraw-Hill, 2018.
2. Property table: Çengel and Cimbala, Table A-3, Properties of saturated water.
3. HRZL IS100-80-125C pump performance curve (manufacturer datasheet).
4. ANSI/HI 9.6.1-2017, Rotodynamic Pumps: Guideline for NPSH Margin.
5. IEC 60812:2018, Failure Modes and Effects Analysis (FMEA and FMECA).

The full reference list is in the report.
