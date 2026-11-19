# Enemies per location: vanilla patch 1.10 vs PC Scholar (2026-10-25)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author): count the enemies in each location in vanilla (patch 1.10) and PC Scholar. The DLC is known to be the
same bar NPC invasions, so leave it out. Community belief to test: Scholar has more enemies in the early game, but
the mid and late game are similar.

Tool: `..\tools\enemy_counts_pc.py` (read only; run with `uv run --no-project --python 3.12 --with openpyxl python -I -X utf8`).
Chart: `..\tools\enemy_counts_chart.py` (matplotlib). Outputs here: `enemy_counts_v110_vs_pc.txt` (full report),
`enemy_counts_v110_vs_pc.tsv` (per map), `enemy_counts_families_v110_vs_pc.tsv` (per map and enemy family),
`enemy_counts_v110_vs_pc.png`.

## Short answer

**It does not pan out in the data.** PC Scholar has more regular enemies in every part of the game, and the early game
gets the *smallest* relative increase, not the biggest.

| part of the game (my grouping) | vanilla 1.10 | PC Scholar | change | first playthrough only* |
|---|---|---|---|---|
| early (9 maps) | 420 | 493 | **+73 (+17 %)** | +20 % |
| mid (8 maps) | 491 | 595 | **+104 (+21 %)** | +27 % |
| late (7 maps) | 297 | 376 | **+79 (+27 %)** | +30 % |
| **base game** | **1,208** | **1,464** | **+256 (+21 %)** | **+25 %** (1,118 → 1,401) |

\*Leaves out the copies that only appear in NG+ (see Method). Counting the 55 → 66 bosses and mini-bosses as well
changes nothing: early +18 %, mid +21 %, late +26 %.

- **The pattern holds under other groupings** (report section F): a stricter route split gives early +14 %, mid +22 %,
  late +27 %; splitting the route into thirds of the vanilla enemy count gives +17 %, +22 %, +25 %. Along the route
  the running PC / vanilla ratio only creeps from 1.17 after the early maps to 1.21 at the end.
- **"Similar" is true for individual places, but they are not all late.** Flat or down: Old Akelarre 0, Doors of
  Pharros −1, Grave of Saints −2, Lost Bastille +3, Drangleic Castle −5, The Gutter −9 (+5 on a first playthrough),
  Dark Chasm 0. Against that, late and optional areas jump: Dragon Aerie / Dragon Shrine **+32 (+107 %)**, Memories of
  the Giants +17 (+46 %), Aldia's Keep +14 (+56 %), Brightstone Cove Tseldora +50 (+45 %), Shrine of Amana +24 (+26 %).
- **Where the early game does gain** is spread over most areas rather than concentrated: Forest of Fallen Giants +20,
  No-man's Wharf +19 (+39 %), Things Betwixt +6, Harvest Valley / Earthen Peak +14, Majula +4 (boars and a Mimic).
- **A guess at why it feels early-heavy** (not tested): the early gain is in main-route areas every player crosses,
  while the biggest late gains are in side areas, and the Drangleic Castle main path is flat. Five extra Pursuers (four
  in the Lost Bastille, one in Things Betwixt) also add to the early game's mini-boss pressure (not counted as regular
  enemies here). The Lost Bastille's extra Belfry Gargoyles are a scripted fly-by, not a fight (see Not counted).

## By location (regular enemies, generators)

Route order, tier as above. "First play" = without the NG+-only copies. "ids" = generator ids kept / added / removed
(churn behind the net change).

