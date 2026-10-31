# Reading the plan

Pick the branch `intake` names. Coordinates go into `plan.json` (`references/plan-schema.md`); `compile` converts and checks them.

## A. Raster image (real-estate plan, scan, photo of a drawing, PDF page rendered with `node "$HF" pdf render <sheet.pdf> --out <page.png>`)

1. `node "$HF" image prepare <image> --out house-forge-output/<name>/source/prep` → look at `grid.png` and the
   `tile_*.png` zooms. Labels are SOURCE pixel coordinates (x right, y down). On a small image (intake warns below 1200 px; walls
   are then 3–5 px thick) read wall faces and openings from close-ups: add `--zoom-box x0,y0,x1,y1` (repeatable, 4×, 10 px grid)
   for every area you need.
2. Write down, for every room: name (as printed), kind, the rectangle (or polygon) of its **inner wall faces** in pixels —
   the boundary between wall and floor, not the centre of a wall line; fractional pixels (e.g. 111.5) are fine and matter on
   small images, here and in `--origin` — and its printed area if any. For every door: the two rooms (or `exterior`), the pixel in the middle of the opening, the swing arc —
   which room it opens into and on which side the hinge is (the arc's centre). Windows: the room, a pixel on the wall in the
   middle of the window, its width. Fixtures (bathtub, WC, washbasin, kitchen run, bed, wardrobe, stairs).
3. Scale — at least two independent estimates:
   ```bash
   C=house-forge-output/<name>/source/calibration.json
   node "$HF" image calibrate --file $C points --a X,Y --b X,Y --length <metres>     # dimension line / scale bar
   node "$HF" image calibrate --file $C area --rect X0,Y0,X1,Y1 --m2 <printed area>  # a rectangular room
   node "$HF" image calibrate --file $C total --rects "X0,Y0,X1,Y1;…" --m2 <sum>     # several rooms
   node "$HF" image calibrate --file $C symbol --kind door --a X,Y --b X,Y           # weak prior: door,
   #                                                    entrance-door, bathtub, bed, wc (references/symbol-legend.md)
   node "$HF" image calibrate --file $C summary --origin <pixel of the outer bottom-left corner> [--use 0,2]
   ```
   `summary` must say OK. If it says DISAGREE: re-check the picked pixels, drop a wrong estimate with `--use` (indexes it
   lists); if still not OK, ask the user for ONE real dimension (a room width, the total area).
   - **Dimensioned plan** (numbers along the walls, in cm on most European plans): the overall chains are the best evidence —
     measure the longest ones with `points` (both directions). Printed room sizes (e.g. 458 × 341) are inner faces: use them
     for the outlines, not only the pixels, and after the build check every room against them (the plan preview shows sizes on
     the 1 m grid). Printed areas are often 2–5 % below length × width (plaster, rounding); prefer chains over areas for the
     scale.
   - **No dimension and no area printed**: only symbol priors are left. Use at least three symbols of two kinds (interior
     doors, bed length, bathtub, WC) — `summary` then says PROBABLE. Set `defaults.confidence` to `"probable"`, say in plan.json `source`
     that the scale rests on symbols, and ask the user for one real dimension when you hand over.
4. Several images (one per level): prepare and calibrate each one (its own `calibration.json`), pick the same building corner as
   `--origin` on every image, and put each `summary` block into that level's `calibration` (references/plan-schema.md).
5. Write `plan.json` with `"units": "px"` and the `calibration` block from `summary`. Sizes (door / window widths, sills,
   heights, furniture dims) are always metres. The compiler converts, snaps to 5 mm and regularises walls.

## B. Vector PDF sheet (architect's drawing)

```bash
node "$HF" pdf text <sheet.pdf> --out house-forge-output/<name>/source/pdf --find pitch,ridge   # words, printed areas, scale, roof facts
node "$HF" pdf svg  <sheet.pdf> --out house-forge-output/<name>/source/pdf     # sheet_grid.png, paths in pt
node "$HF" pdf arcs  <sheet.pdf> --out house-forge-output/<name>/source/pdf --scale <N>   # hinges, leaves
```
Scale: `pt per metre = 72 / 0.0254 / N` (1:N from `pdf text`); confirm it with one printed dimension. Read room outlines from
`sheet_grid.png` (labels in pt), convert to metres yourself or write a `"units": "px"` plan whose calibration uses
`pxPerM = pt per metre` and the origin in pt (the PDF y axis points down like an image). Door hinges come from `arcs.json`.

## C. CAD (DWG / DXF)

```bash
node "$HF" cad dwg2dxf <file.dwg> -o house-forge-output/<name>/source/cad          # DWG only (ODA File Converter)
node "$HF" cad inspect <file.dxf> --out house-forge-output/<name>/source/cad    # layers.json, layer_*.png, texts.json
```
Look at the layer images; propose which layers are walls / partitions / doors / windows / room texts (see
`references/cad-layer-heuristics.md`) and confirm with the user. Several plans on one sheet → pick a `--box` around one level.
```bash
node "$HF" cad walls <file.dxf> --layers "<walls>,<partitions>" --unit <cm|mm|m> --text-layers "<texts>" \
    --box x0,y0,x1,y1 --out house-forge-output/<name>/source/cad/<level>
```
Use `rooms.png` / `walls.json` candidates (metres, origin at the bottom-left of the walls) as room rectangles; map texts inside
them to names and areas; doors and windows from the plan images or the door / window layers. Write `"units": "m"`.

## D. No drawing

Interview: rooms with sizes, which walls they share, where the entrance, windows and bathroom are. Write `"units": "m"`, set
`defaults.confidence` to `"unknown"`, and say clearly in plan.json `source` that the geometry is approximate.
