# The Forlorn: when they invade (PC Scholar 1.0.3.0)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-11-05. The author asked for the spawn conditions, the number of versions and locations, the minimum and maximum
number of invasions in a playthrough, and any cooldown. Tool `tools\forlorn_pc.py` (table `forlorn_pc.tsv`, one row
per invasion area). Data and developer names only; **nothing here is live-tested.**

## 1. Answer

| Question | Answer |
|---|---|
| Conditions | On each map load: not killed in this area yet, then a **15 % roll** (Forest **40 %**, and only once you have picked up the **King's Ring**; Huntsman's Copse **no roll**). A hit picks one of the area's trigger spots at random; walk into it, wait a random 0-5 s (Lost Bastille 0-10, Copse 0-2) and the Forlorn spawns. Never while you are a guest in someone else's world |
| Versions | 2 loadouts (greatsword, scythe), 19 EnemyParam rows: 15 invasion spawns (8 greatsword, 7 scythe), 4 non-invading Forlorn in Aldia's Keep, 1 placed-but-never-spawned in No-man's Wharf, 1 unused row |
| Locations | **15 areas**: 12 base game + 3 DLC (table below) |
| Min per playthrough | **0** in strict terms. The Copse spawn is the only one without a roll, and it still needs you to walk into its trigger box |
| Max per playthrough | **No cap on invasions**: each area re-rolls on every load until you kill that area's Forlorn. **15 kills** that count (one per area), more with Bonfire Ascetics, which reset the kill flag |
| Cooldown | None. One roll per map load, at most one invasion per area per load |

## 2. The 15 invasion areas

