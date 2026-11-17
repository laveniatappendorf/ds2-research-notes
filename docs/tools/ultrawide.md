# Ultrawide for PS3 retail DS2 (BLUS41045 v1.00), 2026-10-31

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

The author asked whether an ultrawide mod is possible. It is: the ratio comes from one division in the code that feeds
the render camera, and an RPCS3 boot patch now replaces it (`rpcs3-patches/imported_patch.yml`, "Ultrawide (aspect
ratio)", also copied into RPCS3's `patches/imported_patch.yml`; backup `imported_patch.yml.bak_ultrawide`).
**Status (2026-10-31): WORKING.** Single RPCS3 patch "Ultrawide (aspect ratio)" v2.0 (23 lines; 3D view, HUD, menus,
inventory and text) confirmed in play by the author at 3440 x 1440 with Stretch To Display Area on. History below.

## Where the 16:9 comes from

- There is no 16/9 constant anywhere: not in the EBOOT (float or double, +-1 ulp) and not stored in the render camera.
  RPCS3's patch.yml (16 Oct copy) has no DS2 aspect patch; Demon's Souls has one (aspect floats + separate HUD
  centring patches), which is the model for the notes / settings.
- Render camera `[App+0x878]` (vtable `0x1c36c20`), retail layout (the demo's +0xc further on):
  +0xa0 eye, **+0xb0 vertical FOV** (rad), +0xc0 view 4x4, **+0x110 projection** (a = b / aspect at +0x110,
  b = 1 / tan(fov/2) at +0x124, c 1.00002, w +1, d -0.1), +0x150 view x projection, +0x190 camera world,
  +0x1d0 view x projection again. Rebuilt every frame (a junk write at +0x110 was gone within 50 ms).
- The App update feeds it from the camera manager M = [App+0x10] (the free-camera route, 0x5511xx): picks slot
  [M+0x24], builds a temporary parameter object (ctor `0xc1fd84`, converter **`0xc1fb2c`**), then calls the render
  camera methods `0xcc9070` and `0xcca1e0` with it.
- Camera slot (0x80 bytes at M+0x30+0x80*i) additions to the demo's map: **+0x5c width, +0x60 height (u32, 1280 x 720
  in every slot)**. Slot 2 (follow camera) is rewritten every frame; slots 0, 1, 3-5 keep pokes.
- Converter `0xc1fb2c`: `lwz r9,0x5c(r4)` (width) / `lwz r10,0x60(r4)` (height) -> stored at obj +0x110 / +0x114,
  converted with `std` / `lfd` / `fcfid` / `frsp`, then **`0xc1fbe8 fdivs f12,f12,f11`** = width / height ->
  `0xc1fbec stfs f12,0x14c(r11)`; FOV / near / far at obj +0x140 / +0x144 / +0x148.

## Correction 2026-10-31: the projection takes its aspect from 0xba1304, not from +0x14c

Patch v1.0 (only the converter's own divide) booted and applied (RPCS3 log "Applied patch", words read back in
memory) but the render ratio stayed 1.7778: obj +0x14c is a stored copy nothing on the projection path reads.
The converter calls **`0xba1304(local, slot)`** at 0xc1fca8, which builds the projection:
- `lfs f3, 0x64(slot)`: **slot +0x64 = explicit aspect override** (0.0 in every slot = use width / height).
- Otherwise `lwz r11,0x5c` / `lwz r8,0x60` -> fcfid -> **`0xba1368 fdivs f3,f13,f0`** = width / height.
- f3 is the divisor of the horizontal scale in both branches of slot +0x70: `0xba14a8 fdivs f8,f9,f3` and
  `0xba17cc fdivs f7,f10,f3`.
Live proof without freezing anything: writing 2.3889 to the ACTIVE slot's +0x64 (slot 2) made the moving render
camera 2.3889 within a frame; the follow camera does not rewrite +0x64 (cleared again afterwards). This is also a
no-reboot way to preview the effect (any CT entry could do it: [[App+0x10]+0x30+0x80*[M+0x24]]+0x64 = aspect).

## Ruled out (pokes with no effect on the render projection)

