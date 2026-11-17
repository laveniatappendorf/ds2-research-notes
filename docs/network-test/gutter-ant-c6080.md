# The Gutter's giant ant: `c6080`

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author, 2026-10-27): there is a giant ant in The Gutter. What was it originally for? It looks like a reused
asset; does the Network Test reference it?

**Answer: it is not a reused model. It is the one built part of a three-part creature the developers called the
腐食虫 *fushokuchū*, "corrosive insect".** The ant is character `c6080` (enemy 608000). Retail gives it its own model,
textures, rig and sounds, places exactly one in The Gutter, and lets it do nothing but stand and die.

(Updated 2026-10-28, after the author asked whether it is enemy 6220.) The developers' own row names survive in the PC
pre-release build of 31 March 2014 (and PC 1.01 / 1.02). They name three parts:

| Enemy | Developers' name | State |
|---|---|---|
| 608000 (`c6080`) | 腐食虫（外殻）：クズ底, "corrosive insect (outer shell): The Gutter" | the placed ant |
| 622000 (`c6220`) | 腐食虫（本体）, "corrosive insect (main body)" | never got a model |
| 623000 (`c6230`) | 腐食虫（子）, "corrosive insect (young)" | never got a model |

The shell's 16 abdomen vents were wired up as an **antidote gas**: 16 damage rows apply an effect that cures poison
and toxic. No animation fires them.

An event links the ant to three poison-free Poison Horn Beetles in the same cave. The Network Test already had all
three parts as placeholder enemy rows, in the large-creature / boss block. The name 腐食虫 is the Corrosive Urn's 腐蝕虫:
the same word, with a variant kanji.

*[image not published: c6080]*

## Retail (v1.00; PC Scholar the same)

| What | Value |
|---|---|
| Model | `model\chr\c6080.bnd`: 15.5 m long, 9.2 m across the legs, 4.6 m tall; own textures `c6080_d/_n/_s`; ragdoll |
| Placement | one generator in `m10_25` (The Gutter): generator 2700, regist 60800000, logic 608010, think 10, appearance event 30, reset by the **Upper Gutter** bonfire (25665). No other map, DLC included (PC Scholar: same generator 2700 and position, draw group 7 -> 4) |
| AI | none: no `ai608xxx` script in the NT, v1.00 or PC Scholar |
| Animations | idle (0), weapon idle (7001, 6 s), two deaths (170010 / 170020), ragdoll loop. No walk, turn or attack |
| Timing files | `c608000_sfx.tae` has no events; `_snd` plays sounds 8002 and 4000 on death; `_pl` idle events only |
| Sounds | three: c608004000, c608008002, c608099999 |
| `EnemyParam 608000` | HP 3070, 300 souls (PS3 v1.00 and Scholar; **1800 on PC Scholar**), drop lot 60800000 = **Poison Moss** at 100 % (as the param reads; not checked in game) |
| Other rows | `EnemyMoveParam 608000` is a byte-for-byte copy of the Poison Horn Beetle (205000) and Razorback Nightcrawler (206000) rows, so it was set up from the bug templates. `EnemyAttackParam 608000` is the generic default row shared by 200+ enemies |

Nothing else in retail uses its textures (byte search of all 11,457 files: only its own model, textures, animations and
sound bank).

Community name: **"Corrosive Ant Queen"** (the SOTFS enemy-defences sheet and the community cut-content sheet). The game
has no name for it, and no developer row says "queen" (`community-cut-sheet-crossref.md`). **PC Scholar adds four small
"Corrosive Ants"** (enemy 672002) in The Gutter. Their model is c6720, which is new in Scholar (no c6720 files in the NT or v1.00),
so nothing ties them to the young (623000). They replace Captive Undead at generators 2520-2523, about (-220, 8, -25),
roughly 30 m from the big ant at (-226, 3, -54) and 5 m above it. PS3 v1.10 has none (`enemy-counts-v110-vs-pc.md`).

