# The "Broken" status: what became of Break (2026-11-12)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author asked about a darksouls2.wiki.gg note: non-player characters can get a **"Broken"** status that cuts the damage
they deal by 5 % and raises the damage they take by 5 % for 60 s. It stacks, each copy has its own timer, it shows
cracks on the character's equipment, and it builds up from Destructive Greatarrows, Acid Surge and Corrosive Urns.
Question: does the Network Test, PS3 v1.00 or the pre-release PC build say anything about it?
Background: [Break element](break-element.md) (the cut Break element), [upgrade paths and PC developer names](upgrade-paths-pc-dev-names.md#3-break-leftovers-the-names-identify) section 3.

Static work only: params, SpEffect scripts and code from the NT (US and JP), PS3 v1.00 and its patches, the six vanilla
PC builds and both Scholars. Nothing tested in game. Scripts: `..\tools\broken_status\` (`spfx.py` reads the SpEffect
scripts of any build; `brk_dmg.py` / `dtype.py` read damage rows; `pedis.py` / `pefunc.py` / `peread.py`
disassemble the PC exes; `ppcimm.py` finds constants in a PS3 EBOOT).

## Short answer

- **The wiki's numbers are correct**, and the effect is live in every retail build. It is SpEffect **900301**:
  outgoing damage x0.95 and incoming damage x1.05, each for 60 s, plus visual id 660. The script is the same in PS3
  v1.00, [the 31 March 2014 PC build](../builds/early-pc-builds.md#31-march-pre-release), PC 1.12, PS3 Scholar and PC Scholar.
- **It is not a cut feature.** It is what retail put in place of the Network Test's Break status. In the NT, Break
  was one status for everyone: when the build-up filled, the target got SpEffect 900300, which deals the
  "weapon break" damage row 9030. There is no x0.95 / x1.05 anywhere in the NT.
- **From v1.00 on, the code splits by character type.** Two of the 14 character categories still get 900300, but
  retail data no longer has a 900300, so nothing happens to them. Every other category, enemies included, gets
  900301. So enemies cannot lose equipment durability, and the Break build-up gives them a small debuff instead.
  The two flagged categories are most likely the player's own character types. **Not confirmed**: the
  table is indexed by a byte at character +0x54 that we haven't mapped.
- Break is still a damage type in retail (damage type 8). Retail added Acid Surge with it, after the NT.
  The "Break resist" defence slot also survives in the SpEffect scripts: [the Ring of Resistance](armour-rings-spells-nt-vs-retail.md#same-id-same-ring-17) quietly raises it.

## 1. The status in code

The status ailments go through one function, a switch on the ailment index.

| index | ailment | SpEffect (NT) | SpEffect (v1.00 onward) |
|---|---|---|---|
| 0 | poison | 900100 | 900100 |
| 1 | bleed | 900200 | 900200 |
| **2** | **Break** | **900300 for everyone** | **900300 if the character's category is flagged, else 900301** |
| 3 | curse | 900400 | 900400 |
| 4 | burning (燃焼) | 900500 | 900500 |
| 5 | toxic | 900600 | 900600 |

Where the ailment names come from: the 31 Mar 2014 PC build's developer names for the generic status-ailment rows,
DamageCtrlParam 99987010-99987060, run in the same order: 毒 poison, 出血 bleed, 武器破壊 weapon break, 呪 curse,
燃焼 burning, 猛毒 toxic. For index 4 there's more: 900500 deals PlayerDamageParam 9051, then 9050 every 0.2 s, and
the developers named those rows 燃焼-開始 "burning, start" and 燃焼 "burning". The exe's own ailment-type list (UTF-16
at PC Scholar file offset 0x10ce2a0) has 燃焼 as type 4, the type 900500 sets. Petrification (石化) is a separate type
there, 10. Which resistance the burning build-up checks isn't traced. (Corrected 2026-11-26: index 4 said petrify,
with no source given. The correction started from checking our notes against the SpEffect script dumps on
LordRadai's GitHub. Every value here comes from our own reading of the game's files.)

So Break is still ailment 2, between bleed and curse. That's the NT's menu order (poison, bleed, Break, curse).

- **NT** (`EBOOT.elf` 0x38b998): `lis r29, 0xd; li r28, 0; ori r29, r29, 0xbccc` = 900300, with no condition.
- **PS3 v1.00** (`EBOOT.elf` 0x486cd4): reads byte 3 of a 5-byte table entry chosen by the character's category,
  then `r25 = 900300 + ((flag - 1) >> 31)`: flag 0 -> 900301, flag set -> 900300.
- **PC 31 March 2014** (exe 0x5a164d): `cmp [category*5 + 0x115bd6b], 0; sete cl; lea esi, [ecx + 900300]`.
  This is the same rule. So the exe has no literal 900301, only 900300 + 1.
- **PC Scholar 1.0.3.0** (exe 0x140145d24): `category = byte [chr + 0x54]` (leaf 0x14014eca0);
  `neg flag; sbb r14d, r14d; add r14d, 900301` (the same rule again). The table is at 0x1410bfff0, with 14 entries:

  ```
  #   b0 b1 b2 b3 b4      b3 = the flag read here
  0   00 04 01 00 01
  1   02 00 01 01 01      <- flagged: 900300 (nothing in retail)
  2   02 01 01 01 01      <- flagged
  3   02 02 00 00 00
  4   02 02 00 00 00
  5   02 01 01 00 01
  6   02 04 01 00 01
  7   02 03 01 00 01
  8   02 03 00 00 01
  9   01 04 01 00 01
  10  01 03 01 00 01
  11  01 03 00 00 00
  12  01 01 01 00 01
  13  01 02 00 00 00
  ```

  Byte 0 splits the entries into three groups: 0, then 1-8 (value 2) and 9-13 (value 1). Byte 3 is also read at
  0x14032f061, where it adds 100 to an id, and at 0x14043a532. Byte 1 is read in many places. None of these name the
  categories.

## 2. The SpEffect scripts

| SpEffect | NT (US and JP) | v1.00 / PC 31 Mar / all later builds |
|---|---|---|
| 900300 | `170:0 (9030, 0)` deal damage row 9030 once; `40:2 (652, ...)` visual 652 | **gone** |
| 900301 | not present | `100040:1 (660, 249, 0)` visual 660; `1000:1 (25, 60 s, x0.95)`; `1000:1 (26, 60 s, x1.05)` |

`1000:1` index 25 = outgoing damage multiplier and 26 = incoming damage multiplier. Other users confirm this:
Iron Flesh 26 x0.6 for 25 s, the item 60406020 25 x1.15 for 5.5 s, and many enemy buffs (for example 96790100: 25 x1.2 /
26 x0.5 for 60 s). The visual ids line up across the ailments: NT poison 650, NT Break 652, curse 653 (NT and retail),
retail bleed 651, retail Broken 660. Visual 660 is probably the cracks the wiki describes. It isn't a param row
(no param has rows 650-660 with matching names), so its asset wasn't traced.

The NT's damage row 9030 is a placeholder: 20 damage of type 5, like its poison and bleed rows 9010 / 9020. So in the
NT, the actual equipment breaking must have come from code reacting to the row. Its control row is
DamageCtrlParam 99987030, "generic – status ailment – weapon break" (PC developer name). In v1.00 and later,
row 9030 is still there but empty: no damage and no SpEffect.

## 3. Break damage (damage type 8) in retail

In the damage params (PlayerDamageParam / EnemyDamageParam) damage type 8 is Break. The NT already used it:
Corrosive Urn 50 Break only, Destructive Greatarrow 100 Break. The PC developer names from 31 March 2014:

| row | developer name | v1.00 / PC |
|---|---|---|
| 33130000 / 10 / 11 | 呪術：酸の噴射（位置検索 / 発生演出 / その場停滞） "pyromancy: acid spray (position search / spawn effect / lingering)" = Acid Surge | 200 Break; **not in either NT** (no item, spell or damage row) |
| 60610000 | 武器破壊瓶 小 "weapon-break urn, small" (the hit) | 50 physical + 40 Break, durability mult 10 (v1.00) / 3 (PC) |
| 60610010 | 武器破壊瓶 小-爆発 "... explosion" | 40 Break, durability mult 10, **applies SpEffect 60610010** |
| 60900020 / 30 | 大矢弾-武器破壊-両手-弱 / 強 "greatarrow – weapon break – two-handed – weak / strong" | 100 physical + 100 Break, durability mult 19 |
| 6010 / 7010 | 汎用-装備破壊 "generic equipment break" | empty |
| 9030 | 武器破壊 "weapon break" | the status row, empty (see 2) |

Enemy attacks with Break damage (EnemyDamageParam, type 8): the 1121xxxx rows (50-70), 1205xxxx (25-80, the acid
horn beetles 毒角虫-酸), 1220xxxx (25-30, the giant acid horn beetle), and the NPC greatarrow 190100150 (20).

Corrosive Urn explosion, SpEffect 60610010: command `1000:9` changes durability in percent. Indexes 13-17 are what
the Repair sorcery (13 +20, 14 +20, 16 +30, 17 +10) and Repair Powder (15 +30, 16 +50, 17 +20) use, so 13 / 14 are
most likely the weapons and 16 the armour. The exact index meanings aren't code-checked.

| build | 60610010 |
|---|---|
| NT | not present (the NT urn has no explosion row) |
| PS3 v1.00, 1.01, 1.02; PC 31 Mar, 24 Apr, 1.02, 1.03, 2 Jun 2014 | 13 -10, 14 -10 (weapons only) |
| [PS3 1.03 onward](../builds/1.01-to-1.03.md#effect-scripts), PS3 Scholar, PC 1.12, PC Scholar | 13 -5, 14 -5, **16 -10** (armour added) |

On an enemy, these durability changes probably do nothing. What an enemy feels is the Break build-up that ends in
900301.

## 4. Break resistance left in the retail scripts

[The defence-side command](../unused/unused-ring-icons.md#what-the-starstones-might-have-done-the-code-2026-10-26) `100090:4 (duration, element, amount)` keeps Break as **element 7**. The rings give the
mapping: 0 physical, 1 magic, 2 lightning, 3 fire, 4 dark, 5 poison, 6 bleed, **7 Break**, 8 curse (Cursebite), 9 petrify
(Ash Knuckle).

- **Yellow Sea Troches** 60220000: SpEffect still present in every retail build (+20 Break resist for 90 s), though
  the item has no ItemParam row.
- **Ring of Resistance** 40150000: elements 5-9 +12, **including 7**, from v1.00. Its text only names poison, bleed,
  petrify and curse. The later Ring of Resistance+1 (+20) and Perseverance (+15) also include 7. Iron Flesh, Yorgh's
  Spear and the WideUse row 130000010 skip 7.
- NT: the attack-side command `80:1` lists elements 0-7, so 7 = Break there too (resins 60240000 / 60260000 set all eight).
- EnemyParam has nine defence values (physical, magic, lightning, fire, dark, poison, bleed, curse, petrify) and **no
  Break resist**. So how fast an enemy fills its Break build-up isn't set per enemy in that param.

## 5. Other developer names (31 March 2014 PC)

- [ArmorParam 991800-991803](armour-rings-spells-nt-vs-retail.md#armour) and EquipInterfereParam 199180-199183: 【魔術兵士破壊テスト】 "sorcerer soldier break /
  destruction test", head / chest / arms / legs. The armour rows are in v1.00 and later but not in the NT. The rows
  give no sign whether this tested Break or something else (for example armour parts breaking off).
- SpEffects are scripts, not params, so 900301 itself has no developer name.

## What this means for the wiki note

- 5 % / 5 % / 60 s / non-player: confirmed from data and code. The "non-player" part is really "not one of two
  flagged character categories" (see 1).
- Destructive Greatarrow, Acid Surge and Corrosive Urn: all three deal type 8 Break damage. So do several enemy acid
  attacks, which hit the player's durability.
- "Stacks with separate timers" and "entities with equipment": not checked. The code picks the SpEffect by character
  category only, not by equipment.
- On the original idea: the NT design (Break wears down equipment, for players and enemies alike) didn't plan a
  damage nerf. The x0.95 / x1.05 debuff is retail's stand-in for characters without durability.

## Open

- Which categories 1 and 2 are (character +0x54). Live check: read the byte on the player, a summon, an invader and
  an enemy (PC Scholar), and watch an enemy's SpEffect list after an urn.
- The Break build-up threshold for enemies and whether copies stack (SpEffect stacking flags, 100040:1 second
  argument 249?).
- What visual 660 points to.
- Whether element 7 (Break resist) does anything for the player in retail.
