# Draw the whole map on PC Scholar (2026-10-30)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

The Network Test table's entry 16 ("Draw the whole map", `..\..\DS2 Network Test\tools\drawall.py`) and the retail PS3
port (`..\..\DS2 Vanilla\findings\port\map-draw.md`, DS2V entry 13, works live) ported to PC Scholar 1.0.3.0:
`tools\pcdrawall.py` and the new table `cheat-tables\DS2SPC.CT`.

**Status: works live from `pcdrawall.py` (2026-10-30)**: the author saw the hidden PvP arenas with it on. The chain and the
applied-collision offset are confirmed in the game (section 5). The table (`DS2SPC.CT`) ticks and re-applies after a
reload, but **the game crashed on the level transition into Majula while it was ticked** (section 6, not yet
diagnosed). Don't walk between maps with it ticked until that's fixed.

## 1. Nothing like it existed

- **Community tables** (`..\..\DS2 Network Test\cheat-tables`: *community advanced* / Bob Edition 4.09.5, *Phokz*,
  *basic*): no entry draws hidden map parts. Their render entries are fog, lighting, sun angle and post-processing.
  `ChrCullingGroupCtrl` (PlayerCtrl +0x240) is about which characters are drawn, not the map.
- **DebugManager v2.0** (debug copy): no draw-all switch either.
  - Its Draw System menu has the opposite: "Disable map drawing" (also characters, objects, SFX, front end).
  - The Map GUI inspects one map piece at a time (Status, Category, DrawGroup, LodLevel, polygon counts), read only.
  - Its "CollisionGroup_" 0-63 / 64-127 panel shows the **Havok collision layer** (`[shape]+0x4c & 0x3f`), not draw or
    display groups.
  - It does have a free camera (FreeCam / FreeCamLock / FreeCamTimeFreeze), useful together with this entry.

## 2. The chain

Read off two sources:
- **DebugManager's map interface code** (`dinput8.dll`, around 0x180029240), which walks the same objects:
  - holder → +0x30 → +0x38 → +0x8 = W;
  - W+0x20 = a part it records as a collision (kind 2);
  - instances [W+0x38+8i] up to a null, each matched on map id [inst+8];
  - per instance, lists at +0x158 (map pieces, max 1,500), +0x160 (objects, max 1,000), **+0x168 (collisions, max
    500, kind 2)** and +0x170 (max 200). Each list has its parts at [L+0x10] and the count (s32) at [L+0x18].
  - Its part record reads part+0x28 (map info), **part+0x30 → MSB entry** (whose first qword points at the name),
    part+0x38 (model entry) and part+0x40..+0x7f (the transform).
- **The PS3 retail chain**, with 4-byte members doubled to 8-byte ones:
  - App+0x1c is word 7, the same as GameManagerImp+0x38;
  - the community tables' GameManagerImp AOB (`48 8B 05 … 48 8B 58 38`) itself loads +0x38;
  - +0x38 is the member the exe reads most (412 reads in the functions Ghidra decompiled).

| Item | PS3 retail v1.00 | PC 1.0.3.0 | Source on PC |
|---|---|---|---|
| Root | App global 0x1d98310 | GameManagerImp static (RVA 0x16148f0, `pcmem.py` BaseA) | community AOB, RTTI |
| Map manager MM | [App+0x1c] | [GameManagerImp+0x38] | DebugManager, word-7 analogy |
| W | [MM+4], [W+4] = MM | [MM+8], [W+8] = MM | DebugManager; back pointer live |
| Current collision | W+0x10 | W+0x20 | DebugManager (kind 2); live |
| Applied collision | W+0x14 | **W+0x28** | PS3 analogy; live: the game resets it within 2 ms |
| Map instances | [W+0x1c+4i], i < s16 [W+0xc8] | [W+0x38+8i]: 38 map slots, then null | DebugManager; live (section 5) |
| Current map slot | | W+0x30 (index) | live |
| Map id | inst+4 | inst+8 | DebugManager |
| Collision list L | [inst+0xfc] | [inst+0x168] | DebugManager |
| Parts / count | [L+8] / [L+0xc] | [L+0x10] / s32 [L+0x18] | DebugManager |
| Loaded MSB entry | [part+0x18], name at entry+0x70 | [part+0x30], name at **entry+0x80** | DebugManager; the files (all 1,553 PC collisions) |
| Display mask | entry+0x40, 4 × u32 BE | **entry+0x44**, 4 × u32 LE | PC MSB layout (`msb2.py` shift 4) |
| Player | [App+0x64] | [GameManagerImp+0xD0] | community tables |

**The masks in the files**: PC Majula (m10_10) has exactly PS3 Scholar's collision masks, 4 bytes later. For each
collision the mask is bit 0 plus the collision's own bit, as on the Network Test. The RTTI part class is `MapEntity`
(vftable 0x1410e7b68, 10 slots).

## 3. Tools

- `tools\pcdrawall.py show | on | off`, the PC counterpart of `drawall.py`:
  - **on** writes all ones into entry+0x44 of every loaded collision, then sets W+0x28 to another collision so the
    game recomputes;
  - **off** writes the masks back from the MSB files in GameData (`file_masks()`, tested offline: 144 collisions over
    m10_10 + m10_04);
  - writes go through `pcmem.Game(write=True)`: Steam offline and the game's connections are checked, and every write
    is logged in `saves\pcmem_writes.log`.
