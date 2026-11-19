# Memory of the Giants: the fireballs and the rolling statue head (2026-11-10)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

Question: the giants' memory has constant fireball bombardment and a statue head that gets knocked off and rolls.
Can either be stopped, or never triggered?

**Answer: yes, both.** The answer was worked out from the data (event scripts and params), then built as a data
change and **tested live on 10 November 2026**: the fireballs no longer fire. The work done is listed in section 5.
The live-flag tricks in section 4 and the 5-minute limit in section 6 were not run live except where section 5 says so.

Sources: `ezstate\event_m20_10_00_00.esd` + `.edd` (developer event names), `EventParam`, `MapObjectInstanceParam`,
`EventLocation`, `SystemDamageParam` (all `_m20_10_00_00` where per-map), Forest `event_m10_10_00_00` groups
37000-37020. Dump tool: `tools\evdump_pc.py <map> [group ...]` (esd.py + edd.py names, follows sub-machine calls).
PS3 v1.00 has the same script for every group below (esd.py diff vanilla -> PC Scholar: only group 20000 changes,
adding summon generator 656 to the immunity list).

## 1. One map, three entrances

All three memories are map **m20_10**. The Forest's corpse events (m10_10 groups 37000 / 37010 / 37020, "giant's
memory warp: boss / tunnel / chandelier") need the Ashen Mist Heart (50910000), clear global flags 105300-105302,
set their own, and warp in:

