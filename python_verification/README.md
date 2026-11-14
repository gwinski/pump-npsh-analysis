# Python Hydraulic Verification

This folder contains the Python-based computational verification of the
constant Darcy friction-factor approximation used in the original pump
system hydraulic analysis.

The analysis uses the Colebrook–White equation to calculate a
flow-dependent friction factor and reassess the system curve and
pump operating point.

This is an extension of the original pump-system project, not a
validation of the entire pump installation.
**Note on the pump curve:** the 13 head values in the notebook were generated from the third-order polynomial fitted to the datasheet readings, so they are not independent manufacturer data. The original datasheet readings are in the Excel workbook of the original project.
