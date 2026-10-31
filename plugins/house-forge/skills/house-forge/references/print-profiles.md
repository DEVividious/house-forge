# Print profiles and settings

A profile is the smallest features a printer makes reliably, in mm on the print. Before the model is built, walls, slabs,
piers and openings smaller than these at the chosen scale are thickened or widened (each change is on the card and in
`print-report.json › adjustments`); after it, every part is checked against them (thin spots, overhangs, bridges).
`--nozzle 0.4` (default), `0.6` or `resin` picks the profile; `--min-wall <mm>` overrides its smallest wall.

| | `--nozzle 0.4` | `--nozzle 0.6` | `--nozzle resin` |
|---|---|---|---|
| Printer | FDM, 0.4 mm nozzle, 0.2 mm layers | FDM, 0.6 mm nozzle, 0.3 mm layers | resin (MSLA), 0.05 mm layers |
| Smallest wall | 1.2 mm | 1.8 mm | 0.6 mm |
| Slab, lintel, roof layer | 1 mm | 1.2 mm | 0.6 mm |
| Pier, post, chimney | 2 mm | 3 mm | 1 mm |
| Smallest opening | 1.5 mm | 2 mm | 0.8 mm |
| Steepest overhang without supports | 45° from the vertical | 45° from the vertical | 30° from the vertical |
| Longest bridge without supports | 15 mm | 15 mm | 5 mm |
| Lowest step (lower: a ramp) | 0.5 mm | 0.6 mm | 0.2 mm |
| Clearance (reported, not used) | 0.25 mm | 0.3 mm | 0.15 mm |

## What the model is

- One part per storey (its floor slab and its walls up to the slab above) and a removable roof. The last storey is cut where
  its ceiling or the bottom of the eaves is lowest; the roof part lies on the bed on a flat face (upside down when that needs
  fewer supports). No connectors, bases or labels: the parts stack on their flat faces.
- Windows and doors are openings; no glass, leaves or furniture. Stairs print as solid steps or a ramp.
- `--eaves trim` cuts the overhangs of the roof at the outer faces of the walls (a compact model, fewer supports).

## Printing (FDM, PLA)

| Setting | Value | Why |
|---|---|---|
| Layer height | 0.2 mm (0.12 mm for 1:150 and smaller) | the profile's slab of 1.0 mm is five layers |
| Walls / perimeters | 3 (1.2 mm with a 0.4 mm nozzle) | the thinnest walls of the model are the profile's wall: solid, no infill |
| Infill | 15 % grid or gyroid | slabs and roof blocks are large solids |
| Top / bottom layers | 5 / 4 | flat roofs and floors close cleanly |
| Supports | only where the card lists them; tree supports, touching the build plate | lintels of wide openings and ceilings of rooms under the roof hang over |
| Brim | 3–5 mm for parts taller than about 60 mm | tall thin walls lift at the corners |
| Material | PLA (any colour; white or grey shows the shape best) | |

Print each part on its own, as it lies in its STL. After printing, the parts stack: storey on storey, the roof on top.
