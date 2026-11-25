# Controller PCB: schematic plan (rev A)

> **Status 2026-11-25:** schematic captured in `pcb/besta-controller.kicad_sch` (KiCad 10, A3, ERC 0 violations). PDF and JLC BOM in `pcb/out/`. The module is the **ESP32-C3-WROOM-02-N4** (KiCad has no C3-MINI-1 symbol/footprint); it has the same GPIO set, so the pin map below is unchanged. Gate series resistors were left out (pull-downs only).
>
> **Layout 2026-11-25:** placed for the printed case (cad/controller-case) and autorouted with Freerouting 2.6.0 (best run: 0 unrouted, 61 vias). GND pours on both layers; no F.Cu pour under the module. Net classes: Power12V 1.0 mm (VIN, +12V, FAN_12V, LED1_N/LED2_N), PowerLow 0.6 mm (+3V3, BUCK_SW, VBUS, GND), default 0.2 mm. **DRC: 0 errors, 0 unconnected**, plus 55 silkscreen warnings (cosmetic, tidy before fab). Fixes applied after routing: the WROOM-02 thermal vias were opened from 0.2 to 0.3 mm drill (JLC standard), one 0.12 mm neckdown was widened to 0.2 mm, and min thermal spokes was set to 1. Case-aligned 3D: `pcb/out/besta-controller-in-case.step` (XY only) and `.stl` (XYZ, drops straight into the case).

One board design, one board per bay. It controls exactly one Bestå compartment:

- 2 fans (independent PWM, separate tach, shared switched 12 V)
- 3 temperatures on one 1-Wire bus: intake, exhaust, ambient (DS18B20 probes)
- 1 compartment LED strip (12 V, PWM-dimmed) + door reed switch
- optional: a 2nd 12 V LED channel (unit backlight, fitted on the AVR board only) and console-on sense (USB VBUS)

## Block diagram

```
 12 V barrel ─► PTC 1.5 A ─► rev-pol P-FET ─► TVS ─┬──────────────► +12V_SW rail ──► LED1 / LED2 strip (+)
   (J1)          (F1)          (Q1)       (D1)     │
                                                   ├─► P-FET high-side (Q2) ─► FAN_12V ─► J2, J3 pin 2
                                                   │        ▲ NMOS (Q3) ◄── GPIO2 FAN_PWR_EN (pull-up: ON at reset)
 USB-C (J8) VBUS ─► Schottky (D2) ─────────────────┴─► AP63203 buck (U2) ─► +3V3 ─► ESP32-C3-WROOM-02 (U1)
   D+/D- ─► USBLC6 ESD (U3) ─► GPIO19/18                                          │
                                                                                  ├─ GPIO3 ─► 2N7002 (Q4) ─► J2 PWM
                                                                                  ├─ GPIO4 ─► 2N7002 (Q5) ─► J3 PWM
                                                                                  ├─ GPIO5 ◄─ J2 TACH (10k↑, 1k series)
                                                                                  ├─ GPIO6 ◄─ J3 TACH (10k↑, 1k series)
                                                                                  ├─ GPIO1 ◄► 1-Wire J4/J5/J6 (4.7k↑)
                                                                                  ├─ GPIO20 ◄─ reed J7 (10k↑, 1k, 100 nF)
                                                                                  ├─ GPIO7 ─► AO3400A (Q6) ─► LED1 − (J9)
                                                                                  ├─ GPIO10 ─► AO3400A (Q7) ─► LED2 − (J10) [opt]
                                                                                  ├─ GPIO0 ◄─ VBUS sense divider (J11) [opt]
                                                                                  └─ GPIO8 status LED, GPIO9 BOOT, EN RESET
```

The USB-C port powers the logic for flashing and debugging without the 12 V brick. D2 OR-s VBUS into the buck input, and because 12 V > 5 V it can't back-feed USB. Fans see ~4.5 V in USB-only mode and may not spin; that's expected.

## ESP32-C3-WROOM-02 pin assignment

| GPIO | Net | Dir | Notes |
|---|---|---|---|
| 0 | VBUS_SENSE | AI | ADC1_CH0. 100k/68k divider from console USB 5 V → 2.0 V, 100 nF. DNP on the AVR board |
| 1 | ONEWIRE | I/O | 3× DS18B20, 4.7k to 3V3 |
| 2 | FAN_PWR_EN | O | **Strapping pin.** 10k pull-up keeps it high at boot (strap-safe) **and** turns fans on at reset |
| 3 | FAN1_PWM | O | → 2N7002 gate, 100 Ω series, 10k pull-down. Open drain, **inverted** in firmware |
| 4 | FAN2_PWM | O | same as FAN1 |
| 5 | FAN1_TACH | I | 10k to 3V3 at the header side, 1k series into the pin |
| 6 | FAN2_TACH | I | same |
| 7 | LED1_PWM | O | → AO3400A gate, 100 Ω, 100k pull-down (strip off at boot) |
| 8 | STATUS_LED | O | **Strapping pin.** LED + 1k to 3V3 (active low) plus a 10k pull-up so it stays high at boot |
| 9 | BOOT | I | **Strapping pin.** Button to GND (the module has an internal pull-up; add 10k external) |
| 10 | LED2_PWM | O | backlight channel, same as LED1, DNP except on the AVR board |
| 18 / 19 | USB_D− / USB_D+ | | native USB Serial/JTAG (flashing and logs) |
| 20 | DOOR_REED | I | U0RXD is an input at boot, so it's safe for a switch to GND. 10k pull-up, 1k series, 100 nF debounce, optional ESD diode |
| 21 | SPARE | | U0TXD (the ROM prints here at boot). Brought out to a test pad / spare header only |
| EN | EN | | 10k pull-up + 1 µF RC, RESET button to GND |

