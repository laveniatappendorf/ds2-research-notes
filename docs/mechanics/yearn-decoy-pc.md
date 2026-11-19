# Yearn and the Alluring Skull: how the decoy works, and which enemies take it

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

PC Scholar of the First Sin 1.0.3.0 (`DarkSoulsII.exe`, Ghidra project DS2SPC, image base 0x140000000). Worked out on 14 Nov 2026 from a live test
(offline, vanilla Scholar copy, a disposable level-838 character), live memory recordings, the params and AI scripts, and Ghidra.
Written up after the work stopped because it grew more complex than planned: **what is solid, what is only predicted, and what is open** are
kept apart below. The open tests are in the beads tracker `<local path> Yearn Decoy` (prefix `decoy`).

## 1. Summary

- **Yearn** (sorcery 31220000) and the **Alluring Skull** (consumable 60530000) are the only two things in the game that make enemies walk to an object instead of the player.
  Each throws a harmless bullet that the exe turns into a target ("decoy") that enemies can pick.
- **It is hard-coded in the exe**, keyed on two BulletParam row ids: **31220010** (Yearn's orb, 10 s) and **60530010** (the skull's orb, 5 s). No param row says "decoy".
- **Which enemies react is decided per enemy**, mainly by two bytes in the enemy's `LogicComParam` row: **Unk67 = 1 takes Yearn, Unk66 = 1 takes the skull**.
  A second condition is also needed (the Ogre shows this). **Found in the code (14 Nov, late; static only, not yet confirmed live):** the enemy only goes for the orb when its AI
  script runs its target search (command 7000) while the orb exists, and that search looks for a decoy **within the command's range (20 m in the Hollow Soldiers' idle script,
  50 m in the shared combat framework), by plain 3D distance: no vision cone, no line of sight**. Enemies whose current script never searches (the Ogre's, Giant's and
  Forgotten Giant's default scripts) never react. Details: section 2a.
- **Giants:** Giant (Unarmed / Club / Pyromancer) in the Giants' memories **take Yearn, not the skull** (seen live). Forgotten Giants, the Last Giant and the Giant Lord take neither.
  Ogres take neither in practice. Dogs: Stray Dogs take neither (seen); Hunting Dogs are unknown.
- **Range:** every loaded enemy with the flag is told about the orb at any distance, but they only walk to it within about **16 m** in the one measurement made (cut-off between 16.0 m and 16.4 m),
  and some soldiers close by never react. The code predicts 20 m (idle soldiers) / 50 m (enemies in combat), 3D, walls do not matter, and close soldiers ignore it when
  their current state is not searching. Not yet tested live.

## 2. The mechanism, step by step

1. **The spell.** SpellParam 31220000: 19 INT, 1 slot, 4 to 8 casts by attunement tier. Bullet chain: `31220000` (travels 1.25 s at 10 m/s, so the orb lands ~12 m ahead) -> `31220001` (0.5 s)
   -> `31220010` (the orb: 10 s life, sfx 5026, damage multiplier 0, no SpEffect). The skull: item 60530000 -> bullet `60530000` (5 s) -> child `60530010` (5 s, sfx 846).
   BulletParam 60531000 / 60531010 also exist in the 6053 family; the hook below ignores them. All the damage rows share the "no damage" DamageCtrl 99986021, so nothing in the params marks the orb as a decoy.
2. **The hook.** `FUN_1404477c0` runs when a bullet is created and compares its row id (at bullet + 0x2C) with `0x1DC612A` (= 31220010) and `0x39B9D5A` (= 60530010).
   For those two it builds a **`TargetBulletCtrl`** (`FUN_140247d20`; vftable 0x1410cfd88; its type, from vtable + 0x28, is **4**; field +0x20 = 1 for the skull, 0 for Yearn),
   and registers it with the **TargetManager** (`[GameManagerImp + 0x48]`, `FUN_1404208c0`) and the **AiManager** (`[GameManagerImp + 0x28]`, `FUN_140422d90`).
   It skips registration for some owners (checks `FUN_14016f700 / f800 / f7b0 / f8f0 / f7c0` on the bullet's owner).
3. **The announcement.** `FUN_140422d90` walks the AiManager's list of `ChrAi` objects (list head at +0x10) and calls each one's vtable +0x58 = **`FUN_1404261a0`** with the new target.
4. **The first gate (a param flag).** `FUN_1404261a0` accepts the target only if the ChrAi is active (`ChrAi + 0x5E` bits 0 and 1) and the enemy's **LogicComParam row** (`ChrAi + 0x2B8`; the row for the spawn record's
   `LogicParamID`) has byte **+0x67 (`Unk67`) = 1** for a target with flag 0 (Yearn) or byte **+0x66 (`Unk66`) = 1** for flag 1 (skull). An enemy with no row never sees a decoy.
   **There is no distance test in this step.**
5. **Taking it.** An accepting enemy's `ChrAi + 0x90` (its perceived target, normally a `TargetCharacterCtrl` = the player) becomes the orb's `TargetBulletCtrl`. It is cleared (0) when the orb expires, then returns to the player.
6. **Using it.** The AI script's command **1:7301** calls `FUN_1404265f0(ChrAi, 1)` (sets `ChrAi + 0x350` bit 0x200, "use the target"); **1:7302** calls it with 0 (bit 0x400, off); `FUN_1404395e0` is the dispatcher. The per-frame
   update `FUN_140427410` then copies `ChrAi + 0x90` into the enemy's output record and sets **`ChrAi + 0x3A9` = 1** (the "has target" byte; cleared with 0x400). It rejects targets of type 2 and 3 (bullet = 4, so it passes).
   The AI conditions: `f7301(n)` = `FUN_14043c160` (tests the goal's target type; n = 4 means no target), `f7302` = `FUN_140425380` (reads `+0x3A9`), `f7303` = goal + 0xAC.
7. **The walk.** In the shared logic framework (soldier `ai102010` and Ogre `ai300010` alike) the battle states run 1:7301 on entry (AI mode slot 0 = 10, entered from the
   perception loop when `f7303` and `f7300` are true; mode 15 also runs it). What makes an enemy actually go for the orb is the **search command**: see section 2a (static, 14 Nov late).

## 2a. Static trace, 14 Nov (late): what makes an enemy actually go for the orb

Ghidra DS2SPC, read through a headless copy of the project (the MCP server was locked by another session). **Code reading only; none of this is confirmed live yet.**

**The AI's target lives in `SearchEzStateCtrl`** at `ChrAi + 0x70` (built by `FUN_140431280`; vtable 0x1410edf48: +0x30 command dispatcher `FUN_1404395e0`,
+0x40 condition dispatcher `FUN_1404319b0`, +0x48 decoy setter `FUN_1404399b0`). Its fields: +0x10 the ChrAi, +0x18 / +0x20 the target handle / target object
(+0x20 = `ChrAi + 0x90`), +0x48 the last search range, +0xA0 search cooldown (`ChrAi + 0x110`), +0xAC "search ran" byte (`ChrAi + 0x11C`), +0xF8 + i x 0x20
five candidate-target slots (target +8, its character +0x10, perception bits +0x1D: 0x04 in sight, 0x08 line of sight clear, 0x40 hostile).

**The AI script functions of the 7300 series** (condition dispatcher): `f7300` = has a target (+0x20 != 0); `f7301(n)` = `FUN_14043c160`, target type test (n 0 / 1 / 2 / 3 =
type 0 / 1 / 3 / 2, 4 = no target, 5 / 6 = a relation test, 8 = another); `f7302` = `+0x3A9`; `f7303` = +0xAC (the last search ran); `f7304` = `FUN_14043af10`; `f7305` = +0xAD.

**Step by step:**

1. **Announcement (gate 1, no range).** `FUN_1404261a0` (flags, section 2 step 4) calls the decoy setter `FUN_1404399b0`: it replaces the target with the orb **only if the
   current target belongs to the orb's owner** (the caster; for an idle enemy the default target is the player), and also points `[chr + 0xC8] + 0x20` at the orb
   (probably a look-at). This alone makes nothing happen: 17 of 18 soldiers took the orb this way and most never moved.
2. **The search command.** AI scripts call `1:7000(range, pick, ...)` (handler `FUN_14043c450`; 7015 shares it). When its cooldown is over it sets "search ran" = 1 and,
   **if its 7th argument is not 0, first runs the decoy search `FUN_140439a90`**: TargetManager bullet targets (`FUN_1403b87f0`, bucket 4) of the kinds the enemy's
   LogicComParam allows (+0x66 skull, +0x67 Yearn), **3D distance from the enemy below the command's first argument**, nearest first. **No cone, no line of sight, no team test.**
   If one is found it becomes the target and the normal search is skipped.
3. **Otherwise the normal search** (`FUN_14043db50`) **clears the target** and picks from the candidate slots by perception: `FUN_14043bef0` (in sight: within
   `detect_distance` +0x0C always; else within `sight_distance` +0x08 (13 m) or `Unk04` +0x04 (20 m) [depending on a character state](earthen-peak-aggro-pc.md#4-light-vs-dark-same-in-both-builds-checked-2026-11-02) (+0x380 object, byte +0x0A) and `Unk32`;
   a larger constant when the target's owner has `[chr + 0xB8] + 0x6D4` set; inside the cone +0x10 / +0x14, all-round in a state at ChrAi + 0x960) and a raycast
   `FUN_140428f50` for line of sight. The orb never gets these bits (it is not a character), so a normal search **drops** the orb.
4. **Reaction.** The shared logic framework enters battle (mode 10, whose entry runs 1:7301 -> `+0x3A9` = 1) when `f7303` and `f7300` are both true, i.e. a search just ran and
   left a target. So an enemy goes for the orb exactly when **a search with the decoy argument runs while the orb is within that search's range**.
5. **Which scripts search, and how far** (scan of all 485 scripts; `tools\yearn_decoy_table_pc.py`): 282 scripts have a 7000 / 7015 search, 279 with the decoy argument on.
   Hollow Soldier / Infantry default scripts (`ai102000`, `ai103000`): **20 m** (a look-around routine). The shared logic framework (`ai102010`, `ai121010`, `ai300010`, ...):
   **50 m** (its perception loop, probably only in alert / combat states). **No search at all** in the default scripts of the Ogre (`ai300000`), the Giant (`ai121000`), the
   Forgotten Giant (`ai309000`) and the Stray Dog (`ai227000`). The Hunting Dog default (`ai209000`) searches 100 m with the decoy argument **off**.

**What this predicts (to test):**

- Idle Hollow Soldiers: react within **20 m 3D** only if their spawn's idle behaviour searches (think 111 / 113: few spawns, section 2b); behind a wall too. Once their AI is active (`f5621`): up to **50 m**, and the orb **beats the player** (decoy search first).
- Close soldiers that "never react" were in a state that did not search during the orb. The logger's `f7303` / searches column will show it.
- The Ogre takes the skull (flag 1) but its idle default script never searches, so it does nothing. **An Ogre already fighting you should take a skull within 50 m.**
- The community quirk "stands still near a skull out of sight, ignoring the player": no line of sight is needed, so an enemy within range takes the skull as its battle target, then cannot path to it. While the skull is its target it ignores the player.
- No map-wide hijack: an enemy far from the orb drops it at its next normal search and can pick the player up again (some delay possible).
- Live results so far fit: Giants (logic 50 m search, in combat) took Yearn and walked; Forgotten Giants (flags 0 / 0) never; the Ogre (idle, no search) never moved.

## 2b. Which states search, and when (static, 14 Nov late; tracker decoy-2u1.6)

Read from the AI scripts' call graphs (sub-machine calls = command `6:<group>`) plus Ghidra. **Static only; the key condition is still unidentified.**

- **Two scripts per enemy.** The *logic* script (`ai<LogicParamID>`, e.g. `ai102010`) is a framework shared by nearly every enemy. The *default* script (`ai<DefaultLogicParamID>`, e.g. `ai102000`) holds the enemy's own behaviours as root groups. The framework calls into it by group id.
- **The AI mode** = `f5900(n)` = byte `ChrAi + 0x3A1 + n` (`FUN_140425100`; 8 slots, written by command 1:5300(n, value)). Slot 0 drives the framework:
  - 0 = its own search loop
  - 10 = battle
  - 11–14, 17, 18, 20 = other framework behaviours
  - 15 / 16 = handed to a default-script behaviour
- **The framework's search (50 m).** Root group 10 waits until **`f5621`** is true (and 1 s has passed), then runs its main loop (A13). In mode 0 that loop calls A1: a decoy-enabled `7000(50, ...)` search, repeated about every 3 s until it finds a target. It searches again after a battle state. `f5621` = ChrAi vtable +0x120 -> `FUN_1403144c0` -> `FUN_14036da20`: `o = [[chr + 0x100] + 0x10]`; true if `o + 0x108 != 0`, or `[chr + 0xC0] + 0xF4` bit 4, else byte `o + 0xC4`. **What sets it is not found.** It is either "the AI is active (near the player)" or "the enemy is alert", and those predict opposite things for unaware enemies.
- **No throttle.** The search cooldown (`SearchEzStateCtrl + 0xA0` = `ChrAi + 0x110`) is only ever decremented or set to 0, never above 0, so a search state searches as fast as it is entered.
- **Lower state maps don't search.** These are the default script's groups 10–60, one per spawn, from `EnemyDefaultLogicParam[DefaultLogicParamID].Unk01`. Only the Charred Loyce Knights' map has a search (3.5 / 5 m); the rest just wait in modes 0 / 15 / 16.
- **The idle behaviour is per spawn point: `GeneratorParam.AIThinkID`** = the default script's root group that runs while the enemy is idle. The values match the root ids: 10–60, 100–116, 150, 200–250, 219 / 220, 310 / 320, 400 / 410. Some of these include a 20 m decoy search:
  - Hollow Soldier 111 (wait and search) and 113 (look around)
  - Hollow Infantry archer 112
  - Royal Swordsman 102
- **Per generator** (`findings\yearn_decoy_generators_pc.tsv`, `tools\yearn_decoy_table_pc.py`): of **2,458 spawn points**, only **103** have an idle behaviour with a decoy search, and only **15 also have the Yearn flag and 13 the skull flag**. **1,156** have the framework's 50 m search plus a decoy flag. Forest examples:
  - Hollow Soldiers 2501–2504, think 111, 20 m, together near Cardinal Tower
  - archers 2031 / 2033, think 112, 20 m
  - most other Forest soldiers are think 10 / 100 / 150 etc., with no idle search

**So:** an unaware enemy reacts only if its spawn's idle behaviour searches (rare, 20 m), or if `f5621` is true while it is in the framework's mode-0 loop (50 m, within about 3 s). The first range run fits "most soldiers there were not in a searching state". Whether `f5621` is "near the player" or "alerted" decides whether Yearn lures *unaware* enemies at all: the next live run reads it directly (`decoy_watch_pc.py` logs mode slot 0, `f5621` and the spawn's think id).

## 2c. Ogres, and chaining skulls (static, 14 Nov late; the author's two questions)

**Is the Forest Ogre special?** There are 12 Ogre spawn points (`yearn_decoy_generators_pc.tsv`), and the Forest one (m10_10 generator 8000) differs from all the others:

- It is the **only Ogre with idle behaviour 235**. That is a route/patrol routine (sub-machine A45(11, 9, 1, 1, 1, 2.5, 5): walk commands 5804 / 5466 and timers, no target search). The others use 10 (Things Betwixt, Aldia's Keep 5050, Shrine of Amana), 20 (Aldia's Keep 2500), 111 / 112 (Aldia's Keep).
- It alone sets generator fields `Unk78 = 80000000` (other users hold 5000xxxx region-like ids, e.g. the Majula NPCs and boars; possibly the route or an area) and `Unk80 = 80` (−1 by default; 40 for invading NPCs, 8 for Royal Swordsmen). Meanings not known.
- Leash 80 m (others 18–100).
- Its perception row is the **same as most Ogres** (LogicComParam 300010: sight 13 m, 20 m in the other state, detect 1.8 m, cone 1.05 / 0.785 rad). So its small aggro range is not from sight values; the idle routine and the two unknown fields are the candidates.

**For decoys all Ogres look the same in the code.** The Ogre default script `ai300000` has no decoy search in any behaviour, so no idle Ogre takes a decoy. All of them (flags skull 1 / Yearn 0) can take the skull only through the shared framework's 50 m search, once `f5621` is on. Test another Ogre to see whether the Forest one is really less reactive (tracker).

**Can skulls be chained with no gap?** Code reading (`FUN_1404399b0`, the TargetCtrl classes; shared framework `ai102010` groups A5 / A9 / A0):

1. **A new skull does not steal enemies from an old one that is still alive.** At announcement the setter swaps only if the enemy's current target belongs to the caster. It compares `current target vt+0x30` with the orb's owner (`[orb + 0x18] + 0x78`). For the player's `TargetCharacterCtrl`, vt+0x30 is the player's character (match). For an old orb's `TargetBulletCtrl`, it is the bullet object (never a match).
2. **When the old skull ends** (5 s after it lands), the target handle goes invalid and is cleared, and the battle flag drops (`FUN_140439c40` -> `FUN_1404265f0(ChrAi, 0)`). In the framework's battle state (A5) "no target" (`f7301(4)`) sets mode 11 and waits 1 s. Mode 11 runs A9, which waits 3 s. Only then does the framework return to mode 0 and its search loop.
3. **The next search takes a decoy first** (if one is within 50 m and allowed by the flags), before it looks for the player.

**[Corrected by live run 2, section 4a: the observed gap was about 1 s, the time to re-acquire the player via mode 12, and a new skull was taken at once. The 4 s figure below was the A9 path and did not happen.]** So there is always a gap of about 4 s after each skull ends. In that gap the enemy has no target and does not search: it is passive, but it is also not looking for you. Then it takes whichever skull is alive at that moment, or reacquires you if it can perceive you.
- **Practical timing:** the next skull must still be alive about 4 s after the previous one ends. A skull thrown while the previous one is still targeted is wasted unless it lands within about 1 s of that one ending. Best window: from about 1 s before the old skull ends to about 4 s after.
- With that rhythm (about one skull every 6–9 s) the enemy alternates "on the skull" (5 s) and "lull" (about 4 s) and should not re-acquire you, unless it is hit or the timing slips.
- Untested. The 1 s / 3 s waits are the framework's (`f3() >= 1` in A5, `f3() >= 3` in A9); idle behaviours from the default script are not involved once the enemy is in the framework's battle state.

## 3a. The lists (14 Nov, late; after the live runs)

**Files:** `yearn_decoy_lists_pc.xlsx` (sheets *Enemies*, *Spawn points*, *Notes*), plus the same data as `yearn_decoy_enemies_v2.tsv` (one row per EnemyParam id) and `yearn_decoy_generators_pc.tsv` (one row per spawn point). Built by `tools\yearn_decoy_table_pc.py`. The first draft `yearn_decoy_enemies.xlsx` is left as it was.

Each spawn point's idle behaviour (AIThinkID) is classed by the rule checked live on 13 spawn points (12 Cardinal Tower soldiers and the Forest Ogre; all 13 match):

| Class | Rule | While idle | Spawn points |
|---|---|---|---|
| **framework** | AIThinkID 10–60, the shared combat framework drives it | goes for the decoy, up to 50 m by the script (live: up to 17 m seen) | 1,399 |
| **searches** | a special behaviour with a decoy-enabled search | goes for it within that range (static only) | 91 |
| **close guard** | a special behaviour that uses its target directly | only up close; distance hint from the script (live: 103 within ~6 m) | 84 |
| **ignores** | other special behaviours | takes it as a target, does nothing | 737 |
| **unknown** | behaviour not found in the script | – | 147 |

With the flags applied, across 2,458 spawn points:
- **Yearn:** 644 would go for it while idle, 52 only up close.
- **Skull:** 564 would go for it while idle, 37 only up close.

Per enemy type (759 EnemyParam ids): 210 have the Yearn flag and 164 the skull flag. All of those take a decoy within 50 m once fighting you. Of them, 162 / 129 have at least one spawn point that takes Yearn / the skull while idle.

Still assumed: the enemy's AI must be running (`ChrAi + 0x5E` bit 0, rule unknown). The framework's 50 m is the script's figure.

## 3. Which enemies react

The data table is `yearn_decoy_pc.tsv` (818 spawn kinds = EnemyParam id x LogicParamID) and the spreadsheet `yearn_decoy_enemies.xlsx` (same, with a "Confirmed live" column).
Columns: the two flags, whether the logic / default AI script contains the condition `f7304`, the predictions, and the live notes.

**Review, 14 Nov (late): `f7304` is a red herring.** In the scripts that have it, it sits only in a small shared sub-machine the compiler copies in (group 2147483630 in `ai102000`, `ai103000` and `ai121000`,
identical in all three: it picks one of three argument sets when `f7304(2)` holds and `2 < f5703() < 99`). It is not a decoy handler. The soldier's and the Ogre's logic scripts (`ai102010`, `ai300010`) are built on the same framework,
and both call 1:7301 (sets `+0x3A9`) on entering their battle states. The second gate is the search command (section 2a). **New table: `yearn_decoy_pc_v2.tsv`**
(`tools\yearn_decoy_table_pc.py`): flags plus the decoy search ranges in each enemy's logic and default script; 244 spawn kinds can react to Yearn, 197 to the skull, "when that
search runs" (which state searches is not modelled). The old TSV and the spreadsheet below are kept unchanged. Hunting Dogs (flags 1 / 1): can react through the 50 m framework search;
their own default script's 100 m search has the decoy argument off.

**Earlier reading (kept for the record):** the `f7304(2)` condition is only a correlation. It appears in 21 of 485 AI scripts (102000, 102100, 103000, 103100, 108000, 113000, 121000, 124000, 132000, 135000, 152000, 306000, 306001, 313000, 319000, 319001,
666000, 810003, 810004, 820002, 820003), always in the default-logic script of the enemies that reacted live and never in the scripts of those that did not. Its code (`FUN_14043af10`, mode 2) is a spatial query that compares hit objects with this enemy's
character, so it may be an incoming-projectile check, **not obviously a decoy test**. The 1:7301 / f7301 / f7302 / f7303 machinery exists in nearly all scripts (444 / 485 contain 1:7301), Ogres included, so it is not the gate either.
The prediction columns ("YES" = flag set **and** `f7304` present; "flag only" = flag set, no `f7304`) are therefore **predictions**, strongest where live results agree. Over all 818 spawn kinds: 102 predicted to react to Yearn, 85 to the skull,
142 / 112 "flag only", the rest not at all.

### Live results (14 Nov 2026)

| Enemy | Flags (Unk66 skull / Unk67 Yearn) | `f7304` script | Yearn | Alluring Skull |
|---|---|---|---|---|
| Hollow Soldier / Infantry / Rogue (Forest) | 1 / 1 | yes | **reacts** (archer + melee, 4 of 4 casts; also in the range run) | not tested |
| Giant (Unarmed, Club, Pyromancer), Memory of Vammar | 0 / 1 | yes (121000) | **reacts**: all three took the orb as target, together; the first walked to it during a scripted attack | **no effect** (several thrown; never a 5 s target) |
| Forgotten Giant, Memory of Jeigh | 0 / 0 | no | **ignores** | **ignores** |
| Forgotten Giant, Black Gulch | 0 / 0 | no | **ignores** | not tested |
| Ogre, Forest | 1 / 0 | no | **ignores** | **takes the target (5 s) but does nothing** (4+ throws, one landing right in front of it) |
| Stray Dog, Lost Bastille | 0 / 0 | no | **ignores** | not tested |
| Bosses (Last Giant, Giant Lord, Pursuer ...) | 0 / 0 | no | The author: all ignore (not each tested) | not tested |
| Hunting Dog (Aldia's Keep, Gutter) | 1 / 1 | no | **unknown** (flag says yes, script says no) | unknown |

The Ogre is the key observation for the second gate: the flag gate passed (its `ChrAi + 0x90` switched to a `TargetBulletCtrl` for ~5 s on every throw) but `+0x3A9` stayed 0 and it never moved.

## 4. Live evidence behind the mechanism

- **Soldiers, 4 casts** (a recording of ~18 soldiers' AI objects at 5 Hz): 17 of 18 switched `ChrAi + 0x90` to a `TargetBulletCtrl` at the first cast, including 15 that never moved. Only the archer (103040) and a melee soldier (103010) also set
  `+0x3A9` = 1 and walked to it. `+0x3A9` followed the orb: 0 -> 1 at the cast, back to 0 about 9 s later, again for each cast; it never changed in the controls. A different bullet object each cast.
- **The fixed player slot:** `ChrAi + 0xA10 / +0xA18` (a `TargetCharacterCtrl` and the player's `PlayerCtrl`) never changed while an orb was out: that is the player slot, not the decoy.
- **Giants, 3 casts:** all three Giants switched to the same `TargetBulletCtrl` at the same instant (about 36.6 s, 54.1 s and 87.4 s into the log) for the full ~10 s; no 5 s bullet target ever appeared during the skull throws.
- **Ogre + skull:** `+0x90` = `TargetBulletCtrl` for 5 s on each throw, `+0x3A9` = 0 throughout.

## 4a. Live run 2: Alluring Skulls at Cardinal Tower (14 Nov, evening)

`tools\decoy_watch_pc.py 150 --trace` (log `decoy_watch_log.tsv`, trace `decoy_watch_trace_20261114-190827.tsv`). 7 skull orbs (8 thrown) landed 4–15 m from the author, 1.7–40 m from the 12 watched Hollow Soldiers.

**Spawns identified** (live position + (−208.0, −10.51, +132.98) = [MSB spawn position](../modding-world/warping-by-script.md#the-coordinate-frame); all 12 within 0.7 m; now built into the logger's `--map`).

| Spawn (gen) | Enemy | Idle behaviour (AIThinkID) | Result |
|---|---|---|---|
| 2235 | Hollow Soldier (Spear & Shield) | **10** (plain: the framework drives it) | **Took all 7 skulls** (17.4 m the first time). Each time it raised the flag about 1 s after the skull appeared and went into battle (modes 17 / 10), then walked to the skull (within 1.7–2.4 m). When a skull ended: target cleared, mode 12, and it **re-targeted the player about 1 s later** (5 of 5 times). |
| 2240 | Hollow Soldier (Bastard Sword) | **103** (guard: uses a target that is within 6 m and in front) | Ignored skulls at 9.3 m and 12.3 m. At **5.8 m** raised the flag in 0.4 s, went to mode 15 and **stayed alert (flag 1, mode 15) for the rest of the run without leaving its post** (moved ≤ 0.5 m). |
| 2236, 1280, 1300, 1401, 1410, 1411, 2200, 2261–2263 | soldiers / archer | 310, 109, 100, 219, 410 | **Took every skull as their target** (announcement, any distance) but never raised the flag, never moved, stayed in mode 0, and never engaged the player either. |

What this shows (live, verified):

- **The announcement reaches every flagged enemy whose AI is running.** All 12 had `ChrAi + 0x5E` bits 0 and 1 set, and all 12 switched `+0x90` to each skull at 0.0 s, at 1.7 to 40 m.
- **Whether it then acts depends on the spawn's idle behaviour (AIThinkID), not on distance alone.** Plain behaviour 10 hands control to the framework, which goes for the skull. Guard 103 uses only targets within about 6 m. The special behaviours 100 / 109 / 219 / 310 / 410 ignore both the skull and the player while idle.
- **`f5621` was 1 for every enemy, aware or not.** It is not "alerted". The bit that differed between areas was `+0x5E` bit 0: off for all 7 enemies in Things Betwixt (no search ever run since load), on for all 12 here.
- **A new skull takes an enemy off the player at once.** Its current target belongs to the caster, so the announcement swaps it (gen 2235, skulls 3–7, while it was fighting the author).
- **The first range run fits the same two spawns.** Its "flagged on all 5 orbs but barely moved" soldier (102010) = guard 2240; its "16 m, walked 17 m" soldier (102000 = Spear & Shield) = gen 2235.
- **Correction to section 2c's chaining prediction.** After a skull ends, gen 2235 did **not** sit in a 4 s lull. It went to mode 12 and re-acquired the player in about 1 s, and a skull landing 3.1 s after the previous one ended was taken at once. So the framework's lost-target path in this state is mode 12 (A7), not mode 11 (A9). Still untested: a new skull landing while the old one is still the target (predicted not to take over).

## 4b. Live run 3: overlapping skulls (14 Nov, evening)

`decoy_watch_pc.py 150 --trace --map m10_10_00_00` (trace `decoy_watch_trace_20261114-191632.tsv`, now with the target object address per sample). 16 skulls at Cardinal Tower: 8 overlapping pairs (B landed 2.0–4.3 s after A, while A was alive) and 2 singles. Subject: gen 2235 (Spear & Shield, think 10), the soldier that goes for skulls.

| Situation (gen 2235) | Result |
|---|---|
| B lands while A is his target | **Stays on A** (8 of 8). The target object does not change when B appears. |
| A ends | Target none for **1.0 s** (1.0–1.1 s every time; mode 12), then **takes B** (8 of 8; modes 17 / 10, flag 1) |
| Last skull ends, none left | Re-targets the player after 0–1.0 s |
| A skull lands while he is fighting the player | **Takes it at once** (9 of 9) |

- **Confirms the static reading:** a new decoy never replaces a live decoy as the target of an enemy that is acting on it (the setter only swaps targets owned by the caster).
- **The gap is about 1 s,** not the 4 s first predicted. During it the enemy has no target; it does not turn on the player unless no skull is alive when the gap ends.
- **Chaining rule (live):** land each skull 1–4 s after the previous one, so the next is still alive 1 s after the current one ends. With throws every 2–4 s the soldier stayed on skulls continuously apart from the 1 s blanks.
- **Idle, non-acting soldiers behave differently.** Spawns with behaviours 100 / 109 / 219 / 310 / 410 (and the guard 103 once alert) **did** move their target to the newer skull while the older one was alive, 0–1.9 s after it appeared. Their idle routines seem to re-pick a decoy about every second, probably the nearest (search commands 7003–7007 have 1 s cooldowns); not traced. It makes no visible difference, since they don't act on it.
- **Tool note:** TargetBulletCtrl objects are pooled; the same address comes back for later skulls, so identify skulls by time, not address.

## 5. Range (one measurement, 5 Yearn orbs)

Setup: Cardinal Tower, 12 Hollow Soldiers tracked, the orb landing 14 to 16 m in front of the player (orb height 3.1). Tool: `tools\decoy_range_pc.py` (read-only; log `decoy_range_log.tsv`).

- **Taking the orb has no range.** All 12 soldiers (4 to 36 m from the orb; some 12 m below it, one 8 m above it) switched `ChrAi + 0x90` to the orb on every one of the 5 orbs.
- **Using it does.** Every enemy that set the flag was **16.0 m or closer** (5.9 to 9.2 m eight times, 16.0 m once). **None of the 17 enemy-orb pairs from 16.4 to 35.6 m ever did.** One soldier (102000) flagged at 16.0 m and not at 22.5 m.
- **Close is not enough.** A soldier 4.5 to 10.9 m away (a 102020, five orbs) never flagged; a 102010 at 13.4 to 15.4 m but 8 m above the orb never flagged. So a second condition exists (line of sight, height, awake / idle / patrol state, or a path test); this run cannot say which.
- the author's count: one soldier on the first cast, then two or three each time, which fits the flags (and he saw the archer and the soldier circling underneath the orb earlier).
- The soldiers' perception values (`LogicComParam`: `sight_distance` 13, `Unk04` 20 (light range), `detect_distance` 1.0 to 1.9) do not equal the observed ~16 m, so the limit is another constant, a path length or a script test. Not identified.
- Only 3D distances at the moment the orb appeared were logged, so horizontal against 3D is not separated.
- **Review, 14 Nov (late): the "16 m" is weaker than it reads.** Only **two** soldiers ever set the flag. One (`...65f750`) set it on all five orbs but moved under 2 m each time; the tool counts the flag if it was *ever* 1 during the orb
  and never checked whether it was already 1 (the soldier was probably fighting the player), so it may not have reacted to the orb at all. The 16 m figure rests on the other soldier alone (`...7bf490`: no at 22.5 m, yes at 16.0 m,
  then walked 17 m). The log has no facing and no enemy-to-player distance, so a close soldier "never reacting" could simply be facing away, and the 16 m soldier may have noticed the player rather than the orb.
  Soldier perception (`LogicComParam` 102010): `sight_distance` 13, `Unk04` 20, `vision_cone_horizontal_angle` 1.05 rad, `vision_cone_vertical_angle` 0.785 rad, `detect_distance` 1.0.
- **Target hijack (hypothesis, untested).** Every flagged enemy switches `+0x90` to the orb at any distance. If its perception then checks the orb instead of the player, an enemy that cannot perceive the orb is blind to the player
  for 10 s (Yearn) / 5 s (skull). That would explain the community quirk of enemies standing still near a skull and ignoring the player. Test: tracker decoy-0f2.7 (cast, then teleport among far enemies in the same loaded map).
  **The static trace (section 2a) predicts it is NOT map-wide:** the next normal search drops the orb and can pick the player again. Within the search range the orb wins over the player.
  The quirk is explained by "no line of sight needed" plus no path. The teleport test is still worth running: it checks both predictions at once.

## 6. Open questions (all in the tracker)

1. The exact range and whether it is 3D, horizontal or path length.
2. Line of sight (walls), the height effect, and the community "stands still near a skull out of sight" quirk: reaction or pathfinding?
3. Why a close, level soldier sometimes never reacts (state: asleep, idle, patrol?).
4. Confirm live the static answer (section 2a): reaction = a decoy-enabled search running while the orb is within its range (20 / 50 m, 3D, no line of sight); the Ogre idles without searching.
4a. Target hijack: are enemies that took the orb but are out of its range blind to the player while it lasts? (Predicted: no, only until their next search.)
4b. Answered statically (section 2b) except one condition: what makes `f5621` ("AI active") true. Read it live.
5. The Alluring Skull on soldiers (never tested), the Hunting Dogs, and a sample of the other families the table predicts "YES".
6. Priority: does an enemy already hunting the player still turn to the orb? Two orbs at once? Orb nearer than the player?

## 7. Method and tools (to repeat any of it)

- **Game:** vanilla Scholar copy `<local path> Boss Arena ModEngine Test\Game` with `useModOverrideDirectory=0` in `modengine.ini` (see [the legacy DS2 Mod Engine](../modding-scripts/mod-engine-ds2.md#the-ini)) (original saved as `.bak-bossarena`); Steam in offline mode. The `DS2 Scholar PC Debug` copy has the Boss Arena data mod loose (it moves bonfires), so do not use it for this.
- **Character / items (no Cheat Engine):** `<local path> Boss Arena\tools\arenactl.py` run **without** `-I` (`give 31220000`, `give 60530000:20`, `give 50910000`, `stats 99`, `warp <bonfire id>`); `<local path> Scholar PC\tools\pcmem.py` (`bonfires light <ids>`, `near`, `player`).
- **Memory routes:** ChrAi from a character: `[[[chr + 0xE8] + 0x18] + 0x20]`; lock-on target: `[GMI + 0x20] -> +0x28 -> +0xB0 -> +0x18 -> +0xC0`; enemy position: `hm = [[chr + 0xE0] + 0x1B8]`, body = `[hm + 0x80 + 6 * 0x80]`, position at body + 0x1A0 (the player-style chain is wrong for enemies); orb position: DamageManager `[[BaseA] + 0x10]` records (0x128 bytes, id at +0x1C, PXBullet at +0x08, position at PXBullet + 0x20).
- **Finding live enemies** by EnemyParam id (`pcmem.live_chrs`) takes 60 to 90 s per scan and goes stale after every rest or reload: launch the tool, wait for "RECORDING", cast, do not rest in between. When launching with PowerShell `Start-Process`, **quote paths that contain spaces**.
- **Ghidra:** the MCP entry in `<local path> Network Test\.mcp.json` now points at `DS2SPC.gpr` and `C:\ghidra_12.0.4_PUBLIC` (the old Mac entry is kept as `ghidra-mac`).
  Only one process can open the project: if another session's `pyghidra-mcp` is still running it holds `DS2SPC.lock` and the MCP fails with "Connection closed".
  Fallback used on 14 Nov: copy `DS2SPC.gpr` + `DS2SPC.rep` elsewhere and run headless `AnalyzeHeadless ... -readOnly -postScript` on the copy (about 30 s a run; `script.cmd` in the
  Ghidra Project folder shows the command line).
- **Logger, second version:** `tools\decoy_watch_pc.py` (14 Nov, not yet run live; `--test` first). Enemies from the AiManager list (no scan, survives reloads), per sample the target,
  battle flag, "search ran" byte and cooldown, the enemy's own perception bits for the player, positions; log `decoy_watch_log.tsv`, `--trace` per-sample file, `--tp` teleport hotkeys
  (Ctrl+F5-F7 save, F5-F7 jump; writes).
- **Memory entrances:** the Forest (m10_10) event script starts [the three Giants' memory entrances](giant-memory-fireballs-pc.md#1-one-map-three-entrances) (groups 37000 / 37010 / 37020: objects 10101510 / 10101520 / 10101530, flags 105300 / 105301 / 105302, destinations 3700000 / 3701000 / 3702000 in m20_10). 105300 is Jeigh's (the Ashen Mist Heart, item 50910000, at the corpse by The Place Unbeknownst, bonfire 10675); Vammar is one of the other two.
  The large Giants stand in the west of m20_10 (x -90 to -195); the Forgotten Giants and the Giant Lord are in Jeigh's.

## 8. Files

`yearn_decoy_pc.tsv`, `yearn_decoy_enemies.xlsx`, `decoy_range_log.tsv`, `yearn_decoy_pc_v2.tsv` (this folder); `..\tools\decoy_range_pc.py`, `..\tools\decoy_watch_pc.py`, `..\tools\yearn_decoy_table_pc.py`; memory notes `ds2-yearn-decoy`; tracker `<local path> Yearn Decoy\.beads`.
