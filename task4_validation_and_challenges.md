# Task 4 — DBC Validation and Challenge Record

## Validation procedure
1. Load `vehicle_network.dbc` using `cantools`.
2. Encode known engineering values into each CAN message.
3. Decode the resulting payload.
4. Compare decoded values against the original values.
5. Check message IDs, DLC, factor, offset, unit and physical range.

Run:
```bash
python3 src/validate_dbc.py
```

Expected test vectors:
| Signal | Input | Expected raw | Expected payload bytes |
|---|---:|---:|---|
| VehicleSpeed | 65 km/h | 650 | 8A 02 |
| EngineRPM | 2450 rpm | 2450 | 92 09 |
| CoolantTemperature | 87 °C | 1270 | F6 04 |
| FuelLevel | 72 % | 144 | 90 00 |
| BatteryVoltage | 12.50 V | 1250 | E2 04 |
| AmbientTemperature | 28 °C | 680 | A8 02 |

## Challenge 1 — Raw versus decoded data
Raw view example: ID `0x100`, bytes `8A 02 00 00 00 00 00 00`.
Decoded view: `VehicleSpeed = 65.0 km/h`.
Raw data is the transmitted hexadecimal payload. Decoded data is the physical
engineering value interpreted using the matching DBC definition.

## Challenge 2 — Modify the DBC
Experiment: temporarily change the VehicleSpeed factor from `0.1` to `0.2`
without changing the transmitter. A raw value of 650 will then decode as
130 km/h instead of 65 km/h. This demonstrates that a wrong factor creates an
incorrect physical value. Restore the original factor after the experiment.
Also try changing the unit text; the numeric value stays the same but the
displayed unit becomes misleading.

## Challenge 3 — Add a new signal
AmbientTemperature is added as message ID `0x105`, start bit 0, length 16,
unsigned Intel, factor 0.1, offset -40, range -20 to 60 degC. The DBC,
transmitter and dashboard all include this signal. Verify a sample of 28 degC
decodes from raw 680 (`A8 02`).

## Challenge 4 — AI review
Review checklist:
- Unique message IDs: yes, 0x100 through 0x105.
- Signal positions fit in DLC 8: yes, bits 0–15.
- Scaling and offsets match transmitter raw conversion: yes.
- Physical ranges match the assignment: yes for the five required signals.
- Names and units are descriptive: yes.
- Unused payload bytes are reserved: yes.

## Issues and corrections
Any observed mismatch should be investigated by comparing the CAN ID, DLC,
byte order, start bit, signal length, factor and offset. Do not fix a displayed
value by changing the dashboard alone; correct the source definition or
transmitter so both agree. Record actual issues found during your own run.
