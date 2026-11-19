# The wiki's "unknown" ring icons and the Network Test's cut rings (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author pasted [the community wiki's table of cut ring icons](../network-test/wiki-unused-content-crossref.md) (darksouls2.wiki.gg, "Cut Content", rings: five
"Unknown Ring" icons, the Ring of War and the two itemless Illusory Rings). Question: do any of them belong to the
Network Test's unused rings ([armour, rings and spells: Network Test vs retail](../network-test/armour-rings-spells-nt-vs-retail.md#rings), "Rings")?

Image: `unused-ring-icons.png` (wiki crops left of the bar, game files right). Tool: `tools\wiki_icons.py`
(`dump`, `unused`, `compare rings`; first written as `ring_icons.py`); the wiki crops are in
`unused-ring-icons\wiki1-8.png`. The same session matched the wiki's cut upgrade materials: [unused material icons](unused-material-icons.md).

## Answer

| wiki | where it is in the game files | Network Test ring |
|---|---|---|
| 1, tearstone-like | **vanilla PC only** (found 2026-11-09): an unnamed `IC_0040270000` in every vanilla PC build. Built on the tearstone ring design | **Rainbow Tearstone Ring** 40270000 (仮・虹色の涙石の指輪), cut. Tied by id since 2026-11-09 |
| 2-4, star rings (blue / purple / yellow) | **vanilla PC only** (found 2026-11-09): unnamed `IC_0040170000`-`IC_0040200000`. The fourth, red one is pixel-identical to retail's **Old Sun Ring** icon (40480000) | **the Starstone rings** 40170000-40200000 (仮・魔法 / 炎 / 雷 / 闇星石の指輪), cut. Tied by id since 2026-11-09 |
| 5, "Hornet Ring" | the Network Test's menu texture **`yubiwa_test`** ("ring test", `menu\02.febnd`, 64 × 128), the stand-in for an item-icon slot in the menu layouts | **none**: a placeholder, not an item icon |
| 6, Ring of War | the Network Test's own **IC_0040050000**; the same art (shifted 1 px) is Scholar's **IC_0041030000 Ivory Warrior Ring** | **Ring of War** 40050000, which retail kept under the same id as the Ring of Steel Protection, with new art |
| 7-8, Illusory Rings of the Guilty / Vengeful | retail IC_0040560000 / IC_0040570000, the only ring icons no retail item uses | none: retail-only |

So four of the eight belong to the Network Test's cut rings. On 2026-10-26, wiki 1 and 2-4 were matched only by
design, because no PS3 build or PC Scholar has their art. On 2026-11-09 their files turned up, under the cut rings'
own ids, in every vanilla PC build (see "Vanilla PC holds the missing icons" below). Wiki 6 is a kept ring, and
wiki 5 is a placeholder.

## How

- **Item icons** are `menu\tex\Icon\IC_00<icon id>.tpf`; the icon id is ItemParam +0x00 in both builds (the demo's
  extra word is +0x04). Every ring's icon id is its own item id, and "+1" / "+2" rings share the base icon.
- **The Network Test ships only six ring icons**: 40050000 Ring of War, 40140000 Dispelling, 40150000 Resistance,
  40260000 Red Tearstone, 40720000 Guardian's Seal and 40730000 Crest of Blood (the JP test has the same six). It has
  none for its cut rings, nor for the Royal Soldier's Ring, Ring of Restoration or Binding Ring.
- **No PS3 build and not PC Scholar has an icon under a cut ring's id** (40170000-40200000, 40240000, 40250000,
  40270000, 40380000, 40490000). Checked by name in the US and JP tests, v1.00, patches 1.01 / 1.03 / 1.10 and PS3
  Scholar, and by hash in PC Scholar's index (all 16,289 of its files are named, so it has no hidden ones).
  **Correction (2026-11-09): the vanilla PC builds have all nine**, as files no dictionary names. They weren't in
  the 2026-10-26 check (see "Vanilla PC holds the missing icons" below).
