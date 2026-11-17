# Avelyn and crossbow live test plan (PS3 v1.00 on RPCS3, planned 2026-11-01)

!!! warning "Status: Plan, not run"
    A test or idea written down. Nothing here has been tried. See [status labels](../status-labels.md).

Follow-up to the static trace in `mundane-sl50-v100.md` ("Bolt damage in v1.00"). The trace left these points unproven,
because the code that combines crossbow AR and bolt damage was not found and the reload speed fields are unexplained:

1. A bolt's damage is the crossbow's AR plus the bolt's hidden flat value (Wood 20, Heavy 50, elemental 110 of its
   element), not the bolt AR the menu shows (Wood 70, Heavy 100, Lightning 70 + 160).
2. Each of the Avelyn's three bolts carries the crossbow's full AR. The alternatives are the AR split over the three
   bolts, or the Avelyn using the weaker x0.686 rows.
3. Mundane's bonus counts in full on crossbows in v1.00.
4. Powerstance shots use the x0.686 rows.
5. The real time from one shot to the next. The Avelyn's reload animation is 2.5 s, against 1.83 s for other crossbows.

Side questions the same session can settle cheaply:

- What physical type bolts count as. The Ogre takes x1.35 from thrust and x0.7 from slash and strike.
- Whether the distance cut in the bolts' data (-70 % between 40 m and 70 m) works on PS3.
- Whether an Avelyn shot uses one bolt or three.

Formula under test (H1, from Radai's guide). Per bolt:
0.98 x type absorption x (0.9 x (crossbow AR + physical flat) - defence / 6) + 0.98 x 0.9 x elemental flat x (100 - elemental defence) / 100.
Predictions come from `..\tools\avelyn_predict.py`.

## What's needed

- **From the author:**
  - RPCS3 running BLUS41045 with the v1.00 Warrior loaded at a bonfire. The character was SL12 at the last read on
    2026-10-17, with all bonfires unlocked.
  - At the controller for the in-game actions: levelling at the Herald, equipping from the menu, shooting.
  - A yes to changing the save. It's backed up first; say whether to restore it afterwards.
- **From me, before the session (no game needed, about an hour):**
  - **`vxbow.py log`, read-only.** It finds the player and every enemy within 80 m through the CharacterManager
    ([App+0x0c], enemy vtable 0x01c28ea8). It polls each enemy's HP (+0xfc, assumed to match the player's layout) and
    max HP (+0x104), both positions (+0x70), and the player's arrow-slot counts (equipment records 12-15, u16 count
    at +0xc). It prints one time-stamped line per HP drop and per ammo change, with:
    - the distance to the target;
    - the interval since the last shot;
    - the equipped weapon and bolt ids;
    - a "capped" mark when a hit killed (the drop then equals the remaining HP, so it can't be used).
  - **`vgive.py givew <weapon id> <upgrade> <infusion>` and `givea <arrow item id> <count>`.** These extend the
    goods give that worked live for the Torch (2026-10-17) to the weapon and ammunition list classes, with the known
    layouts (`port\equipment-inventory.md`):
    - weapon node tail: f32 durability, 0xff, upgrade, infusion;
    - weapon mirror entry: f32 durability at +8, upgrade at +0xc, infusion at +0xd;
    - ammunition mirror entry: u16 count at +8.

    Mundane is infusion 9.
  - **`vxbow.py tank`, optional and the only enemy write.** It sets one enemy's HP and max HP to 3000, so
    three-bolt shots don't kill it.

## Session steps

1. **Back up the save.** Copy `C:\Games\PS3 Emulator\rpcs3-v0.0.39-18764-aeaa62a2_win64_msvc\dev_hdd0\home\00000001\savedata\NPUB31358-GAME_000`
   to `..\saves\<date>_before_avelyn_test`.
2. **Check the reads.** Run `..\tools\vmem.py checks`; every row must read OK.
3. **Dry run in Things Betwixt.** Start the logger. Undead Travelers must read 160 or 180 HP (EnemyParam
   115006 / 115015, NG), which proves the enemy HP offset. The ammo count must drop when one bolt is fired.
4. **Souls and levels.**
   - Check the souls offset first: pparam [chr+0x378] +0xe8 must change when souls are picked up.
   - Then write souls and level at the Emerald Herald to STR 25 / DEX 15, the Avelyn's requirements one-handed. The
     lowest stat stays 5, so the Mundane bonus is 80.
   - No rings and no buffs, and check NG (journey 1).
5. **Give the items:**
   - 2 Light Crossbows (4600000);
   - Heavy Crossbow (4610000);
   - Avelyn +0, Avelyn +10, and Mundane Avelyn +10 (4660000, upgrade 10, infusion 9);
   - Wood, Heavy and Lightning Bolts x99 (60910000 / 60920000 / 60940000).

   Before anything saves, check each one in the inventory menu: name, +level, Mundane icon. Note the AR the menu shows
   for each crossbow and bolt.
6. **Damage tests (table below).** Equip from the game's own menu, never by slot write. Shoot from 5-15 m (the logger
   prints the distance). Hit the torso of an enemy that is standing or walking, not attacking, so no counter-hit bonus
   applies. Take three shots per test and ignore any "capped" hit.