- The 13 projection presets inside M (each = 4x4 matrix + {vertical FOV, aspect 1.7778, near 0.1, far 5000} at
  +0x40..+0x4c, e.g. M+0x660): their own matrices rebuild from the aspect field, but the render camera does not use
  them. What does use them is still unknown.
- The paired 1280.0 / 720.0 floats in the writable TOC pools (0x1ca2e18/20, 0x1cb094c/54, 0x1cb0dec/f4,
  0x1cb31f8/200, 0x1cb98f8/900) and the {1.7778, 1.0, 0.0001} pair at heap 0x37e79090 / 0x37e790a0 (restored).

## Live proof

`widetest.py` (scratch): copy slot 2 into slot 3, set slot 3 width 1720 (1720 / 720 = 2.3889 = 3440 x 1440), point
M+0x24 at 3. The render projection ratio went 1.7778 -> 2.3889 at the same FOV (44 deg), and the frozen view showed
more of the Skeleton Lords arena at both sides with the same vertical extent (Hor+), nothing missing at the edges.
`ultrawide_preview.png`: top = today (16:9 with black bars), bottom = the 21:9 capture stretched to 3440 x 1440 the way
RPCS3's Stretch To Display Area would show it. The 3D view has the right proportions; the HUD is stretched sideways.

## The patch (v1.1)

Both divides become 1280 / H with H a constant, leaving the slot's real 1280 x 720 for the viewport. One
configurable word, `li r5,H` (0x38a0xxxx), is used at both sites (RPCS3's patch.yml only uses configurable values
with be32 / bef32 / bef64, so both sites share one register to share one setting):

| address | was | now |
|---|---|---|
| 0xba133c | `clrldi r9,r8,0x20` (79090020) | `li r5,H` (configurable) |
| 0xba1348 | `std r9,0x270(r1)` (f9210270) | `std r5,0x270(r1)` (f8a10270) |
| 0xc1fb5c | `mr r11,r3` (7c6b1b78) | `li r5,H` (configurable) |
| 0xc1fb80 | `std r10,0x170(r1)` (f9410170) | `std r5,0x170(r1)` (f8a10170) |
| 0xc1fbec | `stfs f12,0x14c(r11)` (d18b014c) | `stfs f12,0x14c(r25)` (d199014c), r25 = clrldi of the same r3 |

Register checks: in 0xba1304, r9's only use before its reload at 0xba1390 is the redirected store, and r5 is
written (0xba13b4 / 0xba17ac) before any read; in 0xc1fb2c r5 is only set at 0xc1fcc0 for the call, r11 had no
other use; 0xba0e90 sets its own r5. v1.0 used `li r12` at 0xc1fb5c only. H: 536 (3440x1440, 2.388), 540 (2560x1080,
exact), 533 (3840x1600, 2.402), 360 (32:9, exact), 800 (16:10, exact), 720 (no change).
Assumes the slot width is 1280 (true in every slot here); a different width would scale the ratio with it.

## Use

RPCS3: Manage Game Patches -> DS2 -> tick "Ultrawide (aspect ratio)", pick the screen; per-game config GPU ->
**Stretch To Display Area ON** (currently off for BLUS41045); boot the game. Check: `[App+0x878]` +0x124 / +0x110
should read 2.388 in play.

## Open

- Boot test of v1.1 (v1.0 had no effect, see the correction).
- HUD / menus stretch. A centring fix would need the UI's screen transform (candidates: the 1280 x 720 UI canvas
  values; the menu system is the .fe / flo one). Lock-on marker and enemy HP bars: unchecked.
- Pre-rendered videos will stretch. Cutscenes in engine should follow the same converter (unchecked).
- The 1.7778 in M's presets: unknown user (culling, LOD or targeting?); no visible edge pop-in in the test shot.

## HUD and menus (2026-10-31, patch v1.2, not yet boot-tested)

