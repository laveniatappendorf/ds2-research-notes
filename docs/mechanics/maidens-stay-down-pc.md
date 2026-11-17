# Why knocked-down Amana maidens never stand up (PC Scholar 1.0.3.0, 2026-10-24 / 25)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Question (the author): the Shrine of Amana's spellcasting maidens, once knocked down, never stand up again although they
aren't dead. A get-up animation exists (viewable in SmithBox). Does the AI have anything for standing up, and why don't
they?

## Short answer

- **The cause is one stray animation event.** The maidens' face-up fall animation `a00_16_0740 DmgDownFaceUPStart`
  in their network `c1050.nmb` carries a `DownDuration` event (user value 270) that the equivalent animation of
  enemies that get up does not have.
- **How it breaks the get-up** (my reading of the code; the live log matches it):
  - when the fall cross-fades into the lying loop, the loop's own `DownDuration` starts while the fall's is still
    active;
  - the fall's event then ends;
  - the game handles event 270 with a plain on / off flag, not a counter, so that end turns "down" off while the loop's
    event is still running.
  - The get-up timer needs 1.5 s of continuous "down". It reaches 0.7-0.8 s, resets, and never runs again, so the game
    never sends the network its `SitUp` request.
- **Proved live on PC:**
  - a read-only memory watch showed exactly that sequence on two sorceresses knocked onto their backs;
  - as predicted, a sorceress knocked onto her **face** (whose fall animation has no stray event) **gets up**.

  A frontal pancake puts them on their backs, which is why players see "they never get up". The healers share the model
  and network and behave the same.
- **Not the AI, not a missing animation, not their params.** The get-up animations are present and wired; no AI script
  in the game orders a get-up; maidens given the Lindelt Clerics' AI (checked in the live enemies) still stayed down.
- **The stray event alone is not enough** (correction 2026-10-25).
  - The same event, on the same track with the same cross-fades, is on the face-up fall of three more networks: c1000
    Forest Grotesques, c1320 Skeletons and c5120 Leydia Pyromancer.
  - But the Skeleton Lord fight's skeletons, which run c1320's network, **do** get up after Large Club knockdowns (the author,
    many times).
  - So something else about the knockdown has to combine with the event; the most likely candidate is which damage
    state machine the hit sends the enemy into. Section 4.
- **Fix (for a mod):** delete that event from `a00_16_0740` in `c1050.nmb`. Listed in
  `<local path> patches and fixes.md`. Untested.

## 1. What was ruled out, and how

| suspect | test | result |
|---|---|---|
| no get-up animation | `morpheme4\chr\c1050.anibnd.dcx`, timeacts | present: `a00_16_0730 DmgDownFaceDownEnd` and `0760 DmgDownFaceUpEnd`, timeact 160730 / 160760, referenced by the network |
| timeact events | all 190 PC timeacts with a down loop | the maidens' knockdown animations have no events, like 164 of the 190 (Lindelt Clerics, Aberrations ...); healer and sorceress timeacts are identical event for event |
| AI scripts | all 485 PC AI scripts | no label or state for getting up; scripts only *wait* on the engine's damage flags (`f5005` / `f5007`) |
| the zeroed EnemyParam block `Unk17_1-4` / `Unk18` (only the sorceress and three Hollow Infantry rows have all zeros) | live, Forest of Fallen Giants: axe / bow Hollow Infantry (zeros) vs knife (−1), all at 5,000 HP, Large Club knockdowns | all got up → not it |
| the sorceress's own setup (starting weapon slot 2, `Unk16` = 1) | live, Amana: sorceress and healer (slot 1) knocked down | **neither got up** → it affects every maiden; not the stance |
| the maidens' AI (logic ai105010 / ai105030 + behaviour ai105000, which **both** kinds run) | live: all three maiden spawn records switched to the clerics' AI (logic 131010, behaviour 131000); the swap was confirmed in the spawned enemies (`EnemyOperator +0xD8 / +0xDC`) | clerics got up, the maidens still stayed down → not the AI |
| EnemyParam / EnemyCommonParam / EnemyMoveParam values | fields where both maidens differ from every enemy seen getting up | only slash absorption, curse defence, souls and a hit effect → nothing plausible |
| the network's knockdown logic | `nmb.py` decode of c1050 vs c1030 (Hollow Infantry) | the seven damage state machines with down loops match pair by pair; every down loop leaves for its get-up only on `SitUp_FaceUp` (request 21) or `SitUp_FaceDown` (22), the same in both |

## 2. The get-up chain in the exe (Ghidra DS2SPC, headless)

1. **"Is down" comes from the animation.** `FUN_140369890`, the network's event-track handler, switches on each
   event's user value.
   - 270 (`DownDuration`) sets the byte `[chr+0xB8]+0x299` when the event starts and clears it when it ends: a plain
     set / clear, while its 252 / 261 / 262 neighbours count up and down.
   - 250 (`DamageDuration`) sets `+0x7E6`.