7. **Reload tests (table below).** Mash R1 for 10 shots per test. The logger's intervals between ammo drops give the
   cycle. RPCS3's frame rate must hold steady while timing (note the FPS overlay), since the logger uses the host
   clock.
8. **SL120 version (optional).** Level every stat to 18, keeping STR 25 (SL116, lowest stat 18, bonus 158), then
   repeat with the Mundane Avelyn +10.
9. **Finish.**
   - Stop the logger and save its log in `..\findings\avelyn-live-test\`.
   - Update `mundane-sl50-v100.md` and the 1.01-to-1.03 notes if needed.
   - Restore the save if the author wants the character back as it was.

### Damage tests (predicted HP drop; target Undead Traveler unless named: defence 248, lightning defence 30, all type absorption 1.0)

| Test | Setup | H1: expected | What the alternatives would give |
|---|---|---|---|
| D1 | Light Crossbow +0, Wood Bolt | **61** per bolt | 105 if the menu value (70) were used |
| D2 | Light Crossbow +0, Heavy Bolt | **87** | 132 (menu 100) |
| D3 | Light Crossbow +0, Lightning Bolt | **111** | 204 (menu 70 + 160) |
| D4 | Heavy Crossbow +0, Heavy Bolt | **101** | 145 (menu); with D2 this gives the AR slope (0.882 per AR point) |
| D5 | Avelyn +0, Heavy Bolts | **79** per bolt, **236** per shot | split AR 86 per shot, x0.686 rows 165, menu 368 |
| D6 | Avelyn +10, Heavy Bolts | **154** per bolt, **461** per shot | split 161, x0.686 rows 322, menu 593 |
| D7 | Mundane Avelyn +10 at lowest stat 5, Heavy Bolts | **149** per bolt, **447** per shot | 236 if Mundane's bonus didn't apply to crossbows |
| D8 | 2 Light Crossbows, powerstance shot, Heavy Bolt | **61** per bolt | 87 if powerstance used the normal rows |
| D9 | Light Crossbow +0, Heavy Bolt, **Ogre** (defence 310) | **104** if bolts are thrust | 77 if untyped, 54 if slash or strike |
| D10 | as D2 but from about 55 m | **57** (x0.65 at the cut's midpoint) | 87 if there's no distance cut |
| D11 | SL116, Mundane Avelyn +10 at lowest stat 18, Heavy Bolts | **218** per bolt, **654** per shot | x0.686 rows 458, split 225, no Mundane 236 |
| D11b | as D11 with Lightning Bolts | **725** per shot | x0.686 rows 508 |

Notes:

- **Rounding.** The game may round or floor each bolt, so allow about ±1 per bolt. Every alternative differs from H1
  by 25 % or more.
- **Avelyn shots and low HP.** An Undead Traveler (160 HP) dies to most Avelyn shots. If the three bolts land on
  different frames, the first one or two drops are still clean. If they land together, use `vxbow.py tank`. Failing
  that, shoot the Ogre (1,300 HP) once D9 has given its type multiplier. A Things Betwixt Ogre hits hard, so keep
  your distance or use CT entry 8 (Player cannot die).
- **What D7 proves.** At lowest stat 5, Mundane +10 (AR 165) is almost the same as plain +10 (AR 170), so D7 only
  shows whether the bonus applies at all. D11 is the real check of its size.

### Reload tests (time between shots)

| Test | Setup | If the animations play at normal speed |
|---|---|---|
| R1 | Light Crossbow, one-handed | about 2.23 s (shot 12 frames + reload 55 frames at 30 fps) |
| R2 | Light Crossbow, two-handed | the same, unless two-handing uses other animations |
| R3 | Heavy Crossbow | about 2.23 s |
| R4 | Avelyn, one-handed | about 2.9 s (12 + 75 frames) |
| R5 | Avelyn, two-handed | about 2.9 s |

Notes:

- **What a match means.** If R1 and R4 match, the startup / recovery speeds (Avelyn 0.9 / 0.7, Light Crossbow
  0.9 / 0.3) don't touch the reload. Any other result tells us what they do.
- **Ammo per shot.** Each R4 interval also shows how many bolts one Avelyn shot uses.
- **DPS update.** The measured cycle replaces the rough 2.9 s in the Avelyn DPS figure (about 215 at defence 300).

## Risks

- **Untested writes.** The weapon and ammunition give paths have never run on retail; only the goods path has (the
  Torch). A wrong node could crash the game or be written into the save at the next autosave. That's why the backup
  comes first and each item is checked in the menu before resting or quitting.
- **The no-write route is slow.** In-game, the Light Crossbow and Wood Bolts are pickups in the Forest of Fallen
  Giants, and the Heavy Crossbow +3 is in Harvest Valley. The Avelyn is in the Undead Crypt. A Mundane +10 needs
  Chloanne's Old Mundane Stone, titanite, and a blacksmith who infuses.
- **Assumed offsets.** Enemy HP at +0xfc is assumed (step 3 checks it). Souls at pparam +0xe8 is probable (step 4
  checks it).
- **AI freeze is untested on retail** (CT entry 2), so the plan doesn't rely on it.
- **Session length.** About 1.5-2 hours with the author at the controller, plus my hour of preparation.