| Forest group | dev name | sets | arrival point |
|---|---|---|---|
| 37000 | 巨人の記憶ワープ_ボス (corpse behind the King's Gate = Memory of Jeigh) | 105300 | 3700000 |
| 37010 | 巨人の記憶ワープ_坑道 (tunnel corpse) | 105301 | 3701000 |
| 37020 | 巨人の記憶ワープ_シャンデリア (chandelier corpse) | 105302 | 3702000 |

## 2. Three separate fireball systems

| groups | dev name | launchers (MapObjectInstance) | shot | runs when | off switch |
|---|---|---|---|---|---|
| 1000-1070 | にぎやかし火球発射_1-8 ("ambient fireballs") | 20103700-20103707, one each, state 70 every 8-10 s | 220100060: 907 fire + 454 fire burst (220100070, radius x2) | from load, looping forever | flag **105300**, read at load and at every loop |
| 2000 -> 2012 / 2013 | 前線火球_発射管理 / 前線火球発射_通常 / _ボス ("front-line fireballs") | normal: 20103608 then 20103667, ~2 s apart; boss: 20103652, 612, 608, 667 every 1.5 s; state 75 | 220100700: 1134 fire burst, radius x10.5, lasts 0.8 s | each round waits until the player is inside region 200000, or flag 210020080 (Giant Lord fight on: boss pattern) | **none**: 2000 restarts itself with no flag test |
| 3000 | 火球発射_石像頭ゴロゴロ用 | 20103680 | 220100400: 811 + 454 burst (220100410, radius x6.5) | part of the head event, section 3 | as section 3 |

- 105300 = "came in through the King's Gate corpse", so the ambient fireballs never fire in the Memory of Jeigh.
  They fire throughout the other two entrances.
- Group 20000 (ダメージの無効化) makes the Giant Lord (generator 800) and the two summon NPCs (656, 657) immune to
  220100000 (the head, section 3) and the Giant Lord also to 220100700. The player is not immune.
- 210020080 is set when the Giant Lord's HP bar appears and cleared when he dies (group 8000).

## 3. The rolling statue head (groups 3000 and 3010)

Group 3000 runs once per map load:

1. If the Giant Lord's kill flag **210000081** is set, the event ends at once: the head never moves again.
2. Otherwise it resets the head (OBJ 20101050) and three ballistas it smashes (20102352, 20102359, 20102313) to
   intact. So the head is put back and **rolls again on every visit** until the Giant Lord is dead.
3. It waits for the player to enter region **300000** (a 30 x 5 x 10 box about 24 m from the Giant Lord's
   fight-start box 800000 and about 38 m from the Jeigh arrival point).
4. Launcher 20103680 fires once. **5.6 s later the script itself** sends the head into [state 70](../modding-scripts/map-object-states.md#1-what-an-object-state-is) (falls and
   rolls), if the head is still intact. A timer drives it, not the fireball's hit.

The head carries SystemDamageParam **220100000: 9999 x10, hitbox x5, i.e. a kill**. Group 3010 then switches
navmesh 301000 once the head lies in state 20 (fallen).

## 4. How to stop or avoid each

**In normal play**
- Ambient fireballs: enter the memory from the King's Gate corpse; they never start. There is no other way.
- Front-line fireballs: no off flag. A round only starts while you stand in region 200000 (a long box, 40 x 20 x
  150, centred about 25 m past the head trigger), so they stop once you leave it, after the round in progress.
  During the Giant Lord fight they fire from anywhere.
- Head: kill the Giant Lord. After that it never rolls again. Before that, it rolls whenever you enter region
  300000.

**Live flag writes (CE)**
- 105300 = 1 while inside: each ambient launcher stops at its next loop (8-10 s). The 5-minute limit (section 6)
  stays off for this visit, because group 12000 only reads the flag at load. The next corpse warp rewrites the flag.
- 210000081 = 1 before the memory loads (a map-local flag: see [Event flags and choosing safe ids](../modding-scripts/event-flags-and-free-ids.md#2-which-blocks-are-in-memory)): the head stays put, **but the Giant Lord is gone too** (it is his kill
  flag; group 8000 ends at once when it is set). Good for exploring, not for the fight.
- Front line: no flag. Clearing 210020080 mid-fight should drop the boss pattern (nothing sets it again until the
  next fight), but this is untested.

**Permanent (data change):** [delete these rows](../modding-scripts/event-script-groups.md#b-disable-a-fromsoftware-group) from `EventParam_m20_10_00_00`. This is the route that was built and
tested (section 5).
- Ambient fireballs: 1000, 1010, 1020, 1030, 1040, 1050, 1060, 1070.
- Front line: 2000, 2012, 2013. 2012 and 2013 are started by 2000 (command 130003); their rows have Unk0B = 3
  where the other rows have 2, so they are probably start-on-request rows. Delete all three to be safe.
- Head: 3000, 3010. With 3000 gone, the head and the ballistas stay as placed; 3010 only changes navmesh after a
  fallen head.

To keep the fireballs on screen but harmless, zero `damage_0` in SystemDamageParam 220100060 / 070 (ambient),
220100700 (front line) and 220100400 / 410 (head shot). For the head, zero 220100000. Those rows are only used by
these launchers in m20_10 (the ids are this map's 2201xxxxx block; not checked against other maps' objects).

## 5. The work that was done, and the live test (10 November 2026)

1. **Traced the scripts** (sections 1-3): which event groups launch the fireballs and the head, what starts and stops
   each, and which damage rows they use.
2. **Checked what must stay.** The Giant Lord's own fight script (group 8000), his summon, and the bullet immunity
   (group 20000) were left alone, so the fight itself works as before.
3. **Deleted 14 rows from `EventParam_m20_10_00_00`:**
   - ambient fireballs: 1000, 1010, 1020, 1030, 1040, 1050, 1060, 1070 (8 rows);
   - front-line fireballs: 2000, 2012, 2013 (3 rows; 2012 and 2013 are started by 2000);
   - rolling head: 3000 and 3010 (2 rows: its fireball, the scripted fall 5.6 s later, the 9999 x10 hit, and the
     navmesh switch after it);
   - plus row 12000, the 5-minute limit for the Jeigh entrance (the same method; see section 6).
   Only that one param file changes; no script, model or damage row was edited.
4. **Offline checks.** The rebuilt file was compared with the unmodified one: exactly those rows are gone and the
   boss, summon and immunity rows are still there; it is the only file that differs.
5. **Live test on PC Scholar 1.0.3.0 (Steam, offline), entering the memory directly rather than through the Forest
   corpse warp.** The checks were:
   - on arrival and on the walk to the Giant Lord's fog: no fireballs flying or landing, and no statue head falling
     or rolling;
   - during the Giant Lord fight: no fireball volleys, and the fight otherwise unchanged (he wakes when you cross
     his region, and winning works as before);
   - dying once and coming back in: no fireballs again.

   Result: **pass**. The fireballs no longer fire, and the statue head no longer rolls.

Still untested: the live-flag writes in section 4, and whether the zeroed-damage option (fireballs on screen but
harmless) behaves as described.

## 6. Related: the memories' 5-minute limit

Groups 12000 / 12010 / 12020 (one per entrance flag 105300 / 105301 / 105302, read at load only) run global
timer 0. At 5 s they show mapevent 2011 "One cannot reside within memory for long". At 270 s they show 2012 "The
ashen mist has thinned...". At 290 s they set flag 210010001. At 300 s they show 2013 "The ashen mist fades..."
and warp you back to that corpse's point in the Forest (3700000 / 3701000 / 3702000). The Giant Lord's death
pauses the timer for 9 s. Deleting row 12000 (as in section 5) switches the limit off for that entrance.

## Related pages

- [Quieting and removing enemies near an arena](../modding-world/removing-enemies.md): the rest of the Giant Lord memory clean-up: the 14 corridor enemies between the fog and the Giant Lord, and what wakes him.