2. **The down timer** is `ChrDownActionCtrl` (RTTI).
   - **Where:** vtable 0x1410BF148; update `FUN_140133700` = slot 3. The factory `FUN_14030dce0`
     (`CharacterActionCtrl` / `EnemyActionCtrlBase` slot 4) builds it for every character, at action-ctrl `+0xF8`
     (`[chr+0xE0]+0xF8`).
   - **Arming:** a rising `+0x299` arms it (`+0x29`) and starts "down" (`+0x18`). A falling `+0x299` ends "down"; with
     "down" off, the elapsed time `+0x10` and the arming reset.
   - **Threshold** (`+0x14`) = the hit's down time (damage data `+0x18`) × `EnemyCommonParam +0x304`, which is 1.0 in
     all 225 rows. It's set by `FUN_1401336e0` from the damage code `FUN_14013b4d0`. Live: **1.5 s** for a Large Club
     knockdown.
   - **Firing:** when elapsed ≥ threshold, armed, alive (`CharacterCtrl` slot 0x1B0 = HP `chr+0x168` > 0) and not held
     by a throw (`+0x298`, set by the throw controller `FUN_14032c7d0` / `FUN_14032d2e0`), it sets **bit 40 of
     `[chr+0xB8]+0x4C0`**.
3. **The request.** `ChrDamageMotionCtrl` (vtable 0x1410E2330) does the sending.
   - Its setup, `FUN_14035e7e0` → `FUN_14035f1c0`, resolves the request names `DamageFullBody, SitUp_FaceUp,
     SitUp_FaceDown, RagDoll_End, DamageLanding, HitBack, SitDown` in the character's own network. `FUN_14035ef90` does
     the same for 13 damage control parameters (`IsDown` among them).
   - Its update `FUN_14035e820` turns bit 40 into **`SitUp_FaceUp`, or `SitUp_FaceDown` when the body faces down**, and
     clears the bit. Bits 49 / 48 / 41 / 14 send `DamageFullBody` / `DamageLanding` / `RagDoll_End` / `SitDown`.
   - `BossDamageMotionCtrl` (`FUN_14044d2e0`) does the same for bosses with breakable parts.
4. **The network** then leaves the down loop for the get-up.

Every link is shared code; the down loops are 3.0 s long in c1050, c1030 and c1310 alike; the request names resolve the
same. So the failing link had to be found live.

## 3. The live proof (2026-10-25)

`..\tools\pcmem.py downwatch 105010` (read-only). Two sorceresses near Crumbled Ruins, at 5,000 HP, knocked onto their
backs with the Large Club. Both gave the same sequence:

| time after the knockdown | "down" byte `+0x299` | down timer |
|---|---|---|
| 0 s | **1** | armed, counting |
| 0.1 → 0.7 s | 1 | elapsed 0.1 → 0.7 s, threshold **1.5 s** |
| ~0.8 s | **0** | disarmed, elapsed reset to 0 |
| afterwards, for as long as she lay there | 0 | never armed again; the sit-up bit never set |

0.8 s ≈ the 0.667 s face-up fall plus its cross-fade into the loop.

**The prediction.** A sorceress knocked down from behind, onto her face, gets up: her fall animation
(`a00_16_0710 DmgDownFaceDownStart`) has no `DownDuration`, the same as c1030's. Confirmed in game by the author. The watch
log didn't capture that knockdown: its enemy list is taken when it starts, and the one knocked onto her face was
probably not among the three it found. So this part rests on the in-game observation.

**The difference behind it** (event tracks, read with `nmb.py`):
- `a00_16_0740 DmgDownFaceUPStart`:
  - c1050 tracks: Footsteps, DamageDuration (250), **DownDuration (270, the whole clip)**, IK, MasterAdjustControlSlope,
    TimeAct;
  - c1030 tracks: Footsteps, DamageDuration, IK, InertiaMove, MasterAdjustControlSlope, TimeAct, with **no
    DownDuration**.
- The loops (0720 / 0750) carry `DownDuration` in both networks.

## 4. Other characters with the same stray event

Scan of all 173 PC networks (135 have a down loop): `DownDuration` on a fall-start animation (0710 / 0740 / 0610 /
0630 / 0210 / 0220) occurs only on `a00_16_0740` of:

| network | enemies (EnemyParam rows using it) |
|---|---|
| c1050 | Amana Shrine Maiden (healer 105000, sorceress 105010) |
| c1000 | Forest Grotesque (unarmed, bone club, rock thrower; 5 rows) |
| c1320 | Skeleton (falchions; falchion and shield; 18 rows) |
| c5120 | Leydia Pyromancer (512000) |

