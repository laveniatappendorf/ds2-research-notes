# Item placements and item lots: vanilla v1.00 vs PS3 Scholar (2026-10-23)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Tool: `..\tools\item_compare.py` (run `uv run --no-project --python 3.12 python -I -X utf8 tools\item_compare.py`
from this folder). Every placement of both builds: `item_placements.tsv` (924 rows: map, entity, part, model,
position, lot, contents in each build).

**Short answer: Scholar on PS3 places every base-game item exactly where vanilla does.** All 652 world pickups on the
24 base maps have the same position, the same container object, the same lot id and the same contents. What changed
is the DLC (new maps, bosses and drops), the online result rewards (more generous), three enemy drop chances, one
filled-in drop list (the Forest of Fallen Giants Salamander) and a set of duplicate boss-reward lots.

## 1. How retail places an item

Decoded on Things Betwixt (m10_02), then used for every map:

1. The map's MSB has an object part (type 1) whose word **+0x9c** is its entity id.
2. That id is a row of `param\MapObjectInstanceParam_<map>.param` (40-byte rows). The row's **+0x24** is the item
   lot id (0 = no item). Chests also repeat the lot at +0x00.
3. The lot is a row of `ItemLotParam2_Other` (regulation, 124-byte rows): +0x2c ten item ids (10 = empty slot),
   +0x04 ten counts, +0x0e ten upgrade levels, +0x18 ten infusions, +0x22 ten unknown bytes, +0x54 ten chances.