With v1.1 + Stretch To Display Area the 3D view is right but the whole 2D layer is stretched sideways.
- **Canvas** = 1280.0 / 720.0 in the TOC pool 0x1cb8938 (-0x7fe4 = 0x1cb0954 width, -0x7fec = 0x1cb094c height;
  same pool: 1.0 -0x7ff4, 0.5 -0x7fbc, 2.0 -0x7fb4, -2.0 -0x7fac). Readers (pool load lwz r30,-0x6730(r2)):
  0xbddbc4 (scale pair W/1280, H/720 -> out +0 / +4), 0xbdedd0 (**2D projection builder**), 0xbe07b4, 0xbe15f4
  (canvas rect -> screen pixel ints with anchor flags, uniform s), 0xbe1c14. All but the builder get the screen rect
  through vtable +0x138 of [ctx+0xc] (`0xb2a7e0` = vtable 0x1c31fe0 +0x138 reads [this+0x214]+0x980 is the only
  matching method found; its live object was not found).
- Poking the canvas width 1280 -> 1720 shrinks the HUD both ways (uniform s = min(...)), so the canvas is the wrong lever.
- **2D projection objects** (vtables 0x1c38578, 0x1c383a8): 4x4 at +0x30 = [2/1280, -2/720, 0.0001, offsets
  -1.00078 / 0.99861] (e.g. 0x305187c0, 0x37ea2980), +0x78 -> screen rect copy {0,0,1280,720}; rebuilt every frame.
  The device object 0x379806b0 (vtable 0x1c37fc8) holds the screen rect at +0x980 (L,T,R,B ints, depth 0..1 at
  +0x990), also rewritten every frame (pokes don't stick).
- **Builder 0xbdedd0**: rect via `bl 0xb12084` (its ONLY caller; `0xb12084` copies [[r3]+0x980..+0x994]); W = R-L, H = B-T;
  local matrix [0][0] = 2/W, [1][1] = -2/H, [3][0] = (0.5/W)(-2) - 1, [3][1] = 1/H + 1; s = min(W/1280, H/720);
  layer flags byte +0x40: bit 0 -> x scaled by s else by W/1280, bit 4 -> y scaled by s else by H/720; the
  0xbdf46c.. paths add centring / anchoring offsets ((W - 1280 s) / 2 / s etc.) to the second matrix (+0x220 / +0x224).
- **Fix**: make 0xb12084 report the rect bottom as H (536): s = 536/720 = k, so x shrinks by k (undone by RPCS3's
  stretch) and y is -2/536 * k = -2/720 in both y modes; uniform layers keep their shape and sit where their anchor
  flags put them; layers scaled by W/1280 (full-screen fades / backgrounds) keep the full width.
  `0xb12094 lwz r8,0x98c(r9) -> li r5,H` (shared configurable word) and `0xb120a4 stw r8,0xc(r4) -> stw r5,0xc(r4)`.
- Risk to check in game: the other UI helpers (scissor / clip rects via vtable +0x138) still see 1280 x 720, so a
  scrolling menu list could clip in the wrong place. If so, patch 0xb2a7e0 the same way (0xb2a7f4 / 0xb2a804).

## Menus and inventory (2026-10-31, patch v1.3, not yet boot-tested)

v1.2 result (the author): **in-game HUD correct** (round emblem; the anchor flags put the HP bar / items at the left
screen edge and the souls counter at the right edge = a real ultrawide HUD), **menus and inventory still stretched**.
- Layer flag byte (builder 1: [layer+0x40]; builder 2: argument r5): x 0x01 fit+left, 0x02 fit+right, 0x04 fit+centre,
  0x08 unscaled, none = stretch by W/1280; y 0x10 / 0x20 / 0x40 / 0x80 the same for top / bottom / centre / unscaled.
  Fit offsets go into the second matrix (+0x220 / +0x224 in builder 1).
- Builder 1 (0xbded78, OPD 0x1ce01b0 in slot 0x1cb09d0, called indirectly) sends its result straight to the GPU
  (`bl 0xb120bc`, shader constants); it is the HUD path.
- **Builder 2 (0xbe0794)** is a near copy that reads the screen rect through vtable +0x138 (twice: 0xbe0844 for the
  base matrix 2/W, -2/H at +0x270 / +0x284, offsets +0x2a0 / +0x2a4; 0xbe095c for the scale factors) and is the menu
  path. The two persistent 2D projection objects (0x30518790 vtable 0x1c38578, 0x37ea2950 vtable 0x1c383a8) kept
  2/1280 under v1.2.
- v1.3 gives builder 2 the same 1280 x H screen: 0xbe085c `subf r0,r9,r0` -> `li r5,H`, 0xbe0868 `extsw r0,r0` ->
  `extsw r0,r5`; 0xbe0970 `subf r11,r0,r11` -> `li r5,H`, 0xbe097c `extsw r11,r11` -> `extsw r11,r5`.
  The shared vtable getter 0xb2a7e0 stays untouched (other UI helpers keep 1280 x 720, as they did for the HUD).
- If menus still stretch after v1.3: their layers are flagged x-stretch (no 0x01/0x02/0x04); then remap the
  stretch branch to fit+centre (builder 1 0xbdf4c4, builder 2 near 0xbe105c) at the cost of full-screen layers.

## v1.3 result and the menu toggle (2026-10-31)

v1.3 booted and applied (all words read back), but the inventory stayed stretched. Live tally of the 2D draw
constants (transposed 4x4 uploads: x scale m[0], y scale m[5], x / y offsets m[3] / m[7]) with the inventory open:
474 matrices, **all x scale = 2/1280** (none squeezed), y offset +1.002 = 1 + 1/536, so they do go through a patched
builder (the height trick is live) but take the **x-stretch branch**: the menu layers are flagged "fill the width".
The two persistent 2D projection objects (vtables 0x1c38578 / 0x1c383a8) stay 2/1280 and are not the menu path.
- Builder 1 x branches: bit 1 -> s; else 0xbdf4a0: bit 2 -> 0xbdf4d4 (fit right), bit 4 -> 0xbdf550 (fit centre,
  offset (W + s*C) * 0.5 / s into +0x220), bit 8 -> 0xbdefd4 (unscaled), none -> 0xbdf4c4 (x *= W/1280).
- Builder 2 x branches: bit 1 -> 0xbe09e0; else 0xbe105c: bit 2 -> 0xbe118c, bit 4 clear -> 0xbe11f0
  (unscaled / stretch), bit 4 set -> 0xbe1074 (fit centre, re-reads W from the first rect, offset into +0x260).
- Separate patch **"Ultrawide: menus keep their shape"** (off unless ticked): 0xbdf4b4 `bne cr7,0xbdf550` -> `b 0xbdf550`,
  0xbe1070 `beq cr7,0xbe11f0` -> `nop`; flags 8 and none become fit + centre. No effect at 16:9 (s = 1, offset 0).
  Cost: any full-width layer (fades, full-screen backgrounds) is centred at 16:9 as well. Not boot-tested.

## Text (2026-10-31, main patch v1.4 + toggle v1.1, not yet boot-tested)

Toggle v1.0 result (the author): menu boxes keep their shape, but **all text** (counts, descriptions, level, even "L1" /
"Equipment" labels and the souls number) stays at the stretched positions, outside its boxes. Comparing captures:
HUD text ("Lightning Spear") never moved or narrowed under v1.2 either; it only looked right because it sits near
left-anchored boxes.
- Text renderer **0xbeb6c0**: calls the scale pair 0xbddbc4 (only used on its flag-0x40 branch at 0xbec2f0), the
  layout matrix helper 0xbe1be4(ctx, out, flags = [text+0x1d]) and the rect conversion 0xbe15f4, then the font
  module (0xb608xx..0xb620b4).
- **Font projection 0xb60c40** builds its own pixel ortho from the DEVICE rect (`ld 0x988(r4)` / `ld 0x980`),
  independent of both getters.
- The helpers' screen rect comes through vtable +0x138; its target is still unidentified (0xb2a7e0 has no live
  objects; a getter-shape scan of all vtable +0x138 slots found no live UI candidate), so v1.4 overrides the height
  inside each consumer instead (robust to the getter): 0xbddc14/1c, 0xbe1658/60, 0xbe1900/0c, 0xbe1b14/1c,
  0xbe1c4c/54, 0xbe1e34/44, 0xbe1f2c/3c (rect bottom load -> li r5,H; B - T -> H - T; r5 free: flags already copied
  to r27 / r28), font 0xb60c78/88/94 (li r5,H; extsw r11,r5; std r11 -> r11 = H also reaches 0xb60d60).
