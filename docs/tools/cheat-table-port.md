# Porting the Network Test cheat table (DS2NT.CT) to retail v1.00: what carries over

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Static only, 2026-10-17 (no retail save or boot yet). Method: `tools\xmatch.py` (this folder) finds Network Test code in
the retail EBOOT by masked instruction windows (immediates, displacements and branch offsets masked; `--loose` also
masks registers; `--multi` votes over windows through a whole function), then the matched pairs were disassembled or
decompiled (`DefineAndDecomp.java` on DS2NT_mac and DS2V) to read which offsets moved. MSB checks with the shared
`gamedata.py` (`DS2_GAME=vanilla`).

**Corrected the same day (section 4):** the App global is **0x1d98310** (0x1edfbe0 is a vector block the collision-part
constructor copies), the player character is **[App+0x64]** in retail (the demo's App+0x60), and several demo "offsets"
(chr+0x588, chr+0x5f24, player data +0x174 / +0xb1c / +0xd7c, the inventory's +0x74 / +0xc104) only worked because the
demo's heap places the next objects at fixed distances: retail needs the real pointer chains. Sections 2-3 are the
first pass, kept as written.

## 1. Short answer

The table's **engine-level tricks port**: world speed, free camera, draw the whole map, out-of-bounds kill off, the
i-frame hold, and (with re-found offsets) the Havok-based movement entries and the AI freeze. **Everything keyed to the
demo's data or heap layout has to be rebuilt**: every fixed heap address, the character / player / equipment /
inventory offsets, the attribute block (retail has Adaptability), and the item dropdowns (spells are all renumbered,
rings partly cut, retail has four ring slots). Nothing can be ticked on retail until stage 2 (a retail character and
save, the App object and chr offsets confirmed live).

## 2. Per entry

| # | Entry | Mechanism in retail | What changes |
|---|---|---|---|
| 1 | Find base + resolve objects | yes | RPCS3 guest base is the emulator's, not the game's. New anchor string + TOC check (retail TOC 0x1d1f068). Better root than the demo's fixed heap address: retail keeps the App pointer in a static global, **probably 0x1edfbe0** (two code sites agree; the demo's equivalent is 0x1b09e50). The "Checks" vtables need retail values. |
| 2 | Enemy AI off | yes (AI update matched: NT 0x74a8c0 -> retail 0x95cc20, same frame-skip test) | frame-skip byte **AI+0x2a8 -> AI+0x19c**; the output block (NT AI+0x2b0..+0x32f), chr -> AI chain and vtables still to read. |
| 3 | Ghost (no-clip) | likely (Havok library code identical, see 4) | chr -> Havok entity (NT chr+0x5f24), pad buffer (NT 0x100aae5c), OOB-check internals used by the landing ([chr+0x588], skip counter [chr+0x84]+0x3b4, on-ground byte) all to re-find. |
| 4 | No gravity | likely | Havok code matches byte for byte apart from TOC offsets (e.g. NT 0x111f544 -> 0x13368d4, 0x1156d84 -> 0x136e05c), so world gravity (+0x14) and the body transform should be the same; only the chain from the chr differs. |
| 5 | Out-of-bounds kill off | yes | ChrMoveParam row 100 has the **same 772-byte layout; kill time +0x150 = 7.0 s in retail** (demo 60.0; [the JP test's 01.02 already had 7 s](../network-test/jp-network-test-vs-us.md)). More important on retail: Ghost needs it within 7 s. |
| 6-7 | Save position / Teleport | likely | same dependencies as Ghost. |
| 8 | Player cannot die | likely | HP is at chr+0xFC on PC vanilla (demo +0xF0), so min HP is probably +0x100; confirm on PS3. |
| 9 | Infinite stamina | likely | PC vanilla stamina near chr+0x140 / +0x14C (demo +0x17C); confirm. |
| 10-11 | Apply / make permanent (weapons, armour, spells, rings, ammo, items) | probably, rebuilt | equipment block differs (retail **four ring slots**, so later slots shift), inventory layout and saved index table to re-find. Dropdowns rebuilt from retail params: [weapons 300/306](../network-test/weapons-nt-vs-retail.md) and [armour 408/412 keep their ids, rings 43/51](../network-test/armour-rings-spells-nt-vs-retail.md), **spells all renumbered**; the demo-only cut items have no retail rows. The grey placeholder weapons are still in retail (whip 6470000 untested there). |
| 12 | Player i-frames always on | yes, already decoded | **S = [chr+0x94], counter S+0x5a0** (demo [chr+0x84]+0x3dc), second counter S+0x5b4, ratio S+0xa4 ([roll i-frames and Agility](../mechanics/agility-iframes.md)). Super / ultra armour counters not mapped yet. |
| 13 | Enemy swap | probably | App+0x20 member keeps its offset (state-28 handler); generator chain, object vtable / +0x40, and the load-hang release ([[App+0x0c]+0x3b0]+8) unverified. Retail has 24 maps, so the picks become per map. |
| 14 | Free camera | **yes, same design** | retail 0x551150 is the demo's 0x4071b0: camera manager **[App+0x10], index +0x24, six 0x80-byte slots from +0x30** (unchanged). [Render camera](ultrawide.md#where-the-169-comes-from) **App+0x1d28 -> App+0x878**, feed switch byte **App+0x1d2c -> App+0x87c**. FOV = slot +0x50 (slot copied whole). Body pinning needs the chr offsets. |
| 15 | World speed | **yes, identical** | frame function NT 0x927010 = retail **0xb26d38** (byte-identical apart from TOC offsets): switch byte +0xe4, mode +0xe0, mode-2 constant **0x1cac3b0** (1/60 at 0x1cac3a8, 0.05 at 0x1cac3a0). main() (NT 0x8f95d8 = retail 0xaeeb48) is identical, so the game object is again main's stack frame (demo 0xd0040aa0 via pointer 0x01b0b1d8); the retail pointer is not located yet ([read it live](port-live-check.md)). |
| 16 | Draw the whole map | **yes**, section 3 | |

Value groups: "Player", "Attunement", "Spell school", "Inventory" and "Checks" are fixed demo heap addresses: rebuild.
"Player vitals" shifted (HP +0xFC on PC vanilla). "Attributes and souls": pparam = **[chr+0x378]** in retail (demo
+0x368), retail order has Adaptability and no Resistance / Agility. "Movement values": as entries 3-5.

The App object: its low members keep their offsets (state-28 handler NT 0x3fe2e0 = retail 0x54b1a0 uses +0x0c, +0x10,
+0x1c, +0x20, +0x24, +0x38, +0x3c, +0x40, +0x27c..+0x288, +0x46c, +0x60 in both), but the tail is 0x14b0-0x1590 bytes
shorter: App state +0x23ec -> **+0xe5c** (state machine NT 0x3fdf38 = retail 0x545e48), feature mask +0x23d0 -> +0xe40
(App init NT 0x4059f0 = retail 0x54d7c0), +0x21d0 -> +0xd20.

PPU patches (not the table): "Disarmable Trapped Chests" and "Force Enable ThreadDebugger" patch demo code by address
and hash; neither applies. [Retail dropped the fuse rows from EventCommonParamFloat](../mechanics/agility-timeline.md#4-the-change-came-in-steps-and-the-trap-system-kept-growing-first); whether the disarm roll survives is
the open stage-1 trap topic.

## 3. Draw the whole map on retail

**Data side: identical.** Retail MSB parts have the same header (draw / display / backread masks at entry +0x30 /
+0x40 / +0x50). Retail has the demo's map, `m10_23_00_00.msb` (Huntsman's Copse; same id): 75 collisions against the
demo's 64 (62 shared; demo-only h04_1700_0001, h04_1900_0001; retail-only h04_2201, h07_0501, h07_7400 x2, nine
h99_3xxx). Collision object +0x60..+0x6c is still the runtime copy of the mask (retail code 0x599708 / 0x599e60 reads
it as the demo does), and App+0x1c is still the map member.

**Not confirmed yet** (needs a matched function or a live read): W = [[App+0x1c]+4] with current / applied collision at
+0x10 / +0x14, the collision list at map object +0xc88 / count +0xc8c (the +0xc88 hits found in both EBOOTs are Havok
code, not the map), and the chr floor-collision field (demo chr+0x35c; the demo's refresher 0x5a58fc has no direct
match, its two callees do: 0x3c1828 -> 0x4da004, 0x3c144c -> 0x4d9c08).

**Changes for a port:**
1. Retail chains (root, W, map object, list offset).
2. Do not embed the masks: the demo entry carries m10_23's 60 file masks. Retail has 24 maps, so save each loaded
   collision's +0x40 when ticking (or read the current map's MSB) and restore that; re-apply after loads per map.
3. Frame rate: 60 FPS was measured on the demo's one map; retail maps are bigger, so test on a large one.

**What it would show in m10_23.** The demo's [three hidden arenas](../unused/hidden-map-parts.md) are still there in retail, unchanged: collisions
h06_7000 / 7100 / 7200 (display groups 70 / 71 / 72, same masks) and the same map pieces (6 / 161 / 161), with more
objects. Retail adds three regions that name them: 決闘_死合_吊り橋 / 立体部屋 / 中央足場 _BGM再生範囲 ("duel / death
match: suspension bridge / multi-level room / central platform, BGM range"). With retail's map offset (+336, -11, -363
from the demo's coordinates, from the Chariot, Skeleton Nobles and arena-light regions) they sit on the arenas:
**group 70 = suspension bridge** (x -406; the demo's "round floor crossed by a long bar"), **71 = multi-level room** (x
-486; the square floor with four pits), **72 = central platform** (x -566; the raised centre with a wooden scaffold).
So in retail these are most likely the PvP arena stages (Brotherhood of Blood arena matches; that covenant sits in
Undead Purgatory, part of this map), not cut content. On retail, draw-all plus the free camera would show all three at
once.

The author (2026-10-17) went and looked: groups 70-72 are the retail PvP arenas.

## 4. Rebuild started (2026-10-17, same day)

Three static investigations ([port: draw the whole map](port-map-draw.md), [port: player character](port-character.md), [port: equipment, AI freeze and enemy swap](port-equip-ai-spawn.md)) gave the retail values;
each value has a twin code site in the demo unless marked probable there. Highlights:

- **Root:** App global 0x1d98310 (`new App` of 0xf10 bytes at 0x878a4c, the only store), App vtable 0x1c1f7b8, state
  +0xe5c, player character **[App+0x64]** (MM::Update 0x8af1a4; 225 retail reads of +0x64 against 4 of +0x60).
- **Character:** vtable 0x1c2aa58 (enemies' base 0x1c28ea8), HP +0xfc (min / max / cap +0x100 / +0x104 / +0x108,
  probable), stamina +0x140 / max +0x148 (probable), S = [chr+0x94] (i-frames +0x5a0, super / ultra armour +0x598 /
  +0x59c, OOB skip counter +0x56c, sink flag u16 +0x478 bit 0x1000), pparam [chr+0x378] (VGR END VIT ATN STR DEX INT
  FTH ADP from +0x04, souls +0xe8, SL +0xcc probable), player data [chr+0x2d4] (vtable 0x1c29a70).
- **Real chains** (the demo's large chr offsets were heap neighbours): Havok wrapper [[[[chr+0xb8]+8]+8]+0x1a4]
  (entity [W+0x10] vt 0x1c57ce8, world vt 0x1c581f0, Havok internals unchanged), OOB check [[chr+0xa8]+0x8c] (internals
  unchanged, plus a reference copy at +0x30; the kill moved to FUN_008357c8 and still reads ChrMoveParam +0x150),
  on-ground = byte [[chr+0xb8]+8]+0xb4 (the game's filtered test).
- **Map draw:** MM [App+0x1c], W [MM+4] (+0x10 current / +0x14 applied, as in the demo), but the collisions come from
  every loaded map instance: [W+0x1c+4i] for i < s16 [W+0xc8], loader [inst+0xfc], parts [L+8] / count [L+0xc]; part
  vtable 0x1c2b770, +0x18 -> MSB entry; the map id moved to MM+0xb4.
- **Camera / speed:** camera manager vtable 0x1c2f860 (ctor 0xa12248, probable); game-object pointer 0x1d99650.
- **AI:** CharacterManager vt 0x1c29f08, AI = [[[chr+0xac]+0xc]+0x10] (vt 0x1c2dc60), frame-skip +0x19c, output block
  +0x1d0 (the demo's +0x2b0 layout, but one zero word at +0x30 instead of two); all-AI-off counter AiManager+0x0c.
- **Not ported yet:** pad buffer, render camera class, floor-loader state byte. (Equipment, inventory and the enemy
  swap followed the same day: last paragraph.)

**Tooling** (shared, in `..\..\DS2 Network Test\tools`): `ctprofile.py` holds every game-specific value per build and
renders it as the Lua table K; `ct_move.lua` reads all of them from K (retail-only branches: pointer-hop chains, the
instance walk and live-mask saving in draw-all, the filtered on-ground byte, floor from W+0x10, a fixed 2.5 s hold
while the floor-loader state is unknown, keyboard steering while the pad buffer is unknown); `ctlib.py` writes the XML.
`make_ct.py` regenerates **DS2NT.CT** through them: proven equivalent to the previous table (same 307 entries, ids,
addresses and types; the Lua differs only by the K block, comments, and branches the demo's values never reach;
Lua 5.3 compile clean; old table in `Ghidra Project\export\win\port\DS2NT_before_port.CT`).
`..\tools\make_ct_vanilla.py` writes **..\cheat-tables\DS2V.CT** (12 script entries + base finder; README next to it)
and `..\tools\ct_mocktest.py` runs it against a fake memory image built from the retail layout: 36 / 36 checks (base
finder, chains, draw-all over two map instances with a reload, world speed, the body entries, free camera, a full
Teleport and Ghost flight and landing, AI freeze). None of it has touched the retail game.

**Equipment, inventory and enemy swap ported (2026-10-17, later; [port: equipment and inventory](port-equipment-inventory.md)).** The open static
questions closed: inventory nodes are 0x20 bytes (id +0xc, ItemParam word 0 +0x10, index +0x14, flags +0x16, the
demo's tail at +0x18), mirror entries are 16 bytes at X+0x18, the store keeps a free-list head at S+0x1e014 and item
counters, and its list objects sit in seven arrays with the **demo's list codes** (MenuCategoryIconParam). Equipment
records hold the same param ids as the demo's slots, plus a category byte from a slot table the EBOOT builds with its
own slot names (`Spell[0]`..`Spell[13]`: the 14-entry group is the attunement slots; four rings). The generator chain
has no code twin (it was a heap observation), so retail scans for the generator vtable; the load wait is
[[App+0x0c]+0x418]+0xc in the same App state 21. `ct_equip.lua` and `ct_enemy.lua` now read K like `ct_move.lua`.
DS2NT.CT was regenerated and proven equivalent by `..\..\DS2 Network Test\tools\ct_abtest.py`: static, the same 307
entries and table script, the Lua differing only in the three rewritten entries; behaviour, the old and new tables run
apply / make permanent / the refusal / enemy swap on one fake demo image with identical results and memory (11 / 11;
old table in `Ghidra Project\export\win\port\DS2NT_before_equip_port.CT`). DS2V.CT gains entries 14-16 with retail item
lists and one enemy list per map (`..\tools\ct_items_vanilla.py`) and eight inventory rows in "Checks";
`ct_mocktest.py` (harness now the shared `ctmock.py`) covers them: 60 / 60. Then read live (read only) on the author's
Warrior ([port: equipment and inventory](port-equipment-inventory.md#5-read-live-2026-10-17-read-only-warrior-sl-12-things-betwixt-bonfire) section 5): E, the records, the store, all 25 lists, the equipped nodes and
mirror entries, the pointer / saved tables and 46 generator objects matched, and the table's own Lua lookups (equipment,
inventory with its consistency check, the generator scan, the map vote) ran against the live process without a write.
One correction: a list holds every owned item of its menu category (weapons by WeaponTypeParam +0x130), so on retail
the Lua never moves nodes between lists (`LIST_BY_ITEM`); the demo's 0x8c / 0x96 "hands' lists" were its character's
two weapon categories. Untested: the writing paths (README steps 7-9). Also from [the first live read](port-live-check.md): min HP's normal
value is -99999 (profile `MINHP_NORMAL`, restored by "Player cannot die"), chr+0x108 is not the HP cap.

## Related pages

- [Draw the whole map on PC](draw-whole-map-pc.md): the "Draw the whole map" entry carried one build further, to PC Scholar 1.0.3.0, with the retail chain of section 3 beside its PC counterpart.
