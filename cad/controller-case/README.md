# Controller case

Two-part printed enclosure for the bay controller PCB (`pcb/besta-controller.kicad_pcb`, 90 × 60 mm). Every connector is reachable from outside with the case closed. Source: `controller-case.sc` (SolidCraft). Exports: `export/case-base.stl`, `export/case-lid.stl`, `export/controller-case.step`.

Outside: 101.2 × 65 × 22.1 mm (121.2 mm across the mounting ears).

## How it goes together
```
 lid   ┌──────────── LED / button holes ─────────────┐
       │  boss + M3 heat-set insert (×4)             │
 ══════╪═══════ PCB top = split plane (z 8.6) ═══════╪══  connector windows straddle the split:
 base  │  standoff 5 mm (clears THT legs)            │  half a notch in the base, half in the lid
       └── countersunk M3 × 14 from below (×4) ──────┘
```
The four screws go up through the base standoffs and the PCB's corner holes into inserts in the lid bosses. They clamp the board and close the case in one step.

## Connector windows (model X = KiCad x − 100, model Y = 160 − KiCad y)
| Wall | Ref | What | Window (w × z-range rel. to PCB top) |
|---|---|---|---|
| North | J4 J5 J6 | temp intake / exhaust / ambient (JST-XH 3P) | 11 × −2.5…+7 |
| North | J7, J11 | reed, console sense (JST-XH 2P) | 8.5 × −2.5…+7 |
| South | J1 | 12 V barrel | 11 × −1…+11.5 |
| South | J2 J3 | fan 1 / fan 2 (KK-254 4P right-angle) | 11.5 × −3…+6.5 |
| South | J9 J10 | LED compartment / backlight (JST-VH 2P) | 9.5 × −2…+11 |
| West | J8 | USB-C | 12 × −3…+5.5 |

East end: the ESP32-C3-WROOM-02 antenna overhangs the PCB edge by 6.2 mm, and the case leaves room for it. The case is plastic, so keep metal away from that end when mounting it.

**The window X positions are plain numbers copied from the PCB placement. If a connector moves in KiCad, update its `Win_*` feature.** Window heights for VH and USB-C come from datasheets (KiCad has no 3D model for them): check against real parts before the final print.

## Lid holes
- Ø2.2 over D3 (status) and D4 (power).
- Ø2.5 over SW1 BOOT and SW2 RESET: press with a paperclip (the actuator is ~10 mm below the lid surface).

## Mounting
2 ears with 2 × Ø4.5 holes each, for 4 mm wood screws into a Bestå side panel or floor. Mount it the right way up so the LED holes face you.

## Hardware
4× M3 × 14 countersunk, 4× M3 heat-set inserts (Ø4 × ~5.7).

## Printing
PETG/ASA. Base: floor down. Lid: top face down. No supports: the windows are notches open at the split line.

## Parameters
`bx`, `by` (PCB), `ant` (antenna overhang), `clr` 0.5, `wall` 2, `floor_t` 2, `so_h` 5, `pcb_t` 1.6, `lid_h` 11.5 (clears the 10.7 mm barrel jack), `top_t` 2, `r_out` 3, `h_in` 4 (PCB hole inset), `post_d` 7, `insert_d` 4, `insert_depth` 7, `ear_l` 10, `ear_w` 36, `ear_hole` 4.5.