- Toggle v1.1 adds the helpers' stretch remaps: 0xbe1db0 `bne` -> `b 0xbe1ee4`, 0xbe17e4 `bne` -> `b 0xbe1a60`.
- To check after boot: text inside its boxes (menus with the toggle, HUD labels at the edges without it), glyphs
  not wide, lock-on marker / enemy HP bars / damage-free world-anchored UI still in the right place (any layer with
  y flag 0x80 "unscaled" would now be mapped through -2/H and land too low), scrolling-list clipping.

## v1.4 result and the unscaled fix (2026-10-31, main v1.5 + toggle v1.2, not yet boot-tested)

v1.4 + toggle v1.1 (the author): text about 90 % right, but some normally hidden text shows: overlapping "NO TEXT"
placeholders under the item name in the equipment window (first letter culled: "O TEXNO TEXNO ..."), and the
collapsed Message tab's dropdown item "Written History" on the menu bar ("Read History" stays hidden).
- No GPU scissor command is built in code (no 0x08C0 header); the font box set by 0xb60a74 (+0x48..+0x54) is only
  stored, not used for glyph culling in the font module.
- Live text records (vtable 0x1c34808: +0x10 node, +0x1d flags, +0x20 font, +0x2c UTF-16 string, +0x38 size,
  +0x44 colour): 203 in the menu; flag bytes 0x44 x41, 0x24 x14, 0x21 x7, **0x00 x115** (all tab / dropdown labels,
  shown and hidden alike, e.g. "Player Status", "Written History", "Read History", "Message"), **0x88 x26** (empty
  strings, x and y unscaled). Hidden-vs-shown is not decided by the flags; the hiding mechanism (node / mask) is
  not identified.