| location (map) | tier | vanilla | PC | change | % | first play | ids kept / +added / −removed | biggest changes (vanilla → PC) |
|---|---|---|---|---|---|---|---|---|
| Things Betwixt (m10_02) | early | 24 | 30 | +6 | +25 % | 20 → 26 | 23 / +7 / −1 | Basilisks 0 → 6 |
| Majula (m10_04) | early | 11 | 15 | +4 | +36 % | 4 → 8 | 11 / +4 / −0 | Undead Boars 0 → 3 (one giant), a Mimic; basement skeletons regrouped by NG+ cycle (8 → 8) |
| Old Akelarre (m10_29, map named "Majula") | early | 11 | 11 | 0 | 0 % | 11 → 11 | 11 / +0 / −0 | a Gyrm replaces a Rock-Thrower Grotesque |
| Forest of Fallen Giants (m10_10) | early | 107 | 127 | +20 | +19 % | 105 → 123 | 80 / +47 / −27 | Hollow Infantry 42 → 54, Skeletons 0 → 6, Royal Swordsmen 0 → 3 |
| Heide's Tower of Flame / Cathedral of Blue (m10_31) | early | 13 | 17 | +4 | +31 % | 10 → 14 | 11 / +6 / −2 | Heide Knights 0 → 6, Old Knights 13 → 11 |
| Lost Bastille / Sinners' Rise / Belfry Luna (m10_16) | early | 104 | 107 | +3 | +3 % | 93 → 98 | 89 / +18 / −15 | Rupturing Hollows 30 → 34; Stray Dogs 26 → 22, Jailers 11 → 7 |
| No-man's Wharf (m10_18) | early | 49 | 68 | +19 | +39 % | 47 → 66 | 46 / +22 / −3 | Hollow Infantry 6 → 12, Stray Dogs 2 → 7, Sailors 26 → 31 |
| [Harvest Valley / Earthen Peak (m10_17)](../mechanics/earthen-peak-aggro-pc.md#3-what-scholar-did-change) | early | 90 | 104 | +14 | +16 % | 81 → 98 | 70 / +34 / −20 | Undead Supplicants 0 → 8, Manikins 15 → 22; Steelworkers 11 → 6, Beetles 17 → 12 |
| Unseen Path to Heide (m10_30) | early | 11 | 14 | +3 | +27 % | 11 → 14 | 7 / +7 / −4 | Hollow Infantry 3 → 6; Old Knights 5 → 1 |
| Iron Keep / Belfry Sol (m10_19) | mid | 48 | 54 | +6 | +12 % | 40 → 48 | 43 / +11 / −5 | Alonne Knights 16 → 26; Alonne Captains 16 → 13 |
| Huntsman's Copse / Undead Purgatory (m10_23) | mid | 93 | 123 | +30 | +32 % | 88 → 119 | 79 / +44 / −14 | Hollow Prisoners 3 → 13, Skeletons 25 → 35, Captive Undead 10 → 17, Moths 5 → 11 |
| The Gutter / Black Gulch (m10_25) | mid | 94 | 85 | −9 | −10 % | 73 → 78 | 60 / +25 / −34 | Captive Undead 61 → 44, Corrosive Ants 0 → 4; fewer NG+ copies (21 → 7) |
| Shaded Woods (m10_32) | mid | 45 | 61 | +16 | +36 % | 44 → 60 | 41 / +20 / −4 | Forest Guardians and other humans 7 → 18, Hollow Falconers 0 → 5; Grotesques 18 → 12 |
| Doors of Pharros (m10_33) | mid | 32 | 31 | −1 | −3 % | 32 → 31 | 31 / +0 / −1 | a Mimic gone |
| Brightstone Cove Tseldora (m10_14) | mid | 112 | 162 | +50 | +45 % | 107 → 157 | 25 / +137 / −87 | Parasite Spiders 56 → 97, Hollow Falconers 0 → 7; most generators redone |
| Grave of Saints (m10_34) | mid | 42 | 40 | −2 | −5 % | 42 → 40 | 39 / +1 / −3 | Rupturing Hollows 6 → 3 |
| Aldia's Keep (m10_15) | mid | 25 | 39 | +14 | +56 % | 25 → 39 | 24 / +15 / −1 | Mimics 1 → 6, Dragon Acolytes 5 → 7 |
| [Shrine of Amana (m20_11)](shrine-of-amana-pc.md#1-enemies-how-many-of-each) | late | 94 | 118 | +24 | +26 % | 88 → 113 | 80 / +38 / −14 | Maidens 16 → 21, Clerics 12 → 16, Basilisks 0 → 4, Aberrations 48 → 52 |
| Dragon Aerie / Dragon Shrine (m10_27) | late | 30 | 62 | +32 | +107 % | 30 → 62 | 21 / +41 / −9 | Rupturing Hollows 4 → 25, Black Dragon Knights 3 → 20; Drakekeepers 10 → 4 |
| Memories of the Giants: Vammar / Orro / Jeigh (m20_10) | late | 37 | 54 | +17 | +46 % | 37 → 54 | 30 / +24 / −7 | Royal Soldiers 16 → 27, Ironclad Soldiers 0 → 7 |
| Drangleic Castle / King's Passage / Throne of Want (m20_21) | late | 64 | 59 | −5 | −8 % | 62 → 59 | 33 / +26 / −31 | Stone Soldiers 19 → 10, Alonne Captains 9 → 1; Old Knights 0 → 3 |
| Undead Crypt (m20_24) | late | 61 | 72 | +11 | +18 % | 57 → 72 | 44 / +28 / −17 | Insolent Knights 0 → 12 (event-gated); Syan Knights 9 → 4 |
| Dragon Memories (m20_26) | late | 0 | 0 | 0 | n/a | | | holds a single cat (an NPC) |
| Dark Chasm of Old (m40_03) | late | 11 | 11 | 0 | 0 % | 11 → 11 | 11 / +0 / −0 | unchanged |

Over all base maps the biggest family gains are Parasite Spiders 56 → 97, Rupturing Hollows 46 → 70, Hollow Infantry
51 → 72, Skeletons 38 → 58, Hollow Prisoners 38 → 54, Poison Moths 5 → 23, Hollow Falconers 4 → 16, Insolent Knights
0 → 12, Basilisks 12 → 23, Royal Soldiers 16 → 27, Alonne Knights 16 → 26, Black Dragon Knights 3 → 20. Biggest cuts:
Alonne Captains 25 → 14, Captive Undead 74 → 64, Stone Soldiers 19 → 10, Forest Grotesques 30 → 23, Drakekeepers 10 → 4,
Steelworkers 11 → 6.

## Not counted (shown separately in the report, section C)

- [Bosses and mini-bosses](../mechanics/bosses-pc.md#ai-ezstate-scripts) (boss models, Pursuers, Belfry Gargoyles, Ruin Sentinels, Dragonriders, Guardian Dragons,
  Flexile Sentries, the Executioner's Chariot): **55 → 66** (+7 early, 0 mid, +4 late). Pursuers +5 (four in the Lost
  Bastille, one in Things Betwixt), Belfry Gargoyles +2 (one new in Aldia's Keep, one in Drangleic Castle, both
  respawning up to 12 times with an item drop; the five of the Belfry Luna boss fight are unchanged), Dragonriders +2.
- The Lost Bastille Belfry Gargoyle fly-by: **1 → 5** (generators 8010-8014 in m10_16). `event_m10_16_00_00.esd` flies
  each one once from a perch at about (-56, 26, -38) to region 1605010 in the middle of the belfry arena, and stops
  for good once the boss flag 116000091 is set. Vanilla has 8010 only. PC sends two waves, each started by its own
  trigger region (1605000, a new spot; 1605001, vanilla's 1605000 spot) and spaced 0 / 1.2 / 1.8 s. Corrected
  2026-11-01: the first version counted these as mini-bosses (56 → 71, Belfry Gargoyles +6).
- NPCs, summons, invaders, the Forlorn, Aldia, human bodies, cats: **266 → 353** (+87): [the Forlorn](../mechanics/forlorn-pc.md) 17 in the base
  maps, 33 unequipped "human" bodies (EnemyParam 696000: 10 HP, no equipment, spawn state 12 / 14, probably corpses
  or scenery, 13 of them in the Undead Crypt), and a net 37 more named NPC invaders and summons.
- DLC maps (m50_35-38).

## Method

- **Baseline = vanilla 1.10**: the v1.00 disc + patch 1.10 (`DS2_GAME=v110`). Its 2,094 generators are identical
  (enemy, position, spawn limits, gating) to the PS3 Scholar disc's, so PS3 Scholar is the same baseline. Against
  v1.00 only [Aldia's 8 generators](1.08-to-1.10.md#3a-the-base-game-patch-archive-patchdata-109-110-independent-of-the-read-order) and 10 Drangleic Castle rows differ.
- **PC** = Steam 1.0.3.0 (2,458 generators), read as in [placements and balance: PS3 Scholar vs PC Scholar](placements-balance-pc.md).
- **Counting unit**: one `GeneratorParam` row = one spawn point = one enemy. The community sheet's spawn counts equal
  these rows for 584 of 644 enemies, which shows both count the same thing; I have not confirmed in game that one row
  never spawns more than one enemy.
- **Kind** by EnemyParam id / model (`classify()` in the script): boss and mini-boss models; characters (ids from
  692000 up, Tark, Agdayne, Vengarl's summons) as NPC; hostile exceptions kept as enemies (Bell Keepers, Black Dragon
  Knights, the Shaded Woods humans, Insolent Knights, the Amana Pyromancers). Borderline: the Gutter Denizen
  (887000, counted as an NPC invader) and the 696000 bodies (excluded). The fly-by gargoyles are their own kind
  ("scripted", `SCRIPTED` in the script), so they count as neither enemies nor bosses.
- **[Gating](../modding-world/ng-plus-differences.md#ng-content-is-extra-generator-rows-data)** by `AppearanceEventID`: 0 and the small flags 10 / 30 / 50 = always; 1 = NG only (replaced by an NG+
  copy at the same spot: the Torturers and Lindelt Clerics pair up this way); 2-9 = NG+ cycles (the Majula basement
  skeleton waves are 2 to 8, matching the sheet's NG+ tags); 100 and up = an event flag. "First playthrough" drops
  the 2-9 copies (90 in vanilla, 63 on PC). "Core" (+25 % overall, in the report) also drops the event-gated ones
  (93 vanilla, 118 PC; they include the Insolent Knights and the statue-like Undead Travellers).
- **Tier grouping is mine**, from the usual route: early = Things Betwixt, Majula, Old Akelarre, the Forest, Heide's,
  Lost Bastille, Wharf, Harvest Valley / Earthen Peak, Unseen Path; mid = Iron Keep, Copse, Gutter, Shaded Woods, Doors
  of Pharros, Tseldora, Grave of Saints, Aldia's Keep; late = Amana, Dragon Aerie, Memories, Drangleic Castle,
  Crypt, Dark Chasm. Contested: Harvest Valley / Earthen Peak, Amana, Aldia's, Tseldora, Grave of Saints, the Aerie.
  Compound maps (e.g. Lost Bastille with Sinners' Rise and Belfry Luna) sit in one tier.

## Not checked

- Nothing was verified in game (no in-game enemy counts).
- The meaning of `AppearanceEventID` 10 / 30 / 50 (Things Betwixt, Shaded Woods, the Gutter) and of the large event
  ids; treated as always present (event-gated ones are shown separately) (see [GeneratorParam fields](../modding-world/generatorparam-fields.md#appearanceeventid-and-the-condition-bytes)).
- NG+ cycles beyond the 2-9 gating, respawn limits (12 times against once) and enemy strength: counts only.
- Whether the unnamed human ids 838000 / 838100 / 838300 (Shaded Woods) behave as ordinary enemies; counted as such.
- Whether vanilla's single fly-by gargoyle plays at all: its generator's AppearanceEventID is 1605000, which in this map
  is a trigger region's id (PC gates it by the flag 116000035 instead). Other maps were not searched for scripted
  set pieces like the fly-by.
