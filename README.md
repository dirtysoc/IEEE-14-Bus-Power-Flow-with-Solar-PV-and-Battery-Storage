# IEEE 14-Bus Power Flow with Solar PV and Battery Storage

A power systems study modeling the IEEE 14-bus test system in
[pandapower](https://www.pandapower.org/): run AC power flow, integrate a solar
PV plant and a battery, and analyze the impact on bus voltages and system
loading.

**Tools:** Python · pandapower · matplotlib



---

## Repository structure

```
ieee14-solar-battery/
├── README.md
├── requirements.txt
├── .gitignore
├── src/
│   ├── m1_baseline.py     # baseline power flow + bus voltages
│   ├── m2_solar.py        # add 5 MW solar (sgen) at bus 13, compare
│   ├── m3_battery.py      # battery at bus 8: discharge vs. charge
│   └── m4_visualize.py    # voltage-profile plot across all scenarios
├── figures/
│   └── voltage_profile.png
├── notebooks/             # exploratory Jupyter work
└── results/              # generated output tables
```

---

## Concepts

- A **bus** is one electrical node with a single voltage (magnitude + angle).
- Each bus has four quantities (`|V|`, angle, `P`, `Q`); power flow **fixes two
  and solves for the other two**, by bus type:
  - **Slack**: `|V|`, angle fixed; solves `P`, `Q`. Balances the whole system.
  - **PV (generator)**: `P`, `|V|` fixed; solves `Q`, angle.
  - **PQ (load)**: `P`, `Q` fixed; solves `|V|`, angle.
- **Power flow** finds the voltage at every bus so power balances everywhere;
  line flows and losses then follow.

---

## Milestones

### M1 — Baseline (`src/m1_baseline.py`)
Load IEEE 14-bus, solve, read results. Network: 14 buses, 11 loads (PQ),
4 generators (PV), 1 slack, 15 lines, 5 transformers; total load ≈ 259 MW.
The slack supplies ≈ 232 MW, with ≈ 13.4 MW of line losses. Bus angles grow
more negative with distance from the slack — power flows "downhill" in angle.

### M2 — Solar PV (`src/m2_solar.py`)
Add a 5 MW solar plant at **bus 13**, modeled as an `sgen` (injects real power,
does not regulate voltage — matches a solar inverter). **Result:** bus 13
voltage rises 1.0355 → 1.0403; neighboring buses rise slightly; the slack's
output drops (solar covers local load, unloading the system). The effect is
largest at the injection bus and fades with electrical distance.

### M3 — Battery storage (`src/m3_battery.py`)
Add a battery (`storage` element) at **bus 8** (a heavily-loaded 29.5 MW bus
whose voltage is free to move). The battery is **two-way** — the sign of
`p_mw` sets its role:
- discharging (`p_mw < 0`) injects power → bus 8 voltage **rises**;
- charging (`p_mw > 0`) draws power → bus 8 voltage **falls**.
Same device, opposite effects. This is the basis for time-shifting energy:
charge when there's surplus (midday solar), discharge at peak demand (evening).

### M4 — Visualization (`src/m4_visualize.py`)
Voltage profile across all 14 buses for four scenarios (baseline, +solar,
+solar/battery discharge, +solar/battery charge), with the ±5% limit band.
The scenario lines fan out precisely at buses 8 and 13 — the device locations —
showing the local nature of the impact. See `figures/voltage_profile.png`.

---

## How to run

```bash
pip install -r requirements.txt

python src/m1_baseline.py
python src/m2_solar.py
python src/m3_battery.py
python src/m4_visualize.py     # writes figures/voltage_profile.png
```

(The `numba` warning from pandapower is harmless; it just notes the optional
speedup isn't installed.)

---

## Key takeaway

Placing generation close to load raises local voltage and relieves the
transmission system; a battery adds *time-shifting* on top — storing surplus and
releasing it at peak. This is a small-scale version of a real planning question
grid operators (e.g. ERCOT) ask about siting distributed and inverter-based
resources.

## Possible next steps

- 24-hour time-series (a daily demand + solar curve driving battery charge/
  discharge), where the battery's energy capacity `max_e_mwh` starts to matter.
- Line-loading analysis (`net.res_line.loading_percent`) alongside voltages.
- Reactive-power / smart-inverter voltage control.