Boot-state check (with the ESP in reset or not yet configured):
- Fan PWM gates pulled low → the 2N7002s are off → the fans' internal pull-ups make PWM high → **100 %**.
- FAN_PWR_EN pulled high → fans **powered**.
- LED gates pulled low → strips **off**.

This meets the "full speed on boot / fault" rule in hardware, before any firmware runs.

## Sub-circuits

**Input protection:** J1 barrel (5.5 × 2.1 or 2.5, **pending measurement**) → F1 PTC (1.5 A hold) → Q1 AO3401A in reverse-polarity configuration (drain to the input, source to the rail, gate to GND via 10k, 10 V zener gate–source clamp) → D1 SMAJ15A TVS → bulk 47 µF/25 V.

**Buck (U2 AP63203WU-7, fixed 3.3 V, 2 A):** per the datasheet's typical application. CIN 2× 10 µF/25 V X7R 0805, BST 100 nF, L 4.7 µH (shielded, ≥2 A sat; confirm against the datasheet's 12 V→3.3 V table), COUT 2× 22 µF/10 V. Keep the SW node small and keep the ground return under the inductor solid.

**Fan power switch:** Q2 AO3401A high-side (source = +12V_SW). Gate divider 10k (G–S) / 10k (G → Q3 drain) gives Vgs ≈ −6 V, inside AO3401A's ±12 V limit. Q3 2N7002 gate ← GPIO2 (100 Ω, 10k pull-up to 3V3). 10 µF on FAN_12V.

**Fan headers J2, J3:** right-angle KK-254-style 4-pin (HC-2510-4AW, C2982043), so standard PC fan plugs fit through the case wall. Pin 1 GND, 2 FAN_12V, 3 TACH, 4 PWM. The footprint is the generic 1×04 horizontal pin header: check the pad positions against the HC-2510 drawing.

**1-Wire J4 (intake), J5 (exhaust), J6 (ambient):** 3-pin JST-XH side-entry (S3B-XH-A) (GND, DQ, 3V3). All on one bus; firmware tells them apart by ROM address, and the silkscreen labels are only a reminder. One 4.7k pull-up.

**Reed J7:** 2-pin JST-XH side-entry (S2B-XH-A).

**LED outputs J9 (LED1), J10 (LED2):** JST-VH 2P side-entry (S2P-VH, 10 A); keyed differently from the XH sensor ports (+12V_SW, DRAIN). AO3400A handles ~5 A; the LED1 COB strip draws ~0.6 A. Size the LED2 traces for the backlight length (budget 2 A).

**Console sense J11:** 2-pin JST-XH side-entry (VBUS in, GND) from a USB cable plugged into the console.

**Indicators:** a power LED on 3V3 (green) and the status LED on GPIO8 (blue).

## Board
- 2-layer, **1 oz** (2 oz isn't needed: the worst case is ~3 A on the 12 V entry), solid GND pour on the bottom layer.
- 90 × 60 mm, 4× M3 holes 4 mm in from the corners. Placement done 2026-11-25; routing is pending.
- Connectors on three edges for the printed case (cad/controller-case): **north** J4 J5 J6 J7 J11 (sensors and reed), **south** J1 J2 J3 J9 J10 (power, fans, LEDs), **west** J8 USB-C. **East:** the module antenna overhangs the edge by 6.2 mm.
- Antenna end of the module overhanging the board edge or over a copper keep-out, away from the 12 V switch node and the strip cables. **The Bestå is a closed wooden box, so check Wi-Fi RSSI early.**

## Per-bay population
| Ref | Xbox | AVR | PS5 |
|---|---|---|---|
| J3 / Q5 (FAN2) | ✓ | ✓ (spare) | ✓ |
| J10 / Q7 (LED2 backlight) | DNP | ✓ | DNP |
| J11 + divider (console sense) | ✓ | DNP | ✓ |

## BOM
See `pcb/out/bom-jlc.csv` (generated from the schematic; every LCSC number was checked against a 2026-11-25 JLCPCB catalog snapshot). Open items:
- **J1** C720557 is a 5.5 × 2.0 mm DC-005 (3 A). Swap it if your brick is 2.5 mm, and check the DC-005 drawing against the generic `BarrelJack_Horizontal` footprint before layout.
- Extended-part setup fees: about 15 unique extended parts (module, buck, ESD, connectors, inductor, LEDs).

Off-board per bay: 2× NF-A12x15 PWM, 3× DS18B20 waterproof probes (Ø6 mm), 1 reed switch + magnet, a 12 V warm-white COB strip, and a barrel extension lead.
