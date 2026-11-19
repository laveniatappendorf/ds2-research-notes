# First live read of the retail table's values (2026-10-17, read only)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

The author's retail boot: RPCS3 0.0.43-20315 (Vulkan, 60 FPS), BLUS41045 v1.00, a new Warrior (SL 12) at the Things
Betwixt bonfire. Read with `..\..\tools\vmem.py checks` (host-side ReadProcessMemory; nothing written). Save backed up
first: retail saves to **`dev_hdd0\home\00000001\savedata\NPUB31358-GAME_000`** (not the disc id); copy in
`..\..\saves\20261017_first_boot_things_betwixt` (46 files).

At the title screen (App state 10) the static root already read [as predicted](cheat-table-port.md#4-rebuild-started-2026-10-17-same-day): guest base 0x300000000, App = [0x1d98310]
= 0x30a00170 (vtable 0x1c1f7b8), camera manager 0x1c2f860, CharacterManager 0x1c29f08, game object [0x1d99650] =
0xd0040aa0 (the demo's address too), switch 0 / mode 1 / step 1/30.

In game (App state 30):

| Value | Predicted | Live | |
|---|---|---|---|
| player chr | [App+0x64] | 0x32727a40, vtable 0x1c2aa58 | OK ([App+0x60] holds another object, 0x354ffe30) |
| HP +0xfc | HP | 826 | OK |
| +0x100 "min HP" | -1 normal | **-99999** | the normal value differs from the demo's -1: untick must restore -99999 |
| +0x104 "max HP" | | 826 | plausible |
| +0x108 "HP cap" | | **827** | above max: not the cap (unknown) |
| stamina +0x140 / +0x144 / +0x148 | cur / min / max | 920 / -9999 / 920 | OK |
| name +0xcc | UTF-16 | not text | the name is not inline at +0xcc (pointer or other layout); unused by the table |
| position +0x70 | | -181.63, 12.01, -143.34 | OK (= the Havok body, 12.06 high) |
| S = [chr+0x94] | | 0x32729990, counters 0 at rest | consistent |
| pparam = [chr+0x378] | VGR END VIT ATN STR DEX INT FTH ADP | 7 6 6 5 15 11 5 5 5 (+0x16 = 1, +0x18 = 6), copy identical | = the Warrior class; INT / FTH / ADP all 5, so their order is still open |
| SL +0xcc / souls +0xe8 | | 12 / 0 | SL OK |
| Havok wrapper / entity / world | [chain + vtables](port-character.md#1-table) | all three vtables OK, gravity -9.80665, filter 0x10001 (layer 1) | OK |
| on-ground byte [[chr+0xb8]+8]+0xb4 | 1 standing | 1 | OK |
| OOB check [[chr+0xa8]+0x8c] | +0 = chr | OK, flag 0, timer 0 | OK |
| ChrMoveParam kill time | 7.0 | 7.00 | OK |
| player data [chr+0x2d4] | vtable 0x1c29a70 | OK | OK |
| map W = [[App+0x1c]+4] | W+4 = MM | OK; current = applied part | OK |
| [map instances](port-map-draw.md#1-table) | s16 W+0xc8 | **34** (every world map); only instance 0 (id 0x0a020000 = m10_02, Things Betwixt) has its collision list loaded: 31 parts, part vtable OK, first h01_0200_0000 | OK: draw-all walks all 34, the empty ones add nothing |
| [enemies](port-equip-ai-spawn.md#3-ai-freeze) | AI = [[[chr+0xac]+0xc]+0x10], vtable 0x1c2dc60 | 5 characters, 4 enemies with that AI | OK |

Not checked yet: anything that writes (every script entry), the INT / FTH / ADP order, +0x108, the floor-loader state,
the pad buffer, [the equipment and inventory layout](port-equipment-inventory.md).

## Script entries ticked in Cheat Engine (the author; watched read-only with `..\..\tools\vwatch.py`)

- **World speed (entry 12): writes land.** Three tick / untick cycles 10:43:51-10:44:25: tick = switch 1, mode 2, step
  0.00001; untick = 0 / 1 / 0.03333, exactly the demo's behaviour. On screen the bonfire animation slowed (did not
  freeze: some effects run on their own clock). **Side effect: the character did a "crazy ragdoll"** while it ran;
  nothing lasting in memory (same character object, HP unchanged, no position jump over 2 m, no load). Likely cause:
  bodies the animation drives get velocity = distance / step, and a 1e-5 step makes that huge (the demo note already
  warned that a step of exactly 0 must never be written). To test: a gentler step (0.002 = 1/8 speed) and when it
  happens (on pause or on resume).
- World speed, more tests 10:45-10:49: steps 0.02 / 0.025 (1.2x / 1.5x speed: the character walked normally),
  0.002 (1/8) and 0 (the entry turned 0 into 0.00001 on the tick: the guard works); **no ragdoll after the first
  round** (the author: it began immediately on ticking, the character vanished and reappeared on the untick; the physics body
  never moved, so only the visible model was thrown). Probably the pose at the first round (sitting at the bonfire?),
  not the step: to repro, rest at the bonfire and tick. Several unticks left the step at the user's value: the step row
  was most likely ticked (frozen) in Cheat Engine. Harmless while the switch is off: retail reads 0x1cac3b0 / 3a8 / 3a0
  only in the frame-step function (0xb26e18 / e20 / e28), like the demo.
- **Draw the whole map (entry 13): works.** 10:52:58 tick: 31 of 31 loaded collisions drawing every group; walking
  on, a second map instance loaded 44 more at 10:53:15 and the re-apply timer covered them within one read (75 of 75).
- **Free camera (entry 11): works, controls fixed.** [Camera slot](ultrawide.md#where-the-169-comes-from) 2 -> 3 on every tick, back to 2 on untick; the render
  camera ([App+0x878]+0xa0, vtable 0x1c36c20, feed byte App+0x87c = 1) followed slot 3 exactly: Space / C moved it up and
  down, the arrows turned it. Forward / sideways never moved because retail had no pad address (keyboard I J K L only)
  and the left stick still drove the pinned character. **Pad 0's CellPadData found at guest 0x10721bbc** (the demo's
  was 0x100aae5c): len 24, sticks +12..+19, DS3 motion sensors ~0x200 at +44..+51; the author moved both sticks and both
  traced their full range there. No static pointer leads to it (only allocator bookkeeping nearby), so the profile now
  has PAD = 0x10721bbc with PAD_SIG: ct_move.lua's ds2ntPadOK() trusts it only while len = 24 and the sensors read
  0x100-0x300, else the keyboard (the demo path is unchanged: no PAD_SIG). Tools: `..\..\tools\vcamprobe.py`,
  `vpadfind.py` (scan / narrow).
- **Unlock all bonfires (new, 2026-10-17).** [The bonfire array](../modding-world/bonfire-lit-state.md#where-the-list-lives) is where the PC Scholar tables' path halves to:
  manager [[App+0x38]+0x2c], entries [mgr+0x10], count [mgr+0x14] = 58 (MapObjectBonfireParam has 64 rows; names in
  BonfireName.fmg). Entry = 0x10 bytes {u16 bonfire id, u8 flags (0x80 = lit: the PC tables' bit 0, PPC bit fields
  run from the top bit), u8, u32 param row, 8 bytes}. The character had Fire Keepers' Dwelling (2650) and The Far Fire
  (4650) lit. Save backed up twice (`saves\20261017_114619_before_bonfire_unlock`, `..._114656_..._2`, the second
  identical to the 11:46:31 save), then `tools\vbonfire.py unlock` set 0x80 on the other 56 (log
  `saves\bonfire_unlock_log.json`, `vbonfire.py restore` undoes it while the game runs): **the author confirmed Travel
  lists all of them**, and the watcher logged a warp at 11:48:55 (App state 30-31-19-28-30, a new map of 72
  collisions, same character object; Draw the whole map, still ticked, re-applied there within 6 s). The PC tables call the game's own setter; the bit alone was enough here.
  Table entry 17 "Unlock all bonfires" does the same from Cheat Engine (mock-tested, 62 / 62).