Bonfire = the bonfire the generator belongs to, which is also the bonfire whose **Ascetic resets that area's kill
flag** (EventFlagListParam row in the Ascetic's reset range).

| Map | Area (bonfire) | Weapon | Chance | Trigger spots | Wait (s) | HP (NG) |
|---|---|---|---|---|---|---|
| m10_10 | Forest of Fallen Giants (The Crestfallen's Retreat) | greatsword | 40 %, after the King's Ring | 4 | 0-5 | 4,450 |
| m10_14 | Brightstone Cove Tseldora (Royal Army Campsite) | greatsword | 15 % | 4 | 0-5 | 2,800 |
| m10_16 | Lost Bastille (Exile Holding Cells) | greatsword | 15 % | 3 | 0-10 | 2,040 |
| m10_17 | Harvest Valley and Earthen Peak (Poison Pool; spots 1-3 near The Mines, 4-6 in Earthen Peak) | greatsword | 15 % | 6 | 0-5 | 1,890 |
| m10_19 | Iron Keep (Threshold Bridge) | greatsword | 15 % | 2 | 0-5 | 2,590 |
| m10_23 | Huntsman's Copse (Undead Refuge) | greatsword | **fixed** | 1 | 0-2 | 2,020 |
| m10_25 | The Gutter map (Upper Gutter), but its one spot is at the **Black Gulch** mouth | greatsword | 15 % | 1 | 0-5 | 2,830 |
| m10_27 | Dragon Aerie (Dragon Aerie) | scythe | 15 % | 3 | 0-5 | 3,910 |
| m10_32 | Shaded Woods (Ruined Fork Road) | greatsword | 15 % | 2 | 0-5 | 2,590 |
| m20_11 | Shrine of Amana (Tower of Prayer) | scythe | 15 % | 3 | 0-5 | 3,460 |
| m20_21 | Drangleic Castle (King's Gate) | scythe | 15 % | 3 | 0-5 | 3,300 |
| m20_24 | Undead Crypt (Undead Crypt Entrance) | scythe | 15 % | 2 | 0-5 | 3,480 |
| m50_35 | Shulva (Sanctum Walk) | scythe | 15 % | 5 | 0-5 | 4,630 |
| m50_36 | Brume Tower (Throne Floor) | scythe | 15 % | 3 | 0-5 | 4,630 |
| m50_37 | Frozen Eleum Loyce (Outer Wall) | scythe | 15 % | 3 | 0-5 | 4,630 |

All drop 16,000 souls and item lot 84910000. The area names come from the generator's bonfire link; where the
trigger spots actually sit is section 7.

## 3. How the events work

Each area has two developer events (names from the `.edd`): **【DC】放浪者A_ランダム抽選と生成** ("Forlorn A: random
draw and spawn"; Forest: "..._グローバルフラグ版", global-flag version; Copse: "放浪者A_固定出現", fixed appearance) and
**【DC】放浪者A_撃破判定** ("kill check"). Shaded Woods calls itself 放浪者B; the logic is the same.

**Draw** (runs at every map load):
1. Wait until in game. Guest (1:130602) -> stop.
2. Kill flag on (block 00, saved) -> stop for good.
3. Draw-done flag on -> skip to the spawn step. It sits in the map's **[02 flag block](../modding-scripts/event-flags-and-free-ids.md#1-a-flag-id-is-a-block-and-a-bit), which resets at every load**
   ([live-proven for block 53702 in the Boss Arena work](../modding-world/removing-enemies.md#removing-bosses-and-allies)), so in practice every load draws again.
4. [Roll 0-99 (1:1)](../modding-scripts/randomness-in-scripts.md#2-the-forlorns-draw-the-games-own-recipe). Hit if **roll <= chance** (1:140531 comparison 5; scripts that split 0-99 into bands always end
   on "<= 99", so 5 = "<="; the arg is 14, Forest 39 -> 15 % / 40 %) **or flag 109990 is on**. The edd calls that
   flag a debug switch: "【【DEBUG】】フラグ109990 ONで確実にアタリ可" ("DEBUG flag 109990 ON makes a sure hit"). No script,
   param or the exe sets it.
5. Hit: set the draw-done flag, pick a trigger spot 1..n at random into an area variable. Miss: set the draw-done
   flag, area variable 0.

**Spawn**: one preset per trigger spot, run in turn; only the one whose number matches the area variable waits. It
waits for the player to enter that EventLocation box (1:140132), then a random min..max wait (1:140511, comparison
3 = ">="), then moves the generator's start to a random spawn point (1:131705) and spawns the dark spirit (1:131741).
After all presets the event sets a "finished" flag (also block 02) and ends. So: one chance per load, at most one
invasion per load, no timer between attempts.

The state that waits is named "敵対・味方数の判定とランダムウェイト" ("check hostile / friendly count, then random
wait"), but its code checks only the wait; there is no phantom-count check in the shipped script.

**Forest extras**: gate flag **100804**, set by Undead Crypt event 8030 王の指輪を取得する ("obtain the King's Ring")
when the ring is picked up. Chance arg 39. Its 4th trigger spot uses spawn points 81000300-81000301 (the others use
+1..+99), harmless.

**Huntsman's Copse (fixed)**: no roll and no kill-flag check in the spawn event. Enter EventLocation 6000000 (a
32 x 22 m box at about (-132, 34, 15)), wait 0-2 s, spawn, set 123020043. What stops it after a kill is the
generator: `SpawnLimit` 1 and `SpawnLimitGameClear` 1 (the other Forlorn generators have [0 = default](../modding-world/generatorparam-fields.md#spawnlimit-and-spawnlimitgameclear)).

**Human or hollow**: neither the Forlorn events nor the other NPC invasion events (e.g. Merciless Roenna's
"NPC黒ファントム出現：オンライン：無条件", "unconditional") check the player's form. If the game blocks NPC invasions while
hollow, that is in the engine, not the scripts. Not checked.

**Online / offline**: no 1:140671 offline check, so both.

## 4. Kill counter and Straid

The kill check sets the area's kill flag and steps a **global** counter (block 10): 1st kill 102750, 2nd 102752,
3rd 102753, 4th 102751, 5th and later 102754 (greatsword Forlorn) or 102755 (scythe Forlorn). The edd names them
head / hands / legs / body / weapon flags. They are the `enable_flag`s of **Straid of Olaphis**'s [shop](../modding-menus/vendor-shops.md#the-two-params)
(ShopLineupParam 768xxxxx):

| Kill | Flag | Straid sells |
|---|---|---|
| 1st | 102750 | Hood of the Forlorn |
| 2nd | 102752 | Gauntlets of the Forlorn |
| 3rd | 102753 | Leggings of the Forlorn |
| 4th | 102751 | Armor of the Forlorn |
| 5th + (greatsword one) | 102754 | Greatsword of the Forlorn |
| 5th + (scythe one) | 102755 | Scythe of the Forlorn |

So the full set takes 4 kills of any kind, and both weapons need at least 6, with one of each kind after the 4th.
The 4 Aldia's Keep Forlorn don't count (no kill check). Those flags are listed in EventFlagListParam rows
10002750-10002755, which no Ascetic range covers; whether they survive NG+ is not checked.

## 5. The other Forlorn

- **Aldia's Keep, 4 generators** (7000 / 7001 = 695004 scythe, 7002 / 7003 = 694009 greatsword; 6,000 souls): not
  invaders. Events 【DC】灯火台やランタンと連動する敵_1-4 ("enemies linked to the braziers / lanterns"): lighting
  brazier 10150700-703 sets 115000070-073 (the generators' AppearanceEventID) and the Forlorn appears; killing it
  lights the matching lantern (10150710-713). Ordinary enemies, no kill counter.
- **No-man's Wharf, generator 938** (694001, HP 1,680): [set up as an invader](npc-invaders-pc.md#2-how-an-npc-spirit-is-made) (Unk84 = 2, activation flag 118020033),
  but no event spawns it or uses that flag. Never appears.
- **EnemyParam 695000** (scythe, HP 2,450): no generator.

## 6. Open

- Whether resting at a bonfire (no loading screen) counts as a load for the re-roll (a live test of what counts as a load is on [Warping by script](../modding-world/warping-by-script.md#b38-what-counts-as-a-load)). Deaths, warps, map crossings
  and quit-outs reload the map.
- Engine-side gates on 1:131741 (hollow form, other phantoms present).
- Whether the Ascetic also resets the Copse generator's [kill count](../modding-world/respawning-bosses.md#the-kill-counter) (needed for the fixed one to come back).
- Whether a trigger box counts only within its height (section 7).

## 7. Where the trigger spots are (added 2026-11-06)

The author asked whether the spots are contiguous or whether you could weave between them. Tool
`tools\forlorn_spots_pc.py`: pictures `forlorn_spots\<map>_<n>.png` (one per spot, 200 m across) and
`forlorn_spots\sheet_<map>.png` (all of a map's spots, 150 m each), table `forlorn_spots.tsv`. Visual map mesh only
(no collision, doors, gates or invisible walls); **not checked in game**.

**Frame.** EventLocation positions are in [the map's own frame](../modding-world/warping-by-script.md#the-coordinate-frame): MSB world minus the map origin (the position of the
baked map pieces: Forest (208, 10, -133), Tseldora (-631, 94, -88), Copse (-166, -11, -10) ...; 0 for the three DLC
maps). With that shift every spot lands on a floor; without it most float in the air.

**Shapes.** 45 spots: 44 boxes (RegionType 3: Scale = full width x height x depth, rotated about y; base at
Position y, assumed) and one cylinder (type 2, Tseldora spot 1: radius 20 m, 3 m high). Many are small: 5 x 5 m
(9, plus one 6 x 5), 3-4 m across (3), thin 3 m strips (3); the rest 10-30 m; the biggest are Shulva 5 (28 x 35 m),
Forest 1 (33 x 15), Copse (32 x 22) and the Tseldora cylinder. Heights are 1-2 m for 36 of the 45 (up to 9 m), set
just above the floor, which suggests the height counts (a floor 2 m above or below the box would not trigger it).
Not confirmed from code.

**Not contiguous.** The spots are separate boxes 21-241 m apart (centre to centre); no two touch. You can't weave
between them in the useful sense, for two reasons:
1. Only **one spot is live per load**: a hit picks one at random, and the rest do nothing that load. Nothing in the
   world shows which one it picked.
2. They are placed on **chokepoints**: most fill a doorway, a corridor, a bridge or the full width of a path, so you
   can't get past that point without stepping in. A few sit in open ground or in side rooms.

So you avoid the Forlorn by not going through the live spot's part of the map, not by weaving round it. To be sure
of no invasion on a load you'd have to avoid every spot in that area.

Per area (my reading of the pictures; "fills" = the box spans the way through):

| Area | Spots | Where |
|---|---|---|
| Forest (after the King's Ring) | 4 | 1: most of a round plaza 54 m from Soldiers' Rest (its rim may be outside the box); 2-4: 5 x 5 m doorways in the Cardinal Tower / Place Unbeknownst buildings (3 leads outside, 4 into a side room) |
| Tseldora | 4 | 1: cylinder over the plaza beside Lower Brightstone Cove bonfire; 2-4: each fills the one path running south-east from there (2, 3) and the wooden platform at its end (4) |
| Lost Bastille | 3 | all 5 x 5 m in corridors / doorways 10-35 m from Straid's Cell and Exile Holding Cells: fill |
| Harvest Valley / Earthen Peak | 6 | 1: 30 m along the path south of The Mines (fills); 2, 3: a room entrance and a narrow passage near The Mines (fill); 4-6: Earthen Peak, a corridor and two spots by Lower / Central Earthen Peak (6 is a 3 m strip; open) |
| Iron Keep | 2 | 1: end of a walkway (fills); 2: passage 13 m from Ironhearth Hall (fills) |
| Huntsman's Copse | 1 | 32 x 22 m on a raised mound at the side of the valley between Bridge Approach and Undead Purgatory; if the 1 m height counts, the valley floor below misses it |
| The Gutter map | 1 | Black Gulch, at the cave mouth 17 m from Black Gulch Mouth bonfire, on the path in |
| Dragon Aerie | 3 | 1: island beside a bridge end (open); 2: across a narrow bridge (fills); 3: 3 x 3 m corner near Shrine Entrance (open) |
| Shaded Woods | 2 | 1: across the path west of Ruined Fork Road into the foggy woods (fills: the clearest chokepoint of all); 2: end of a walkway 57 m from Shaded Ruins (open) |
| Shrine of Amana | 3 | 1: narrow shore path 53 m from Tower of Prayer (fills); 2, 3: open lake, easy to walk round |
| Drangleic Castle | 3 | 1: castle end of the entrance bridge (fills); 2: 4 x 4 m doorway near Forgotten Chamber; 3: 4 x 4 m passage 8 m from King's Gate (fill) |
| Undead Crypt | 2 | 1: corridor between two rooms (fills); 2: a room 47 m from Undead Ditch (fills its floor) |
| Shulva | 5 | 1: long thin strip near Tower of Prayer; 2, 3: corridor and bridge (fill); 4: open courtyard (walk round); 5: covers the **Sanctum Interior bonfire** building (15 m), so resting there means standing in it |
| Brume Tower | 3 | all on walkways (Throne Floor, Foyer, Iron Hallway Entrance side): fill |
| Frozen Eleum Loyce | 3 | doorway, corridor junction, narrow cliff path: fill |

The tool's own walk-round test (connectivity of the floor round each spot) is in the tsv as `test`; it is rough
(a 2-cell sliver next to a box counts as a "side"), so the table above is read from the pictures.

## Related pages

- [Event script groups](../modding-scripts/event-script-groups.md): how a Forlorn draw is switched off in a mod: deleting the EventParam row of its draw-and-spawn group (Earthen Peak's group 4002000).