- **Retail's ring icons are all in use except two**: in v1.00 and on PC, every ring-range icon file has an ItemParam
  row using it, apart from 40560000 and 40570000 (the two Illusory Rings).
- **No ring icon was redrawn after v1.00**: v1.01, 1.03, 1.10, PS3 Scholar and PC carry the same 61 v1.00 ring
  icons unchanged (mean pixel difference under 12 out of 255).
- **Matching**: the wiki crops (about 60-90 px) were template-matched (OpenCV normalised correlation, colour, several
  scales) inside all 3,055 distinct item icons of all builds and over every menu texture (sheets and binders; 98 NT,
  660 v1.00, 430 PS3 Scholar, 462 PC). True pairs scored 0.77-0.88: wiki 6 against the NT Ring of War, wiki 7-8 against
  40560000 / 40570000, and wiki 5 against `yubiwa_test`. Wiki 1-4 found no such partner. Their best were the related
  designs: Old Sun Ring 0.82 / 0.79 for wiki 2 / 3, Blue Tearstone 0.76 for wiki 1. Everything else was 0.70 or below.
  Every pairing was checked by eye.

## Details

**Starstone rings (wiki 2-4).** The wiki's three star rings and retail's Old Sun Ring are one design: a round
signet engraved with a star, with a gem in the centre. The gem colours (blue, purple, yellow, plus the Old Sun
Ring's red) are the colour code of the four quartz rings (Spell blue, Flame red, Thunder yellow, Dark purple). The
Network Test lists four starstone rings in the same block as the four quartz rings (seven ids apart, see "Ring
order" below), in the same four elements (magic, fire, lightning, dark), and 星石 means "star stone". So the set is most likely the starstone rings, and the Old Sun Ring is
most likely the Flame Starstone's art given to a new ring. Retail put the Old Sun Ring at 40480000, the id of the
demo's name-only "Shield Crest Ring", so only the art links it to the starstones. **Confirmed 2026-11-09:** vanilla
PC keeps the four icons under the starstone ids, in id order blue (magic), red (flame), yellow (lightning) and
purple (dark). The Flame Starstone's icon and the Old Sun Ring's are the same picture, pixel for pixel. The demo has no text or effect for
the starstones. The wiki's guess that they "exploded in a Magic AOE" comes from the Old Sun Ring's likeness, and
nothing in the Network Test supports or rules it out.

**Rainbow Tearstone (wiki 1).** A thin multi-band ring in rainbow colours with a small stone, the same band
construction as the red and blue tearstone rings. The demo's 虹色の涙石の指輪 is the third tearstone ring, and retail
kept the other two.

**`yubiwa_test` (wiki 5).** The texture sits in the Network Test's in-game menu binder (`menu\02.febnd`, beside
`In-game_01` and `dummy_Chara`), 64 × 128, the size of an item icon. The menu layouts (read with
`..\..\DS2 Network Test\tools\flo.py`) give it **one sprite each**:
- in the Network Test, in the in-game, bonfire and NPC menus (`l02_01_In-Game`, `l03_01_Bonfire`, `l18_01_NPC`);
- in v1.00 and PS3 Scholar, in the in-game, inventory, equipment, bonfire and status menus.

Their texture tables list it beside `dummyIcon_01`, `IC_CA_dummy` and `dummy_Chara`. The developer test layouts
`l99_04` / `l99_05` list it too, and so does a 64 × 64 `yubiwa_test_02` that has no file. Retail PS3 keeps the sprites
but ships no `yubiwa_test` texture. PC Scholar ships one again (`menu\42.febnd`), but it is a card reading **"Dummy
Icon"**. (The PC layouts weren't read: flo.py handles only the big-endian PS3 ones.)

So it is a stand-in for an item-icon slot. Probably the menu draws the real item's icon there at run time; that was
not traced in code. No ring id points at it. The wiki's Hornet Ring reading has nothing behind it in the data, and
whether the art was first drawn for a planned ring can't be told.

**Ring of War (wiki 6).** The demo's own icon, unchanged in the JP test. In retail, id 40050000 became the Ring of
Steel Protection, with the same +50 physical defence effect (see [the rings note](../network-test/armour-rings-spells-nt-vs-retail.md#same-id-same-ring-17)) and **a new icon** (an ornate
oval signet). The demo's art came back in the Crown of the Ivory King DLC as the Ivory Warrior Ring
(IC_0041030000, from v1.10 and PS3 / PC Scholar), shifted one pixel. After that shift the mean difference is 2.7
out of 255, which is re-encoding noise; the same thing happened to the rainbow crystal's icon ([cut infusion crystals](../network-test/cut-infusion-crystals.md)).
In the demo the ring isn't unused: it is an item-lot pickup with a working effect.

## What the starstones might have done: the code (2026-10-26)

The author's theory: the quartz rings add elemental defence, so the starstones added elemental attack.

**Nothing names the starstones.** They have no event in the demo's `SpEffectRing.emevd`, their texts are
placeholders, and the demo's executable contains **no ring id at all** (`tools\immscan.py`: no lis/ori pair, compare or
data word for 40170000-40200000, nor for kept rings such as 40060000 or 40260000). Every ring effect is script-driven,
so the code has no special case to find. **The demo's quartz rings have no event either**: in the demo the only ring
with elemental defence is the Dispelling Ring (`90[11]`, +15 to types 1-4). The quartz rings' +5 each is retail's.

**But the demo's engine already had the mechanism the theory needs.** Bank 90 keeps two per-damage-type bonus lists,
nine types each (0 physical, 1-4 the elements, [5-8 the statuses](../network-test/broken-status.md#4-break-resistance-left-in-the-retail-scripts)):

| demo command | wrapper (summing `FUN_0048cea0`) | meaning | used in the demo by | retail command |
|---|---|---|---|---|
| 90[07] | `0x47baa0`, add (r8 = 1) | **attack up**, flat | the miracle War 戦争 (60 s, type 0 +50), two enemy effects 93180010 / 20 (type 0 +500) | 100090[02]: Ring of Blades (type 0 +20), Sacred Oath, Flame Weapon (type 3 +50), the **Clutch rings** |
| 90[08] | `0x47ba34`, subtract (r8 = 0) | attack down | nothing | not checked |
| 90[11] | `0x47b9c8`, add | **defence up** | Ring of War, Dispelling Ring, Ring of Resistance, War | 100090[04]: quartz rings (types 1-4 +5), Ring of Steel Protection, Magic Barrier, Sacred Oath |
| 90[12] | `0x47b95c`, subtract | defence down | nothing | 100090[05]: the Clutch rings' -80 physical defence |

- War is retail's Sacred Oath, which raises attack and defence. In the demo it carries both `90[07](60 s, type 0 +50)`
  and `90[11](60 s, type 0 +50)`, which is what fixes 7 as attack, next to the defence command the rings use.
- **Where 7 + 8 go:** `FUN_00392dec` sums them (and one more list, `0x47bb0c`). Its caller `FUN_0065fe78` builds
  nine per-type values as (base + bonus) x multiplier + add. It adds the 7 / 8 vector at x1.0 (TOC float 1.0) to the
  weapon's base value per type, and a second vector (`FUN_00392e74`, from other effect managers, not identified) at
  x0.01 to the per-type multipliers. Types 1-4 go through the same path as type 0.
- So in the demo, a starstone that raised one element's attack would have been one script line per ring,
  `90[07](0, k:N)` with k = 1-4: the attack-side twin of the Dispelling Ring's `90[11](0, k:15)`. No demo effect uses
  command 7 with an elemental type, so it was never exercised, but nothing in the code limits it to type 0.
- **Retail later built exactly that set.** The Scholar DLC's Sorcery / Lightning / Fire / Dark Clutch Rings
  (41040000-41070000) are `100090[02](0, k:30)` for k = 1-4, +30 attack of one element, plus `100090[05](0, 80)`.
  Their gems use the same colour code (blue, yellow-green, red, purple). Retail's quartz rings give the type order:
  1 magic, 2 lightning, 3 fire, 4 dark.
- The names pair too: the quartz rings are 晶石 ("crystal stone") and the starstones 星石 ("star stone"), in the same
  four elements and the same order, in one block of the demo's ring list (not side by side: see "Ring order" below).

**Ring order (2026-11-16).** the author remembered the starstones sitting next to the "crystal" rings. They don't: the
quartz rings are 40060000-40090000 and the starstones 40170000-40200000, with seven rings between. (The demo's
ItemParam +0x08 equals the ring id for every ring, so if that is the sort field the menu shows id order; not checked.)
But the block 40050000-40200000 is laid out by effect, and it mirrors itself:

| id | demo (JP, 仮・ dropped) | retail ring | effect (demo script where it has one, else retail) |
|---|---|---|---|
| 40050000 | 戦の指輪 Ring of War | Ring of Steel Protection | physical defence, demo `90[11](0, 50)` |
| 40060000-090 | 魔法 / 炎 / 雷 / 闇晶石の指輪 | Spell / Flame / Thunder / Dark Quartz | one element's defence each (retail; no demo event) |
| 40100000-130 | 解毒 / 止血 / 防護 / 阻呪の指甲 | Poisonbite / Bloodbite / Bracing Knuckle / Cursebite | one status each: types 5, 6, (7), 8 |
| 40140000 | 全ての退魔の指輪 | Dispelling Ring | all elements, demo +15 types 1-4 |
| 40150000 | 抵抗者の指輪 | Ring of Resistance | all statuses, demo +20 types 5-8; demo text: poison, bleeding, weapon breakage, curse |
| 40160000 | 王の剣の指輪 King's Sword | Ring of Blades | physical attack (retail `100090[02]` type 0 +20; no demo event) |
| 40170000-200 | 魔法 / 炎 / 雷 / 闇星石の指輪 | none | ? |

So: physical defence -> per-element defence -> per-status -> all elements -> all statuses -> physical attack ->
per-element starstones. The starstones follow the physical-attack ring exactly as the quartz rings follow the
physical-defence ring, same element order. Retail leaves 40170000-40200000 empty. Supporting, not proof (the King's
Sword's attack effect is retail's). Side find: the 防護の指甲 knuckle ring sits in the status-7 slot, which the demo's
Ring of Resistance text calls weapon breakage; retail renamed it 封壊の指甲 (Bracing Knuckle Ring, slows equipment
wear), so it was most likely the Break-resistance ring ([Break element](../network-test/break-element.md)). Added to the wiki-team Google Doc
("Where the starstones sit in the Network Test's ring list", under Rings) the same day.

**Verdict.** The code doesn't say what the starstones did. It shows that the demo's engine supported elemental-attack
rings with no new code, the attack twin of the elemental-defence ring it did use, and that retail later shipped that
exact design as the Clutch rings. That makes "elemental attack" the best-supported reading, ahead of the wiki's
exploding-ring guess. That guess rests only on the art: retail's Old Sun Ring uses a different command (`1000[10]`).
Whether the demo had any equivalent of that command wasn't checked.

## Vanilla PC holds the missing icons (2026-11-09)

The author asked for the check to include the pre-release and early PC builds, which weren't installed on 2026-10-26.
Image: `unused-ring-icons-pc.png` (wiki crops beside the PC files); the ten icons as PNGs in
`unused-ring-icons\pc-hidden\`. Tools: `..\..\DS2 Vanilla PC\tools\hidden_icons\` (`pcicons.py names`, `hidden.py`,
`final_sheet.py`).

**How they were found.** The PC archives store files by a hash of their path, not by name. Our name lists come from the
PS3 builds and from PC Scholar, so a file only vanilla PC has shows up as a bare hash: the 2014 PC indexes hold
about 560 such files. Hashing every possible ring-icon path (`menu\tex\Icon\IC_00` and `IC_10`, plus ids
40000000-42999999) against each index named ten of them as ring icons. (Three other hits were weapon model binders
whose hashes happen to collide; they are not icons.)

**Which builds.** [All six vanilla PC builds we have](../builds/early-pc-builds.md): the 31 March 2014 pre-release, 1.01 (launch), 1.02, the
regulation-only update of 2 June 2014, 1.03, and today's 1.12. Three of them hold the archive data (31 March, 1.02,
1.12), and in those the ten files are byte-identical. The other three were checked by index only. No PS3 build has
them (their archives are fully named), and PC Scholar dropped them.

**The ten icons** (64 × 128, 8,389 bytes each, like every retail ring icon):

| id | Network Test ring | icon |
|---|---|---|
| 40170000 | Magic Starstone | star signet, blue stone (= wiki 2) |
| 40180000 | Flame Starstone | star signet, red stone: **the Old Sun Ring's icon**, pixel for pixel |
| 40190000 | Lightning Starstone | star signet, yellow stone (= wiki 4) |
| 40200000 | Dark Starstone | star signet, purple stone (= wiki 3) |
| 40270000 | Rainbow Tearstone | the tearstone band in rainbow colours (= wiki 1) |
| 40240000 | Dragon Claw | a rough, jagged silver band: **the Ring of Giants' icon** (40290000), pixel for pixel |
| 40250000 | the demo's Ring of Thorns | a round iron signet with an engraved creature: **the Old Leo Ring's icon** (40295000), pixel for pixel |
| 40380000 | Rusted Iron | a rectangular copper signet: **the Stone Ring's icon** (40230000, the demo's 仮・大蜘蛛の指輪), pixel for pixel |
| 40490000 | Great Eagle Crest | a gold signet with a spread-winged eagle crest |
| 40600000 | none | **the King's Ring** picture, pixel for pixel |

**Matching the wiki crops.** Same template matching as before, against every vanilla PC ring icon (icons enlarged 2x
to the wiki's display size). Wiki 1 scored 0.96 against 40270000 (the tearstones 0.85 / 0.78), wiki 2 0.83 against
40170000 and wiki 3 0.95 against 40200000. Wiki 4 (yellow), a wider crop on a lighter background, didn't score, but it
is plainly 40190000 by eye. The other star signets score high against wiki 2-3 too (same design), so the colour
decides. All checked by eye.

**40600000 is not a cut ring.** It has no item row and no name in any build, but every retail build (PS3 v1.00, PS3
Scholar, PC) has the King's Ring's short and long descriptions under it too. It looks like a spare or earlier id for
the King's Ring (retail 40510000). In PC 1.12 the real King's Ring's long description was reworded, but this copy
wasn't.

So the wiki's mystery icons almost certainly came from the vanilla PC files. Of the four other cut rings, only the
Great Eagle Crest has art of its own. The icons of Dragon Claw, the demo's Ring of Thorns and Rusted Iron are retail
ring icons, found to match pixel for pixel by the full survey ([hidden PC icons](hidden-pc-icons.md), same day). Like the Flame
Starstone and the Old Sun Ring, the files can't say whether the art moved from the cut ring to the kept one, or a kept
icon was copied in as a stand-in.

## Not established

- **Who first pulled the icons, and when.** The wiki page gives no source.
- **What the cut rings would have done.** The Network Test has no effect script or text for any of them, and the
  icons add none.
- **40600000's history.** Why the King's Ring has a second id with its text and icon wasn't traced.