**But at least c1320 gets up anyway.**
- **Which network an enemy uses:** the game picks the animation network by **behaviour id ÷ 100**, not by model.
  - In 33 of the 34 kinds of EnemyParam row where the model id differs, only the behaviour id's character has an
    animation binder. The odd one out (model 9150, behaviour 8000) has a binder for neither.
  - So the Skeleton Lord fight's skeletons (rows 157001-157033, model c1570, which has no binder of its own; behaviour
    132000-132030) run **c1320's** network.
- **They get up.** the author has knocked them down with the Large Club many times and they don't get stuck.
- **The network data doesn't explain the difference.** The stray event's track position (track 3 on the fall, 4 on the
  loop, channel 4) and the fall-to-loop cross-fades (0.1 s from the start switch, 0.2 s on the fall path) are
  identical in c1050, c1320, c1000 and c5120.
- **So the stray event is necessary for the maidens' failure (the face-down control) but not sufficient.** Something about
  how the knockdown plays out must also differ. Candidates, none checked:
  - **the damage path:** which of the seven damage state machines the hit selects (e.g. a cross-fade transit, where the
    loop's event starts before the fall's ends, versus the blow-away `Sequence` state, where the clips play back to
    back), which may depend on the victim as well as the attack;
  - **how they land:** skeletons landing face down more often;
  - **skeleton-specific behaviour:** e.g. collapsing and reassembling instead of the down loop.

  A `downwatch` on skeletons (rows 1570xx in the fight, or 1320xx in Harvest Valley) would show which.
- Whether the Forest Grotesques and the Leydia Pyromancer stay down is untested.

## 5. Fix options

- **Data (mod):** delete the `DownDuration` event from `a00_16_0740` in `c1050.nmb` (and c1000 / c1320 / c5120), or
  end it before the loop's begins. That needs repacking the NMB into `morpheme4\chr\c1050.anibnd.dcx` and loading the
  edited archive file on PC (unpacked game files via Nuxe, or a mod loader). Not tried.
- **Engine:** make the 270 handler count starts and ends, as its neighbours do. That would be a code patch in
  `FUN_140369890`.

## 6. Tools and live setup made for this

- `..\..\DS2 Network Test\tools\nmb.py` (new, shared) reads PC 64-bit morpheme networks:
  - bundles, nodes, names, requests, event tracks and the file lookup, from LordRadai's MorphemeConnect source;
  - state-machine definitions (attribute type 38), worked out here;
  - `nmb.py <game> <chr> nodes | anims | sm | downexits`.
- `..\tools\pcmem.py` additions (writes are offline-gated and logged in `..\saves\pcmem_writes.log`):

  | command | what it does |
  |---|---|
  | `setstats` | set stats; soul level kept consistent |
  | `bonfires [light]` | list or light bonfires |
  | `nodeath` | minimum HP 1, checked every 0.5 s |
  | `eparam ... hp / slot / set / like / restore` | edit EnemyParam rows in memory |
  | `regist ... ai / restore` | edit spawn records' AI ids in memory |
  | `downwatch` | read-only knock-down timer watch |
  | `near` | nearest spawns (doesn't work yet: the live position uses another origin than the map files) |
- `..\Ghidra Project\scripts\DecompRange.java` decompiles every function in an address range. The Ghidra project
  runs headless via PowerShell: `& "...\Ghidra Project\script.cmd" X.java args`.
- **Live layout found:**
  - `CharacterCtrl +0x28` = behaviour id;
  - `+0x38` → its EnemyParam row;
  - `+0x168` = HP / min / max / cap;
  - `+0xE8` → `EnemyOperator` (AI ids at `+0xD8` / `+0xDC`);
  - generator objects (0xA0 apart) point at their EnemyParam row;
  - a bonfire rest respawns enemies from the in-memory `GeneratorRegistParam` rows.
- **The test save** now has STR 26 / DEX 4 / SL 180 (written for the Large Club), and all 77 bonfires are lit. All
  param and spawn-record edits were memory-only and ended with each game session.
- **Incident:** the game hung once on a bonfire warp. At the time the maidens' spawn records were swapped and the first
  no-death loop was topping up HP every 0.1 s. Which caused it is not known; `nodeath` now only touches the minimum HP
  every 0.5 s.

## 7. Not checked

- The PS3 builds (v1.00, 1.10, PS3 Scholar). The quirk is known from every version, but `nmb.py` doesn't read the
  32-bit big-endian PS3 networks.
- Why c1320's skeletons get up despite the same stray event (section 4), and so exactly which damage path makes the
  event fatal. A `downwatch` on skeletons, and on the maidens' different knockdown types, would answer it.
- Whether the Forest Grotesques and the Leydia Pyromancer stay down after a face-up knockdown.
- The exact frame order of the two `DownDuration` events at the cross-fade (inferred from the 0.8 s window and the
  set / clear handler).
- A patched `c1050.nmb` in game.