- Bug in v1.4: **y unscaled (0x80)** layers / text are raw 720-pixel units; with the screen reported as 1280 x H
  they were mapped through -2/H (1.34x too low and tall). Masks or placeholders of that kind are the likely source
  of the stray text. Main v1.5 sends y-unscaled to y-stretch (x H/720 = identity in device pixels) in all four
  flag routines: 0xbdf48c (bne), 0xbe11dc / 0xbe1df4 / 0xbe1868 (blt on the flag sign bit) -> nop.
- Toggle v1.1 also centred **x-unscaled (0x08)** items, which are device-pixel (screen-space) things such as
  markers; toggle v1.2 remaps only x-stretch: builder 1 0xbdf4c4 -> b 0xbdf550, builder 2 0xbe11fc -> b 0xbe1074,
  layout 0xbe1dc0 -> b 0xbe1ee4, rect conversion 0xbe17f4 -> b 0xbe1a60 (originals restored at 0xbdf4b4,
  0xbe1070, 0xbe1db0, 0xbe17e4).

## v1.5 result (2026-10-31)

v1.5 + toggle v1.2 booted and applied; the stray text ("NO TEXT", collapsed "Written History") is **still there**,
so the y-unscaled masks were not the cause. In-game HUD fine (left items, centre "Poisoned!" and poison meter); with
the toggle on, the stretch-flagged souls box sits inside the 16:9 area, and its number was missing in one capture.
- Menu text "node" objects (vtable 0x1c34550, pointed to by text record +0x10) are scene-tree links: +0x08 id,
  +0x10 / +0x20 / +0xa0 / +0x110 / +0x120 sibling / owner pointers, **+0x90 parent**: "Written History" and
  "Read History" share parent 0x31acacd0 (the Message dropdown), "Player Status" and "Message" share 0x31aca5f0.
  The hiding is therefore done at the dropdown (parent) level; mechanism still unknown.
- Next: one boot with the toggle off tells whether the toggle's centring (dropdown vs its hider out of step) or the
  main patch's text changes reveal the hidden items.

## Toggle-off test and v1.6 (2026-10-31, not yet boot-tested)