Containers: `o00_0240` = the glowing pickup (369 of vanilla's 652), `o00_0210` (111), `o00_0256` (70),
`o00_0200` = chest (47), `o00_0247` (16), `o00_0257` (11), plus a few one-offs (`o02_0001` = the Bone of Order in
Things Betwixt, lot 10027000). Every instance row with a lot has a part in both builds (none orphaned).

## 2. Base maps: nothing moved

| | vanilla | Scholar |
|---|---|---|
| placed lots, base maps | 652 on 22 maps | 652 on 22 maps |
| same position, model, lot id and contents | | **652** |
| moved / different container / different lot / different contents | | 0 / 0 / 0 / 0 |
| added / removed | | 0 / 0 |

Per map (both builds): m10_02 11, m10_04 24, m10_10 63, m10_14 49, m10_15 23, m10_16 66, m10_17 62, m10_18 26,
m10_19 34, m10_23 28, m10_25 50, m10_27 28, m10_29 4, m10_30 4, m10_31 10, m10_32 37, m10_33 13, m10_34 13,
m20_10 23, m20_11 34, m20_21 26, m20_24 24. The Dragon Memories (m20_26) and the Dark Chasm (m40_03) have none.
DLC maps (Scholar only; names from MapName.fmg): Shulva, Sanctum City m50_35 85, Brume Tower m50_36 106, Frozen
Eleum Loyce m50_37 81, Memory of the King m50_38 none. (Eleum Loyce's map names include two empty '（No Text）'
slots, 50370002 and 50370003, between Grand Cathedral and Frigid Outskirts: worth a look in the cut-content pass.)

Lots that no map object places (boss, NPC and event rewards, and variants such as 10026001 = Rusted Coin x5 beside
the placed 10026000 Rusted Coin): vanilla 317, Scholar 434; **312 identical, 5 changed, 117 new, none removed.**

## 3. What changed

### Online result rewards (`ResultEventParam` -> lots 10000-10280)

Only `ResultEventParam` (24-byte rows: flags, IngameSystem message, lot, lot + 1000) points at these lots. Scholar
makes online play pay more:

| result (message) | row | vanilla | Scholar |
|---|---|---|---|
| "Dark spirit %s has been vanquished." | 1200, 2400 | nothing | Human Effigy (lots 10250, 10260) |
| "Arbiter spirit %s has been vanquished." | 1100 | nothing | Human Effigy (10270) |
| "Gray spirit %s has been vanquished." | 1300 | nothing | Titanite Chunk 90 % / Titanite Slab 5 % / Twinkling Titanite 3 % / Petrified Dragon Bone 2 % (10280) |
| "Defeated world master %s." (invader win) | 411 | Titanite Chunk | the same 90 / 5 / 3 / 2 gamble (10180) |
| "Defeated world master %s." | 412 | Pharros' Lockstone | + Smooth & Silky Stone (10200) |
| "Defeated world master %s." | 2015 | Bonfire Ascetic | + Human Effigy (10220) |
| "Abyss spirit %s has been vanquished." | 2100, 2115 | Bonfire Ascetic | + Human Effigy (10030, 10220) |
| "Destination reached. Returning to your world..." | 212 | Pharros' Lockstone | + Smooth & Silky Stone (10190) |

Two rows also changed a flag byte only (1205 `0x04020001` -> `0x04020101`, 2510 `0x04020100` -> `0x04020200`).

### Enemy drops (`ItemLotParam2_Chr`: vanilla 532, Scholar 615; 529 same, 3 changed, 83 new)

| lot | dropped by | vanilla | Scholar |
|---|---|---|---|
| 13800000 | enemy 138000 / 138001 (model c1380, AI label 拷問人（亡者） "torturer (hollow)"), Huntsman's Copse only | Cracked Red Eye Orb 10 % | 20 % |
| 13801000 | enemy 138010 / 138011 (same model, the whip carrier) | Bloodied Whip; Cracked Red Eye Orb 10 % | Bloodied Whip; Cracked Red Eye Orb 20 % |
| 6000000 | not named by any EnemyParam row (dropper not identified yet) | Awestone 3 % | 12.5 % |

Of the 83 new drop lots, 54 belong to enemies that spawn only on DLC maps and 27 to no enemy row that spawns
(5035000-5037500 and a run of 6xx00100-style variants). Two concern base-map enemies, and both fill a reference
vanilla already made to a lot that did not exist:
- **60101000**, the second drop slot (EnemyParam +0x50) of 601000 Salamander (AI label サラマンダー, model c6010;
  Forest of Fallen Giants, and in Scholar also Brume Tower): vanilla points at it with no row behind it; Scholar's row
  = Cracked Red Eye Orb 20 %, **Titanite Slab 3 %, Petrified Dragon Bone 5 %**. Scholar also adds a 601001 variant
  (DLC) that uses this list as its main drop (lot 60100001). Which slot the game rolls when is not known yet.
- **30501000**, the second slot of 305000 (model c3050, AI label 下位：デーモンナイト "lesser demon knight"; Iron Keep
  only, the Smelter Demon): added, but with nothing in it (ten Rubbish slots at 0 %), so no real change.

### New lots (117)

- **DLC map events** (82 lots: 9 in 5035xxxx Shulva, 55 in 5036xxxx Brume Tower, 18 in 5037xxxx Eleum Loyce): non-placed DLC rewards, e.g. 50356110 Crown of the Sunken King,
  50365050 Blacksteel Katana +5, 50365530 Catarina gauntlets + leggings.
- **DLC bosses**: souls 675000 (Fume Knight + Nadalia), 679000 / 679010 (Aava; Lud and Zallen), 680000 (Sir Alonne),
  681000 (Sinh), 682000 (Elana), 690000 (Ivory King), 305010 (Smelter Demon Soul again, named in the Brume Tower and
  Eleum Loyce event scripts); rewards 60013000 Smelter Wedge, 60014000 Scorching Iron Scepter, 60015000 Nadalia's
  soul, 60016000 / 60020000 / 60031000 the three crowns, 60030000 Aava's soul + Garrison Ward Key; 1788000-1788030
  Loyce armour and Alsanna's soul (100 % chance rows); 862000 / 862001 Twinkling Titanite x3, Petrified Dragon Bone
  x3, Titanite Slab.
- **Second copies of base-game boss rewards**: 60040000 Soul of the Last Giant (vanilla lot 309600), 60041000 Ancient
  Dragon Soul (60005000), 60042000 Forgotten Key (60001000), 60043000 Giant's Kinship + 60044000 Giant Lord Soul +
  60044001 Giant Lord Soul and Ring of Giants+2 (vanilla 309700 / 309701, one lot each), 60045000 Rotunda Lockstone
  (60006000), 60046000 Key to the Embedded + 60046001 Demon of Song Soul (602000). Vanilla's lots are all still there,
  unchanged. Two of the copies are wired into Scholar's map scripts **beside** the old lot: the Gutter
  (`event_m10_25`: 60001000 and 60042000) and Majula (`event_m10_04`: 60006000 and 60045000). No script, param or
  emevd names the other copies as a plain number, so what hands them out is open.

## 4. Text

ItemName: Scholar has every vanilla entry unchanged (1,163) plus 171 new ones (1,334). Vanilla's leftover names of
cut items survive: 3510000 **Shadow Claws** is still named in Scholar, still with no WeaponParam or ItemParam row, as in
vanilla. Scholar's params grew to 456 weapons (vanilla 403) and 1,268 items (1,076).

## 5. Open

- What hands out the duplicate boss lots 60041000 / 60043000-60046001 (code? a fallback so key items can't be missed?).
- The dropper of lot 6000000 (Awestone).
- Balance changes beyond items (damage, scaling, poise in WeaponParam / PlayerDamageParam etc.): the patch work
  the author expects in the PS3 build. Next comparison.