### The rig is built for more than retail uses

168 bones, from `c6080.flv`:

- six legs (`L/RLeg1-3`, each with collarbone, two segments, palm, finger, dummy controller);
- four more pairs of limbs, `L1-L4_Arm` / `R1-R4_Arm`: antennae, palps or mandibles (the names don't say which);
- `Mouth1`, `Mouth2`;
- a 15-segment abdomen (`Tail1-15`) with five spring bones per segment on segments 3-10 (`TailN_Spring_C1/L1/R1/L2/R2`),
  so the abdomen wobbles;
- **8 pairs of "holes" along that abdomen** (`L_Hole1-8`, `R_Hole1-8`), each with a `_DmyCtrl` dummy controller. The
  model's dummy points include two runs of eight, 30-37 and 40-47, which fit the 16 holes.

Dummy controllers are where a model's effects and projectiles come from, so the abdomen was built to emit something
from 16 vents. Retail never fires them: its effects timing file is empty and the idle has no effect or projectile
events. The animation network (`c6080.nmb`) is the shared template, and its state names (Attack, CatchAttack, ...) are
in every character's network, so they say nothing about this one.

**What the vents emit: antidote gas** (found 2026-10-28; rows identical in v1.00, PC pre-release and PC Scholar):
- **`EnemyBulletParam 160800010`**, developers' name `c6080 腐食虫　毒消しガス`, "c6080 corrosive insect: antidote
  gas". It is a slow 2-second puff (effect 1989, 0.7 m/s, small random spread).
- **`EnemyDamageParam 160800010`-`160800160`**: 16 rows, one per vent. Each hit box sits on dummy 30-37 or 40-47
  (row 010 on dummy 30 … row 160 on dummy 47) and applies SpEffect 96080010. It does stamina and knock-back damage
  through the generic "no-motion bullet, physical, large" control (`DamageCtrlParam` 590009300).
- **SpEffect 96080010** (`SpEffectEnemy.emevd`, retail only; the NT's script has no such event) runs the same three
  steps as the **Monastery Charm**'s cure: a shared 873 instruction, then cure status 0 (poison) and status 8 (toxic).
  It just leaves out the Charm's HP restore. Poison Moss cures the same two.

So the ant was meant to puff out a gas that cures poison, in a level full of poison, and it drops Poison Moss. Nothing
in the data says whether the gas was meant to help the player or to protect its own brood.

## The developers' names (PC pre-release, 31 March 2014)

The early PC builds (31 Mar pre-release, 1.01, 1.02) keep the developers' Japanese row names in their params, as UTF-16.
`paramdef.py` reads them since 2026-10-28; PS3 builds and later PC builds have none. Every row tied to the ant:

| Param | Rows and names |
|---|---|
| `EnemyParam`, `EnemyMoveParam`, `EnemyCommonParam` 608000 | 腐食虫（外殻）：クズ底, "corrosive insect (outer shell): The Gutter" (クズ底 is The Gutter's Japanese name) |
| `EnemyBehavior` / `Second` / `Third` / `Combo` 608000 | 腐食虫（外殻） |
| `EnemyAttackParam`, `EnemyDefaultLogicParam` | 608000 外殻 "outer shell"; **622000 腐食虫（本体） "main body"**; **623000 腐食虫（子） "young"**; and 621000 達磨 "daruma", a separate cut enemy |
| `EnemyEquipWeaponParam` | 60800000 外殻; 62000000 達磨, 62100000 本体, 62200000 子 (the weapon rows run one step below the enemy ids) |
| `ItemLotParam2_Chr` 60800000 | 腐食虫-外殻 (the Poison Moss drop) |
| `Damage*MotionWeightParam` 6080 | 超超特大 吹き飛び, "ultra-ultra-large, blown away" (its size class) |
| `MapObjectParam` 6080 | 【汎用】c1500スタチューナイトの首, "[generic] c1500 Statue Knight's head": the `o00_6080` prop, confirming the number clash |

The Gutter's generators, read from the same build's `GameDataEbl`:
- **Generator 2700** (the ant): 【梯子エリア前の洞窟】　腐食虫, "[cave in front of the ladder area] corrosive insect". Its
  spawn entry 60800000 is 【腐食虫（外殻）】.
- **Generators 2710-2712**, numbered straight after it: 毒なし毒角虫, "poison-free Poison Horn Beetle", in the same
  cave. Their spawn entry 20500100 is 【毒角蟲：状態異常なし】, "Poison Horn Beetle: no status effect"; they are enemy
  205001, AI `ai205001_毒角虫（毒なし）`.

**The ant is tied to those beetles in retail.** The Gutter's event script (`event_m10_25_00_00.edd`, v1.00, Scholar
and PC) has an event 腐食虫と連動する毒角虫, "Poison Horn Beetles linked to the corrosive insect". It has:
- inputs 腐食虫のジェネレータID (the ant's generator ID) and 腐食虫の生成判定 (spawned / not spawned);
- the flags 腐食虫の稼働フラグ (the ant's "active" flag) and 腐食虫の生成フラグ ON (spawn flag on);
- states 【条件】 condition, 【実行】 run and 【再現】…_空 "replay, empty".

Generators 2710-2712 appear through event 125020025, not the plain appearance event 30 the ant uses. What the event
does in play is not decoded. A reading, not proven: the poison-free beetles stand in for the ant's never-built
"young".

So "is the ant enemy 6220?": no. The ant you see is 6080, the **shell**. 6220 is the creature's **main body** and 6230
its **young**. Neither ever got a model or files (no `c6220` / `c6230` in any build), and no names survive for what
the body was to look like.

## Network Test (Aug 2013)

- **No model, textures, animations, sounds, AI or text.** A byte search of all 3,444 NT files for `c6080` finds nothing.
  (Control: the same search for `c6090` finds the Giant Guard Captain's textures in `o00_4903`.)
- **But the slot exists:** `EnemyParam 608000` with body 6080 (and no behaviour row, 0). It is the same 1200-HP
  placeholder as the NT's other large-creature rows: 600000 Ancient Dragon, 601000 salamander, 602000 Demon of Song,
  607000 Old Iron King, 609000 / 609100 Giant Guard Captain, 621000-623000. Word by word they differ only in their ids
  (body, behaviour / common / move, weapon) and two or three small fields. Damage-reaction rows 6080 exist too
  (`DamageMotionWeightParam`, `...LadderWeightParam`, `DamageDeadMotionWeightParam` identical to retail;
  `DeadDamageMotionWeightParam` differs), and `EnemyAttackParam 608000` is the generic default row.
- So by August 2013 the ant already had a number in the 60xx block, the block for big creatures and bosses, between
  Freja (6030), the Old Iron King (6070) and the Giant Guard Captain (6090). The NT has nothing more: none of the 27 NT
  boss souls (`nt-bosses-not-in-retail.md`) is an insect, no AI label names an ant, and no developer sign in the NT's
  Gutter model (`developer-signs.md`, m10_25) mentions one.
- **The NT also has the other two parts** (2026-10-28): `EnemyParam` 622000 (body 6220) and 623000 (body 6230), the same
  1200-HP placeholder, plus `EnemyAttackParam` 622000 (identical to 608000's). Earlier notes listed them as "unnamed
  NT-only slots, probably bosses"; the PC developer names show they are the corrosive insect's main body and young.
  621000 (body 6210) beside them is 達磨 "daruma", a different cut enemy. The NT has no antidote-gas rows (no
  `EnemyBulletParam` / `EnemyDamageParam` 1608xxxxx, no SpEffect 96080010) and no Gutter params.
- **What sits behind those two enemy rows in the NT** (checked 2026-10-28). Every entry is a template copy:
  - **main body 622000**: the generic `EnemyAttackParam` row (shared with 259 other enemies) and weapon row 62200000
    (2 bytes off the generic 912000 / 60000000 rows). Its `EnemyCommonParam` and `EnemyMoveParam` ids point at rows that
    don't exist.
  - **young 623000**: the generic attack row; an `EnemyCommonParam` row one byte off the Guard Captain's (609000) and the
    daruma's (621000); and an `EnemyMoveParam` row identical to 13 others, among them Agdayne 506500, the Lich, the
    Wraith, the phantom knight and mage, and King Vanclad (past). Its weapon id 62300000 points at a row that doesn't
    exist.
  - Neither has behaviour or combo rows.
  - The fields were matched to params by checking where each field's values exist as row ids (NT layout, not the
    shared decoder).
- **A placeholder row doesn't rule out a boss** (checked 2026-10-28). Each retail boss's NT `EnemyParam` row was compared
  with the ant's. HP is at +0x28 in the NT layout. Three kinds:
  - **The same 1200-HP placeholder as the ant** (at most 4 words off it, ids aside): Covetous Demon 500000, Mytha 501000,
    Najka 503000, Darklurker 506100 (developers' name アガドゥランのスタンド, "Agdayne's Stand"), Vendrick 514600, Ancient
    Dragon 600000, Demon of Song 602000, Old Iron King 607000, Dragonrider 611010. A second template at 1680 HP covers
    Old Dragonslayer 625000, Lost Sinner 626000 and Nashandra 627000.
  - **Rows of their own but far below retail HP**: Guardian Dragon 600, Smelter Demon 490, Flexile Sentry 530, Ruin
    Sentinel / Rotten / the three throne knights / both rat bosses 560, Freja 1400, Looking Glass Knight 1440, Last Giant
    1820, Giant Lord 2420, Pursuer 3600.
  - **Set up for play**: only the Copse's Skeleton Lords (605 / 605 / 405) and Executioner's Chariot (2525).
  - The Lost Sinner has no more in the NT than the ant: a template row, with no AI script, boss row or soul. So the NT's
    silence says nothing either way about the ant.
- Two number clashes to ignore: NT `o00_6080` is an unrelated 0.5 m prop with `c1500` textures, and the item icon
  `IC_0060800000` (both builds) is the Fire Arrow, item 60800000.

## The only ant in the text: the Corrosive Urn

Retail's Corrosive Urn (60610000) is filled with "secretions of giant corrosive ants", and their anthills "line the
horizon of the desert land of Jugo" (Japanese 腐蝕虫と呼ばれる大型の蟻, the land ウーゴ). The NT already had the item,
as 紅液瓶 "crimson-fluid bottle", "destroys equipment with an acid solution", with no long description. So the ant lore
was written after the NT.

The link is the name (corrected 2026-10-28; this said "nothing links" before the developer names turned up). The
Urn's 腐蝕虫 and the developers' 腐食虫 are the same word, *fushokuchū*, with 蝕 "eclipse / erode" for 食 "eat" (a
common variant). So the Urn's giant corrosive ants are this creature's kind. Still, `c6080` has no acid attack: its
only wired-up effect is the antidote gas, and it drops Poison Moss.

## Every other ant in the game (searched 2026-10-28)

The author asked whether anything else, in the NT or retail, mentions ants. Searched:
- every FMG of the NT, the JP NT, v1.00, the JP disc, PS3 Scholar and PC Scholar, all languages (`\bant(s|hill|lion)?\b`,
  蟻, アリ, 腐食虫, 腐蝕虫);
- UTF-16 strings in every event script, AI script, map layout and prefab of the NT, v1.00, Scholar and PC;
- the developers' row names in the PC pre-release regulation and all 345 of its per-map params.

**Text: the Corrosive Urn and nothing else.**
- The full description (60610000, unchanged v1.00 to PC Scholar): "secretions of giant corrosive ants"; giant
  anthills line the desert land of Jugo; "Ant larvae are considered a delicacy, but he who enters an anthill uninvited
  will have their flesh melted to the bone" (幼虫は美味で知られるが、うかつに塚を壊すと骨まで溶かされてしまう).
- **Scholar changed one character**: 腐蝕虫 became 腐蝕**蟲**, the older form of "insect".
- The NT has no ant in any text. Its Urn is 紅液瓶 "crimson-fluid bottle", with no ant lore.
- Gavlan's マイドアリ！ is "many thanks" (毎度あり), not an ant.

**Internal names: the corrosive insect, plus two place labels:**
- **The corrosive insect** (everything above): the Gutter's enemy, generator, event, bullet and drop rows. Nothing
  outside the Gutter uses 腐食虫.
- **Brightstone Cove's 蟻地獄 "antlion pit"** (literally "ant hell"). The developers called the funnel-shaped sand pit
  by the lower bonfire an antlion's trap. Uses:
  - the sliding floor `SlideFloorParam` 10140000 蜘蛛の街_蟻地獄 "spider town: antlion pit";
  - the bonfire 14650 輝石街ジェルドラ_篝火01_蟻地獄横の建物 "Brightstone Cove Tseldora, bonfire 01, the building beside
    the antlion pit" (Lower Brightstone Cove);
  - its event `event_m10_14` 再生の種火01_蟻地獄横の建物;
  - ropes "over the antlion pit", treasure corpses "on the building before the antlion pit" and "for the spider trap
    beyond the antlion pit";
  - five generators "beside the antlion pit": parasite spiders, a Dark Wizard, a basilisk and a stone guardian.

  It's a figure of speech: no ant.
- **Shaded Woods' 腐食獣 "corrosive beast"**: three light points in the `m10_32` layout (ライト腐食獣03 / 004 / 005).
  The map's other lights are labelled by area (bridge, village, upper level, forest, puddle, river), so this one names
  a spot after a creature. No param, enemy or event uses 腐食獣. Where it sits is not pinned down: the light points
  don't line up with the generator positions in either coordinate frame, for every light in the map, so the frame
  wasn't solved.

**The Urn's acid comes from beetles in the data, not ants.** Its sources (v1.00, named from the PC pre-release):
- drops of 毒角虫：酸：アマナの祭壇 "Poison Horn Beetle: acid: Shrine of Amana" (205102; AI `ai205002_毒角虫（アマナ）`) and
  巨大毒角虫：酸 "giant Poison Horn Beetle: acid" in Aldia's Keep (220001) and Amana (220002);
- a corpse in Brightstone Cove's spider-web area, a skeleton in Aldia's Keep's dungeon, and a wooden chest "beyond the
  acid pool" in Drangleic Castle;
- the Rat King's shop, where the row is 【ねずみの王】強力な酸の瓶 "[Rat King] strong acid bottle".

So the ant lore lives only in the description. The game's acid creatures are the horn beetles, and the Gutter ant sits
with poison-free horn beetles tied to it by event.

## Ruled out on the way

`o25_6000-6002`, the other creature-like objects placed in the Gutter, use The Rotten's textures (`c3260`): they are
his corpse pieces (body with cleaver, arm, a falling body), not the ant.

Tools (scratch, not kept): MSB / generator / param listings with `msb2.py`, `paramdef.py`, `fmg.py`; render with
`tools\unused_models\chrsheet.py`; bone and dummy list read from the FLVER header. 2026-10-28: developer names from
`DS2 Vanilla PC\2014-03-31` (regulation + `GameDataEbl` via `pcindex.py`); event names from `event_m10_25_00_00.edd`;
effect script decoded with `tools\speffect.py`. SmithBox's DS2 community row names carry the same names (from PC 1.01).