With the menu toggle OFF (main v1.5 only) the stray text is still there, so it comes from the main patch's text
changes. Also missing since v1.4: the HUD **souls number** and the **quick-item name / count** (bottom of the
screen, about row 650 of 720; "11 Lightning Spear" at about row 410 is fine).
- `0xb60c40` is not just the font projection: a general 2D pixel setup called from 0xb2c5c4, 0xb2cc48, 0xb4750c,
  0xb47958 (vector-built output). Its only uses of the height are the matrix build and a store at +0x2a4. The font
  draw (0xb608dc) goes through methods of the 2D projection object class (vtable 0x1c38578: 0xd4faf8, 0xd50240,
  0xd4f3b0). Live under v1.5: object A kept sx 2/1280, sy -2/720, but its y offset moved 0.99861 -> 0.94397
  (about 20 px), so patching 0xb60c40 never gave text a 536-row projection.
- Text renderer special branch (0xbec348..0xbec3ac): text box = (0, 0, trunc(sx * 1280), trunc(sy * 720)) from the
  scale pair (pool 0x1cb8d4c: -0x7f60 = 1280.0, -0x7f58 = 720.0, -0x7fb8 = 0.5) -> with the scale pair patched it
  became 1280 x 536, the likely cut-off for the bottom text.
- v1.6 reverts the scale pair (0xbddc14 / 0xbddc1c) and 0xb60c40 (0xb60c78 / 88 / 94); keeps the layout / rect
  conversion height overrides and the v1.5 y-unscaled remap.

## v1.6 result and v1.7 (2026-10-31, not yet boot-tested)

v1.6 (the author): stray text ("NO TEXT", "Written History") **gone**, souls number and quick-item text **back**, but all
text sits **too high by the factor 536/720** ("11 Lightning Spear" row ~452 -> ~335; souls number at 0.74 of its
box's height).
- So: the text helpers lay text out in the squeezed 1280 x H space (y x s), the patched 0xb60c40 had been
  scaling text y back to 720 rows, and the text clip boxes (rect conversion output, plus the special-branch
  full-screen box) are applied in **720-row device pixels**. v1.4 / v1.5 = text y fixed but clip boxes squeezed ->
  hidden text revealed and the bottom cut off; v1.6 = both squeezed -> consistent clipping, text too high.
- v1.7 keeps text in device pixels: the helpers still get height H for their scale s (x fit / anchors like the
  boxes) but skip their vertical part entirely: layout matrix 0xbe1d58 `rlwinm` -> `b 0xbe1d70` (return, y scale 1),
  rect conversion 0xbe1748 `rlwinm` -> `b 0xbe17a8` (return, T / B untouched). At 16:9 the y part is identity anyway.
  Dropped as unreachable: the helpers' y-anchor height overrides (0xbe1900/0c, 0xbe1b14/1c, 0xbe1e34/44,
  0xbe1f2c/3c) and their v1.5 y-unscaled nops (0xbe1df4, 0xbe1868). Kept: builder y-unscaled nops (0xbdf48c,
  0xbe11dc), since the 2D builders do use the 1280 x H projection. Main patch = 19 lines.

## Working: v1.7 + menu toggle v1.2, merged as v2.0 (2026-10-31)

The author, after v1.7 with the menu toggle back on: "Everything seems to work" (3D Hor+, HUD at the screen edges,
menus / inventory centred with text in place, no stray text, numbers present). Still to watch in play: lock-on
marker / enemy HP bars, fades and loading screens (may stop at the 16:9 edges), cutscene / video stretching,
other aspect presets (only 21:9 3440 x 1440 tested).
- Merged at the author's request into one entry, **"Ultrawide (aspect ratio)" v2.0**: the four stretch -> fit + centre
  lines (0xbdf4c4, 0xbe11fc, 0xbe1dc0, 0xbe17f4) moved in, the separate "Ultrawide: menus keep their shape" entry
  removed (its leftover "Enabled" line in RPCS3's config/patch_config.yml is ignored).
- Final line list: 3D aspect 0xba133c/48, 0xc1fb5c/80/ec; HUD builder rect 0xb12094/a4; menu builder heights
  0xbe085c/68, 0xbe0970/7c; text helper scale 0xbe1658/60, 0xbe1c4c/54; text skips y 0xbe1d58, 0xbe1748; builder
  y-unscaled -> stretch 0xbdf48c, 0xbe11dc; x stretch -> fit + centre 0xbdf4c4, 0xbe11fc, 0xbe1dc0, 0xbe17f4.
