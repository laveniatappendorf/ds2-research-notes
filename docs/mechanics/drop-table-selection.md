# Enemy drop tables: the phantom's table and the bonfire-intensity ladder (PC Scholar 1.0.3.0, 2026-11-20)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

Every dropper in Dark Souls II carries two item-lot fields, not one: `EnemyParam` (CHR_PARAM layout)
`death_itemlot_id` and `overkill_death_itemlot_id`, and on the map side a generator row's `ItemLotID1` and
`ItemLotID2`. 487 droppers point both at the same row and 708 point them at different rows, so for a good
number of enemies it matters which one the game reaches for. This note is about what decides that.

## Short answer

Two separate mechanisms, and neither of them is overkill.

```
base = generator ItemLotID1, else EnemyParam death_itemlot_id      <- you own the world
base = EnemyParam overkill_death_itemlot_id                        <- you are a summoned phantom
then: try lot (base + bonfire_intensity - 1), walking DOWN one id at a time until a row exists
```

The first mechanism is **who owns the world you are standing in**. The field Smithbox calls
`overkill_death_itemlot_id` is the drop table used by a **summoned phantom**, and the host never sees it. The
second is **bonfire intensity**, which shifts the lot id upward by one per level, and is what makes a handful
of enemies drop different things in NG+ or after a Bonfire Ascetic.

Read from `DarkSoulsII.exe` and traced end to end. Not yet watched live.

## 1. The code

`CHR_PARAM` rows are 360 bytes; `death_itemlot_id` sits at `+0x134` and `overkill_death_itemlot_id` at
`+0x138`. `FUN_1401e2580` is the selector:

- It takes the character's CHR_PARAM row, its generator row, and a count.
- It calls a predicate. When the predicate is false it starts from the generator's `ItemLotID1` (`+0x88`), and
  if that search finds nothing it falls back to CHR_PARAM `+0x134`. When the predicate is true it starts from
  CHR_PARAM `+0x138` instead, with no fallback at all.
- Either way it then loops: try lot `base + count - 1`, and if no such row exists decrement by one and try
  again, up to `count` times.

The count comes from `FUN_140412570`, which adds the area's ascetic count to the NG count and clamps the total
to 8. That is bonfire intensity.

## 2. The branch is "do you own this world?", not "was it a big hit"

The predicate is a virtual call on the game-mode global, so its target is not resolvable statically. What it
means is settled by what else it gates:

- The **world item** path, `FUN_1401e26f0`, is built the same way, and when the predicate is true it returns no
  lot whatsoever. That is the long-documented "a summoned phantom cannot pick up items" behaviour.
- It cannot simply mean "this is an online session", because then no host playing online would ever get a
  normal enemy drop.

So it separates the owner of the world from a guest in it. Three things in the data fall into place once you
read it that way:

- **Sir Alonne.** His own table (lot 68000000) exists but is empty; his armour sits at 5% a piece on the
  phantom table (68000100). Players summoned into an Alonne fight report a pickup appearing on the floor that
  the host cannot see, which is what a guest rolling its own lot client-side would look like.
- **The Drangleic Castle wooden-chest mimic.** The host gets a guaranteed Washing Pole; the phantom table hands
  over a Petrified Something instead.
- **Most enemies give a phantom a token.** Of the 917 EnemyParam rows that carry a phantom lot, 656 point at
  lot 5000000, which is a single Lifegem at 5%. Another 224 resolve to an empty or missing row and so give
  nothing at all — 220 of those point at lot 40, which exists but holds no items. That leaves only 37 enemies
  in the game whose phantom table holds anything distinctive.

`overkill_death_itemlot_id` therefore looks like a misnomer. Nothing in the params or in the selector refers to
overkill, and Sir Alonne rules it out on his own: a boss with a co-op-scaled health pool is not going to be
overkilled, yet his set does drop.

## 3. Bonfire intensity is a separate ladder

Because the search starts at `base + intensity - 1` and walks down, a row at `base + k` takes over from
intensity `k + 1` upward. Only **23 of 539** base lots have any `base + N` sibling, so for most of the game
intensity changes nothing about what drops. Where it does:

| Base lot | Dropper | Intensity 1 | Higher |
|---|---|---|---|
| 2210500 | The Pursuer, Drangleic Castle | no row (falls back to the enemy's own lot) | **2+: Ring of Blades+2, 100%** |
| 2210600 | The Pursuer, Drangleic Castle | no row | 2+: Twinkling Titanite · 3+: Washing Pole + Twinkling Titanite |
| 1250500 | Heide Knight (Iron Mask & Spear), The Gutter | Iron Mask 100%, Heide Lance 100% | 2+: 20% each of mask, chainmail, gauntlets, leggings, **Heide Greatlance** |
| 1190000 | Alonne Captain (Greatbow), Iron Keep | Alonne Greatbow 1%, among six other items | **2+: Alonne Greatbow, 100%** |
| 30100000 | Heide Knight (unused row) | Heide Lance 100% | 2+: 20% each of the set, Heide Lance, Heide Greatlance |
| 30101000 | Heide Knight (Spear), several areas | Heide Spear 100% | 2+: 20% each of the set and Heide Spear |
| 30102000 / 30103000 | Heide Knight (unused rows) | Soul of a Great Hero 100% | 2+: 20% each of the set, plus Heide Lance / Heide Greatlance |
| 2213000 | Mimic (wooden chest), Drangleic Castle | Washing Pole + Petrified Dragon Bone, both 100% | 2+: Petrified Something + Petrified Dragon Bone |
| 12300000 | Suspicious Shadow, No-man's Wharf | set and weapons at 1% | 2+: the same at 5% |
| 15200100 / 15201100 | Royal Swordsman, Forest / Lost Bastille | Lifegem 8% | 2+: Radiant Lifegem 8% |
| 4030000 | Dark Chasm of Old enemies | Bonfire Ascetic 20% | 2+ through 8+: Bonfire Ascetic 50% |
| 82400000 | Aslatiel of Mirrah, Aldia's Keep | Old Mirrah Greatsword 3% | 2+: 5% |
| 83600000 | Mad Warrior Phantom, Belfry Sol | set and Berserker Blade 3% | 2+: 5% |
| 83700000 | Foreign Wanderer Phantom, Things Betwixt | set and weapons 3% | 2+: 5% |
| 84400000 | Catarina Knight Phantom, Forest of Fallen Giants | set 3% | 2+: 5% (the two weapons stay at 3%) |
| 85340000 | Greatbow Phantom (unused) | Durgo's Hat 3% | 2+: 5% |

Five more have siblings that change nothing: 20300000 (Parasite Spider) and 85400000 (Aurous Knight Phantom)
are identical copies, and 2240000, 84900000 and 85000000 (Nameless Usurper, Painting Guardian Phantom) are
empty or unchanged.

## 4. What this means for particular items

| Item | Reading from the data |
|---|---|
| **Bone Fist** | The only weapon whose *enemy drop* requires being summoned. Lot 66101000 is `base + 1000`, far beyond the intensity ladder's reach of `+7`, and no other enemy lot anywhere holds it. 0.1%. The weapon itself is not summon-locked: it also appears once in `ItemLotParam2_Other`, row 50376570 (1%), the world and chest table for m50_37, so there is a non-enemy source in the Frozen Eleum Loyce map. What that row is attached to was not traced. |
| **Heide Greatlance** | Obtainable solo. Lot 1250501 is reached both as the Gutter knight's `ItemLotID2` and as `ItemLotID1 + 1`, so a host gets a 20% shot at bonfire intensity 2 or higher. |
| **Alonne's Set** | Phantom table only — Sir Alonne's own lot is empty. 5% a piece. |
| **Ring of Blades+2** | The Drangleic Castle Pursuer at bonfire intensity 2+, guaranteed. |
| **Alonne Greatbow** | 1% from the Iron Keep Alonne Captain at intensity 1, guaranteed at intensity 2+. |
| **Sanctum Mace** | Not gated at all. Mace-carrying Sanctum Soldiers drop it at 2% on their own table; the 0.1% entries belong to the bow and lance variants' phantom tables. |

## 5. A suggested paramdef change

In the DS2S `CHR_PARAM` definition, `overkill_death_itemlot_id` would be better named for what it does —
`phantom_death_itemlot_id`, or `guest_death_itemlot_id`. The `GENERATOR_PARAM` description for `ItemLotID2`
("Adds itemlot to drop when this enemy is killed. Overkill version.") wants the same treatment, and
`ItemLotID1`'s description could note the bonfire-intensity offset, which does not appear to be documented
anywhere at present.

## 6. Checking it yourself

In Smithbox, with a DS2S project on the SOTFS executable:

- `EnemyParam` → `id ^661000$` → `death_itemlot_id` is 66100000, `overkill_death_itemlot_id` is 66101000.
- `ItemLotParam2_Chr` → `id ^66100000$` → Old Radiant Lifegem 2.5%, Petrified Dragon Bone 5%, and no Bone Fist.
- `ItemLotParam2_Chr` → `id ^66101000$` → `item_lot_2` is 3530000 (Bone Fist), `chance_lot_2` is 0.1.
- For the ladder: `ItemLotParam2_Chr` → `id ^1250500$` and `^1250501$`, with the generator row
  `GeneratorParam_m10_25_00_00` → `id ^2090$` → `ItemLotID1` 1250500, `ItemLotID2` 1250501.

In Ghidra on `DarkSoulsII.exe`: `FUN_1401e2580` (the selector), `FUN_1401e26f0` (the world-item path, for the
predicate's meaning), `FUN_140412570` (bonfire intensity).

## What is not established

- **None of this has been watched in game.** It is read from code and params only.
- The predicate is a virtual call and its target was not resolved statically. "Owner versus guest" is argued
  from what else the same predicate gates, not read off a symbol.
- Item Discovery is not modelled anywhere here; every percentage is the raw `chance_lot_N` value.
- Whether a guest's roll is also reported to the host, or stays purely client-side, was not traced.