- `cheat-tables\DS2SPC.CT` (built by `tools\make_pc_ct.py`, Lua in `tools\ct_drawall_pc.lua`): one script entry,
  "Draw the whole map". It works like the DS2V retail entry:
  - it finds GameManagerImp itself, with the module size and RTTI checked;
  - each collision's own mask is saved just before the first overwrite;
  - a 0.5 s timer re-applies when new maps load;
  - untick writes the saved masks back and recomputes;
  - a refusal writes "NOT ticked: <reason>" into the row.
  How to load it: `cheat-tables\DS2SPC-README.md`.
- `tools\ct_pcmocktest.py`: the table's Lua in Lua 5.3 (CE 7.4's) against a fake x64 memory image laid out as above.
  It covers tick, a map loading while ticked, untick, not in-game and the wrong exe: **all pass**. `ct_luacheck.py`:
  both halves compile.

## 4. Live checklist

1. `show` resolves: **done**. [W+8] == MM; current collision h07_0500_0000 with mask 0x21 (bit 0 + bit 5); 116 loaded
   collisions in two maps; no entry failed the name check.
2. W+0x28 is the applied collision: **done**. Pointed at another collision with no mask changed, the game put the
   current one back within 2 ms. (Not watched while walking from one collision to another.)
3. `pcdrawall.py on`: **done**. "recomputed: True"; the author saw the hidden PvP arenas.
4. `pcdrawall.py off`: **done**. Masks back from the MSB files, the extra pieces left the draw set.
5. The table: **ticks, but the game CRASHED** on the level transition into Majula while it was ticked (section 6).
   Untick not yet seen.
6. Instance array null-terminated: **done**. 38 slots, null at slot 38 (section 5).

## 5. Live results (2026-10-30)

Character standing in m10_23 (Huntsman's Copse area, collision h07_0500_0000), Majula loaded next door. Steam offline.
- **Map slots.** [W+0x38+8i] are 38 `MapAreaCtrlOwner` objects, one per game map in a fixed order (m10_02, m10_04,
  m10_10 … m50_38), then four nulls. Each slot exists all the time; only loaded maps have their part lists (+0x158
  pieces, +0x160 objects, +0x168 collisions, +0x170 a fourth list). W+0x30 holds the current map's slot index (9 =
  m10_23). Right after the area loaded only m10_23 had lists; Majula's appeared seconds later.
- **The draw-set flag.** Diffing every map piece's `MapEntity` between on and off: a piece in the draw set has bit 2 of
  the byte at piece+0x1b8 set (0x1e drawn, 0x1a not), and holds render pointers at +0xe0 … +0x1a0 that go null when it
  leaves. `pcdrawall.py show` now counts drawn pieces per loaded map with it.
- **What on changes**, m10_23 at that spot: 195 of 1,275 pieces drawn normally, 1,194 with draw-all. The other 81 have
  flag values that never include bit 2 in either state.
- **Neighbouring map unchanged.** Majula stayed at 21 of 422 drawn in both states, although its collisions got all-ones
  masks too. So draw-all opens up the map you stand in, and a neighbouring loaded map keeps its usual distant view.
  (Expected to swap over when you walk into Majula; not yet watched.)
- **Off is gradual.** Right after `off`, 394 pieces had already left the draw set and the count was 800; a few seconds
  later it was 195 and stayed there for 25 minutes.
## 6. The table live, and a crash (2026-10-30, open)

Read-only logger once a second (collision, all-ones count, drawn pieces per map) while the author used `DS2SPC.CT`:
- **Tick** (55.8 s): 116 of 116 collisions all ones; m10_23 drawn 195 → 389 → 951 → 1,194 over about 2 s, the same as
  `pcdrawall.py on`.
- **Two short resets** while ticked: at 93 s all 116 masks were back to normal for about 1 s, then all ones again; at
  157–161 s normal, all ones, normal again. Not yet known whether these were quick untick / re-tick or the game
  putting the masks back (the timer would re-apply them either way).
- **Majula unloaded** while ticked (109.5 s, walking in m10_23): no problem. Walking through m10_23 collisions
  (h07_1100, h07_0200, h07_1300, h01_0100, h01_1400) kept the draw set at 1,194–1,195.
- **A load** at about 162 s: the map draw state went null, then came back as a new object. Afterwards all 116
  collisions were all ones again, with Majula reloaded (28 of 422 drawn) and m10_23 at 1,196: the entry re-applied
  after the reload, unless it was ticked again by hand.
- **Crash**: shortly after, the game crashed on the level transition into Majula with the entry ticked. (The logger
  had stopped at the load; its restart found no game.)

Suspects, untested:
1. **A stale applied pointer.** The timer's recompute writes W+0x28 = the first collision in the list that isn't the
   current one, which can belong to a map about to unload. If the game skips the recompute during a transition, W+0x28
   keeps pointing at a freed part.
2. Masks written or a recompute forced while a map is only partly loaded (the timer fires every 0.5 s through the
   transition).
3. The much larger draw set while Majula streams in.

Next session:
- Read the crash record: Windows Application log, event 1000 for DarkSoulsII.exe (exception code + fault offset). Map
  the offset in Ghidra and see whether it reads W+0x28 / a part's entry.
- Repeat with `pcdrawall.py on` (one shot, no timer) and walk into Majula. A crash then points at the draw set itself
  (3), no crash points at the timer (1, 2).
- Likely fix for 1 and 2: recompute with a collision from the current map only (slot W+0x30), skip the timer while
  the player or any loaded map is mid-load. Keep the logger running while testing: `tools\pcdrawwatch.py 2400`
  (read-only, one line per change, survives load screens; shows W's low address bits so a reload is visible).

- Scratch tools used (not kept in the project): a map-slot probe, a per-piece snapshot / diff, a read-only logger, and a
  game-window capture that uses PrintWindow so windows in front of the game are never captured.
