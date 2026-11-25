# Fan cassette (flange + scoop)

One design, printed 2× per fan, used in every bay. Source: `fan-cassette.sc` (SolidCraft 0.44, fully parametric).
Exports in `export/`: `cassette-flange.stl`, `cassette-scoop.stl`, `cassette-probe-clip.stl`, `fan-cassette.step` (all bodies, installed position).

```
          countersunk M3 (from inside the bay)
   ┌──┬───────── guard grille ─────────┬──┐  ← Flange plate (on top of floor panel)
   │  │╔════════ 120×15 fan ═════════╗│  │
 ══╪══╡║                             ║╞══╪══  floor panel, square cutout `cut`
   │  │╚═════════════════════════════╝│  │    sleeve lines the cut edge
   │  ║ standoff (heat-set insert)    ║  │
 ┌─┴──╨─//////// louvres ////////─────╨──┴─┐ ← Scoop (under the panel)
```

The panel is clamped between the flange and the scoop by 4× M3 screws through the fan's own corner holes, so it never relies on screws biting into a (possibly honeycomb) core.

## Intake vs exhaust
Same scoop, rotated 180° about Z. The louvres lean `louvre_angle` from vertical; air leaving downward is thrown toward the low side of the slats (−X in the model). Orient each scoop so intake and exhaust jets point **away from each other** (and away from the baffle strip).

## Parameters (mm)
| Name | Default | Meaning |
|---|---|---|
| `T` | **18 ⚠ unmeasured** | floor panel thickness. Must be ≥ `fan_t` + ~1 |
| `cut` | **126 ⚠** | actual square cutout size: measure after cutting and set |
| `cut_r` | 3 | sleeve outer corner radius = router bit radius (jigsaw: square up the corners or file to ≥3) |
| `fit` | 0.25 | sleeve-to-cutout gap per side |
| `fan`, `fan_t`, `hole_sp` | 120, 15, 105 | Noctua NF-A12x15 |
| `clr` | 0.5 | fan-to-pocket clearance per side |
| `flange_w`, `flange_t` | 12, 3 | flange overlap past the cut, plate thickness |
| `corner_r` | 6 | outer corner radius (flange and scoop) |
| `open_d` | 116 | airflow opening (flange and scoop) |
| `bar_w`, `bar_pitch` | 2, 14 | top guard bars |
| `scoop_t` | 8 | scoop depth below the panel (keep < baffle height) |
| `louvre_angle` | 35° | slat lean from vertical (≤45° prints support-free) |
| `slat_t`, `slat_pitch`, `n_slats` | 1.6, 10, 11 | slats |
| `fan_gap` | 0.3 | standoff top sits this far below the fan, so the screws clamp the **panel** and the fan's rubber pads take up the slack |
| `boss_d`, `insert_d`, `insert_depth` | 8, 4.0, 8 | standoffs / M3 heat-set insert pocket (check your insert's spec) |

Derived: sleeve wall = (`cut` − 2·`fit` − `fan` − 2·`clr`)/2 → 2.25 mm at defaults. Standoff height = `T − fan_t − fan_gap`.

## Probe clip (temperature sensor)
A separate part that holds a waterproof DS18B20 probe (Ø6 × ~50 mm, stainless) over the fan's highest-velocity ring, about 44 mm off centre. It sits on top of the guard and is clamped under the +X/+Y fan screw, so it needs no extra holes. The probe tip points toward the centre and the cable runs off toward the PCB inside the bay.
- On the intake cassette it reads intake air; on the exhaust cassette it reads the bay air being exhausted. It's the same part either way.
- Two snap rings (`probe_d` + 0.3 mm bore, 70 % mouth). Print it pad-down, no supports.
- Parameters: `probe_d` 6, `probe_y` 44 (valid ≲ 48), `clip_t` 2.5, `arm_w` 10, `pad_d` 13, `ring_w` 1.6, `clip_len` 6, `clip_x` 10, `clip_sp` 30.
- The screw at the clip corner needs to be 2.5 mm longer (M3 × 28/30).
- The third (ambient) probe doesn't use a cassette: route it out of the unit, e.g. through the door shadow gap.

## Cable channel
`cable_w` (10 mm) wide at +X 35, cut **through** the full sleeve wall so the fan cable rides between the fan and the panel's cut edge (~3 mm deep), and up through the flange. Turn the fan so its cable leaves on that side.

## Hardware per cassette
- 4× M3 × 25 countersunk (ISO 10642); one of them M3 × 28/30 when the probe clip is fitted. Rule: length ≈ `flange_t + T + 4`; the screw tip must stop short of `insert_depth` below the standoff top.
- 4× M3 heat-set inserts (≈ Ø4 × 5.7 mm), set into the scoop standoffs.
- Optional filter: a 120 × 120 mm sheet of filter foam/mesh (≤2.5 mm) with corner notches, lying on the scoop slats under the fan.

## Printing
- PETG/ASA, matte black. Flange: guard face down. Scoop: bottom face down (slats overhang 35°, no supports).
- 4 walls, 20–25 % infill.

## Deviations from the original spec (flagged)
1. **Square cutout, not 125 mm round:** a 120 mm square fan is ~170 mm across the corners and can't recess into a round 125 mm hole.
2. **Filter is drop-in, not slide-in.** To get at it you remove the flange from inside the bay. A true side-slide slot would add ~4 mm of scoop depth. Say if you want it.
3. **Cable exit:** see "Cable channel" above (widened in v2; the 1.5 mm channel in v1 was too tight for a sleeved fan cable).
