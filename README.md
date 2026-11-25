# Bestå cooler

Closed-door cooling and lighting for a wall-mounted 3-bay IKEA Bestå TV unit: floor-mounted PWM fans in printed cassettes, one ESP32-C3 controller per bay, ESPHome firmware.

```
cad/        SolidCraft models + STL/STEP exports
  cassette/   fan flange + bottom scoop (one design, all bays)   ✅ v1
  baffle/     under-unit recirculation baffle                     ⏳
  ps5-riser/  platform lifting the PS5 over its fans              ⏳
  controller-case/ printed enclosure, all connectors outside       ✅ v1
pcb/        KiCad 10 project (controller, built 3×)  schematic ✅  layout ✅ (routed, DRC clean)
firmware/   ESPHome: one YAML + per-bay substitutions             ⏳
docs/       spec notes, pin map, BOM, measurements
```

## Bays
| Bay | Contents | Fans (floor) | Board extras |
|---|---|---|---|
| 1 | Xbox Series X, flat (exhaust left side) | back intake, front exhaust | console VBUS sense |
| 2 | AV receiver, PSU brick, splitter | 1 rear exhaust (2nd header spare) | backlight channel |
| 3 | PS5 launch model, flat, **on a riser over the fans** | front intake, back exhaust | console VBUS sense |

Doors stay closed; no visible vents. Extra passive intake through a shadow gap at the bottom edge of the doors. The baffle strip under the unit (10–15 mm) separates intake from exhaust.

## Mechanical
- **Square cutout (~126 mm) per fan, fan fully recessed in the floor panel** (120×15 mm, e.g. Noctua NF-A12x15 PWM).
- **Through-clamp:** the top flange and bottom scoop sandwich the panel with 4× M3 screws through the fan corners into heat-set inserts. This works on a honeycomb core.
- The same scoop serves intake and exhaust: rotate it 180° so the jets point away from each other.
- Hole placement: ≥40 mm from the back edge, ≥50 mm from side panels and cam fittings.
- PETG/ASA, matte black. Details in [cad/cassette/README.md](cad/cassette/README.md).

## Power
Mean Well GST90A12 (12 V 6.67 A) → 1-to-4 barrel splitter → extension leads → PCB barrel jack on each board. The 4th output is spare, or feeds the backlight directly if it outgrows the board's MOSFET. 5.5 mm centre-positive; **2.1 vs 2.5 mm to be confirmed.** No daisy-chaining, no hub PCB.

## Controller PCB (one design, ×3)
- 5.5 mm barrel jack, PTC fuse, reverse-polarity protection, 12→3.3 V buck (AP63203).
- **ESP32-C3-WROOM-02 module** (JLC-assembled; C3-MINI-1 had no KiCad footprint), USB-C, boot/reset buttons, status LED.
- 2× 4-pin fan headers, **independent PWM per fan** (open-drain 2N7002, inverted), separate tach (10k to 3.3 V), shared high-side P-MOSFET fan power switch (true 0 RPM), on by default at reset.
- **3× DS18B20** (intake, exhaust, ambient) on one 1-Wire bus, 4.7k pull-up. Intake/exhaust probes mount on the cassette probe clip.
- Door reed switch header (internal pull-up, optional TVS).
- **2× 12 V low-side MOSFET channels:** compartment COB strip (~0.6 A) + **plain 12 V white backlight** (fitted on the AVR board). The addressable/level-shifter/5 V LDO block was dropped.
- Optional: console power sense from USB VBUS (Xbox + PS5 boards). SHT31 footprint dropped (the ambient DS18B20 replaces it).
- Full plan, pin map and BOM: [docs/pcb-plan.md](docs/pcb-plan.md).
- M3 mounting holes, all connectors on one edge. 2-layer, solid ground plane.

## Firmware (ESPHome)
Fan duty = max(curve(ΔT exhaust−intake), curve(exhaust absolute)): min at ΔT ≤ 3 °C → 100 % at 12 °C; 100 % if exhaust ≥ 50 °C. Full speed on boot and on any sensor fault. Runs fully without Wi-Fi/HA. Fans off when the console is off and the bay is cool. Door → LED fade on 300 ms / off 1 s, 5 min timeout, HA override. Room temp from HA for monitoring only.

## Open review notes
- 2 oz copper isn't needed at <2 A per board, and it costs more at JLC; 1 oz suggested.
- A ~3 A PTC is generous for <1.5 A of load; a ~1.5 A hold is suggested.
- Console USB ports must be set to **power off in standby** (Xbox: energy-saving mode; PS5: "Supply power to USB ports: Off"), otherwise VBUS sense can't see the console's state.
- ESP32-C3 has no PCNT peripheral. ESPHome's `pulse_counter` falls back to interrupts, which is fine for fan tach.
