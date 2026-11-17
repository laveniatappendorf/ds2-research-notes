# Bosses the Network Test knows that retail does not

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-10-25. The author asked for every reference to a boss in the Network Test (NT, US `NPUB90979`) that is not in the retail game
(v1.00, `BLUS41045`), across the existing notes and the data: dialogue, AI scripts, placements and the boss-soul list. Translations
are mine. "Retail" below means v1.00 unless stated; where a later build matters it is named.

## Short answer

- **The NT places no boss that retail lacks.** Its only map is Huntsman's Copse (`m10_23`); its generators place the Skeleton Lords
  (three, enemies 154000 / 154010 / 154020) and the Executioner's Chariot (619110), exactly as retail's Copse does. Everything
  NT-only about bosses is data: parameter rows, AI scripts, names, 27 boss souls and a little dialogue.
- **One NT-only boss is clearly identified: the "Giant Guard Captain" 巨人親衛隊長** (bodies c6090 / c6091): boss rows, a health-bar
  name, a test AI script and a stone-giant object, with no model and no retail enemy. (2026-11-03: the petrified
  Guard Captain objects `o00_4900`-`4903` are named 抵抗の砦 "Fort of Resistance", the developers' working name for the
  Forest of Fallen Giants, so he was a Forest boss; `e3-demo-leftovers.md`.)
- **Several more are only slots**: three characters with no NT name (c6210 / c6220 / c6230), two giant-scale objects (c6240,
  c6245), a boss row for c6100, a Lich and a Wraith (both are boss-range ids; the Lich's script survives only in retail) and
  Agdayne as a boss. See table 1.
- **Most of these slots now have names (2026-10-28, section 7).** The PC pre-release build of 31 March 2014 keeps the
  developers' row names. They make:
  - c6240 / c6245 the **Giant King** (巨人王, the first one marked "scrapped"), later rebuilt as the Last Giant and Giant Lord;
  - c5130 **King Vanclad (past)**, and c5140 / c5145 King Vanclad (present), for cutscenes and in-game;
  - c6090 the Guard Captain, with an **after-petrification** form and a **phantom**;
  - c6100 a Dragonrider-family "heavy knight";
  - c6210 達磨 "daruma", and c6220 / c6230 the Gutter ant's body and young.

  Five more slots the NT has rows for, unlisted until now, are named too:
  - c6060 **Hermit Crab Castle**, matching the 宿り大蟹 soul and the crab tower;
  - c6130 **Brainstem Sacred Tree**, matching the 古の神木 soul;
  - c6140 **Two-headed Wyvern**, matching the 双頭竜 soul;
  - c6150 **Leviathan** and c5050 **dread beast**;
  - c6160 **King Vanclad (transformed)**.
- **The soul list has 27 entries; 13 name nothing retail has** (a hermit crab, an earth dragon, Hell Scorpion, Death Stalker, a
  mayfly dragon, a rock giant, Agdayne, a sacred tree, a two-headed dragon, two hands, a witch, an executioner). Table 2.
- **The NT also shows retail bosses under earlier names and forms**: Vendrick as "Old King" and "King (present day)", the Demon of
  Song as a "sweet-voiced frog" and a "giant frog demon", the Rotten as "Butcher", and so on. Table 3.
- **Dialogue**: no NT line names an NT-only boss outright, but the NT-only text talks about a demon-headed sacred-tree forest, a
  dragon egg, giants and a rat king (section 5). (Corrected 2026-10-27: the Milfanito's thanks for calming the cursed
  singing, listed here before, is in retail too under new ids.)

## 1. NT-only boss slots, what the NT holds and what retail kept

Evidence is by character number (cNNNN), checked in params (`EnemyParam` and the behaviour, attack, move and boss rows), AI scripts
(`ezstate\ai*.esd` headers), text (all FMGs), objects and archives. "No model" means no `model\chr\cNNNN.bnd` in the NT or in retail.

| Slot | NT evidence | What retail kept | Read |
|---|---|---|---|
| **Giant Guard Captain 巨人親衛隊長** c6090 / c6091 | `talk\charaname.fmg` 6090 and 6091 = 巨人親衛隊長; `BossParam` 6090, 6091, 609000, 609010; `EnemyParam` 609000, 609010, 609100; behaviour rows (two sets for c6090, one for c6091); test script `ai009988_巨人親衛隊長テスト用`; no model | two stray rows (`EnemyAttackParam` 609000, `EnemyDefaultLogicParam` x3). The 10 m armoured stone giant object `o10_9200` (textures `BossStone`, `c6090`) is a map piece, not a character | the strongest NT-only boss. Two bodies, one name, like the Ruin Sentinels |
| **Old King 年老いた王 / King (present day) 王（現代）** c5140 / c5145 | `EnemyParam` 514000, 514500; AI `ai514000/514010` (年老いた王) and `ai514500/514510` (王（現代）＿インゲーム用, "in-game version") | Vendrick took c5146: retail's `ai514600/514610` carry the label **年老いた王**; the sound bank is still `frpg2_c5140.bnd`; lot 514500 still gives the **Soul of the King** (64210000) | Vendrick's earlier numbering, in two forms (old and present-day king). The time-travel dialogue (section 5) fits two eras, but nothing says so |
| **c6210, c6220, c6230** | `EnemyParam` 621000, 622000, 623000, with attack (and for two, common / move) rows; no AI script, no text | one stray attack row and one default-logic row each | **Named (2026-10-28)** by the developers' row names in the PC pre-release build (31 Mar 2014): 621000 = 達磨 "daruma"; 622000 = 腐食虫（本体） and 623000 = 腐食虫（子）, the main body and young of the "corrosive insect", whose outer shell is the Gutter ant 608000 (`gutter-ant-c6080.md`). The earlier reading "probably bosses" by id range was wrong for 6220 / 6230 |
| **c6240, c6245** | `EnemyParam` 624000, 624500 and behaviour rows; objects `o10_9230` (40 m rocky mass with a toothed mouth, textures `c6240_01-03`) and `o00_9111` (42 m emaciated giant, `BD_c6245`) | stray attack and default-logic rows | giant-scale, boss-sized. **Named 2026-10-28: both are the Giant King 巨人王** (624000 marked ※没 "scrapped"); see section 7 |
| **c6100** | `BossParam` 610000, `EnemyAttackParam` and `EnemyMoveParam` 610000 | the same two param rows (no boss row) | a boss slot with no character, model or text. **Named 2026-10-28:** 重騎馬兵：重騎士（生者） "heavy cavalry: heavy knight (living)", the Dragonrider's family (the Dragonrider is 重騎馬兵：兵士のみ "soldier only") |
| **Lich リッチ c5070, Wraith レイス c5080, c5130** | `EnemyParam` 507000, 508000, 513000 with behaviour rows (507 / 508 only); `ai508000` (レイス) | AI scripts only: `ai507000/507010` (リッチ) and `ai508000/508010/508030`, unreferenced in PS3 Scholar (`cut-content-pc.md`); no enemy rows. The Lich's script exists only in retail, its rows only in the NT | boss-range ids; whether they were bosses or ordinary enemies is not known. **2026-10-28:** retail does keep attack, logic, move, weapon, drop, bullet and damage rows for both, and the names show the Wraith's five spells and the Lich's "absorb into the body" grab; **c5130 is ヴァンクラッド王（過去） "King Vanclad (past)"** (section 7) |
| **Agdayne, boss form** c5065 | `ai506500` (アガドゥラン) and `ai506510` (インゲーム用), soul 60742000 アガドゥランのソウル, NPC menu 5060 墓守 | retail dropped both scripts; Agdayne is an NPC (grave-warden scripts `ai506000/506010` kept) and has no soul | `agdayne-darklurker.md` has the full case |
| **Silver Chariot, earlier numbering** c6180 | `ai618000/618010` シルバーチャリオット; `BossParam` 619100 / 619110; object `o23_9050` with `c6180` textures | `ai619100/619110` and c6191 (the Executioner's Chariot); no 618xxx scripts | the chariot was renumbered 6180 -> 6191; in the NT both exist |
| **Two-faced warriors 両面戦士** | `ai303000` (亡者), `ai303010`, `ai303020` (壁壊し, "wall breaker") | only `ai303300/303310` (ヘビ頭 "snake head" = the Flexile Sentry) | other non-snake-head variants of the warden |
| **Crab 宿り大蟹** | soul 60733000; also (found 2026-10-28) `EnemyAttackParam` 606000, behaviour rows and three weapon rows for c6060 | attack and weapon rows | **Named 2026-10-28:** c6060 = ヤドカリ城 "Hermit Crab Castle" (logic row 6060 ヤドカリ城（仮）, weapon row 60601000 ヤドカリ城（石化後）, "after petrification"). The crab tower in `unused-map-pieces.md` fits it, perhaps as the petrified form |

Test scripts that name bosses (NT only, `ai0099xx`): `ai009916` 上位ボステスト and `ai009917` 下位ボステスト ("upper / lower boss test"),
`ai009988` 巨人親衛隊長テスト用, `ai009989` BOSSテスト用, with the giant soldier tests `ai009900-902` (giant soldier, "rocket man",
wall-breaker), `ai009943/944` (last-sighting giant soldier) and `ai009987`. None is in retail.

## 2. The 27 NT boss souls (`ItemName` 60730000-60755000 and 50140000)

All have the placeholder description 仮テキスト / "No Text Settings" and no English name. No item lot in the NT holds any of them,
and retail has no 6073xxxx items (it renumbered its 34 boss souls 64000000-64330000).

| NT id | Japanese | Translation | Retail counterpart | Basis |
|---|---|---|---|---|
| 60730000 | 紅炎トカゲ | crimson-flame lizard | salamander family? | retail `ai601000` サラマンダー has a boss row but no soul; guess |
| 60731000 | 大蛙のデーモン | giant frog demon | Demon of Song (retail 唄うデーモン) | retail's script for him is labelled 美声カエル "sweet-voiced frog"; probable |
| 60732000 | 蜘蛛の女王 | spider queen | The Duke's Dear Freja (公のフレイディア) | retail AI label 蜘蛛の女王 |
| 60733000 | 宿り大蟹 | hermit crab | none (cut c6060 ヤドカリ城 "Hermit Crab Castle") | crab tower stand-in; developer row names for c6060 (section 7); a debug row tests "the hermit crab's claw" |
| 60734000 | 幼生ナーガ | larval naga | Mytha | retail's script for Mytha, `ai501000`, is labelled ナーガ, and the developers' soul list calls Mytha's soul ナーガのソウル (section 7); confirmed |
| 60735000 | 地竜の化身 | incarnation of the earth dragon | none | NT spell 33181000 仮・地竜の怒り "Earth Dragon's Wrath" is NT-only too |
| 60736000 | ヘル・スコーピオン | Hell Scorpion | none | retail has man-scorpions (Najka c5030, Tark c5020) |
| 60737000 | デス・ストーカー | Death Stalker | none | |
| 60738000 | 鏡の騎士 | mirror knight | Looking Glass Knight (same Japanese, soul 64180000) | exact name |
| 60739000 | カゲロウ竜 | mayfly / heat-haze dragon | none | |
| 60740000 | 岩の巨人カノッソフ | rock giant "Kanossof" | none | the stone giant c6090 is a candidate; nothing links them |
| 60741000 | 黒炎トカゲ | black-flame lizard | salamander family? | with 60730000, two lizard variants; guess |
| 60742000 | アガドゥラン | Agdayne | none (NPC only) | `agdayne-darklurker.md` |
| 60743000 | 古の神木 | ancient sacred tree | none (cut c6130 脳幹の神木 "Brainstem Sacred Tree") | NT item 50360000 神木の小枝 "sacred tree twig", the demon-forest line (section 5), and c6130's rows; a debug row tests "the Brainstem Sacred Tree's spear" (section 7) |
| 60744000 | 双頭竜 | two-headed dragon | none (cut c6140 双頭のワイバーン "Two-headed Wyvern") | c6140's attack and weapon rows (section 7); probable |
| 60745000 | 竜騎兵 | dragon cavalryman | Dragonrider (same Japanese, soul 64020000) | exact name |
| 60746000 | 成体ナーガ | adult naga | Mytha | as 60734000 |
| 60747000 | 貪りデーモン | Covetous Demon | Covetous Demon (soul 64090000) | exact name |
| 60748000 | 魔手 | demon hand | none | |
| 60749000 | 呪手 | cursed hand | none | |
| 60750000 | 魔女リディーレア | the witch Lidirea | none | the name resembles Leydia; no link |
| 60751000 | 幻影の騎士 | phantom knight | Imperious Knight? (AI ファントムナイト) | retail has the enemy, no soul |
| 60752000 | 幻影の術士 | phantom sorcerer | Leydia Pyromancer? (AI ファントムメイジ) | retail has the enemy, no soul |
| 60753000 | ヴァンクラッド王 | King Vanclad | Vendrick (soul 64210000 王のソウル; retail name ヴァンクラッド) | NT `charaname` 7020 ヴァンクラッド王 |
| 60754000 | 王妃デュナシャンドラ | Queen Dunashandra | Nashandra (soul 64250000) | NT `charaname` 7030 デュナシャンドラ王妃 |
| 60755000 | 処刑人 | executioner | the chariot's rider? | the chariot is 刑吏のチャリオット, a different word, but the developers' rows split the chariot into body, horse and **シルバーチャリオット：処刑人（亡者） "executioner (Hollow)"** (c6180; section 7); possible |
| 50140000 | 古の龍 | Ancient Dragon | Ancient Dragon (64230000 古の竜) | 龍 vs 竜 |

Tally: 4 exact or near-exact names (mirror knight, dragonrider, Covetous Demon, Ancient Dragon); 4 more tied to a retail boss by a
retail label or the NT's own name list (frog demon, spider queen, King Vanclad, Queen Dunashandra); 6 uncertain (two lizards,
two nagas, the phantom knight and sorcerer); **13 with no retail counterpart**. Update 2026-10-28: the two nagas are now
confirmed as Mytha. Three of the 13 now match a named cut body: crab = c6060, sacred tree = c6130, two-headed dragon =
c6140. The executioner may be the chariot's rider. Retail bosses with no NT soul at all include the
Pursuer, the Last Giant, the Flexile Sentry, the Ruin Sentinels, the Lost Sinner, the Skeleton Lords, the Rotten and Old Iron King.
(The NT does have parameter rows for several of these, table 3.)

## 3. Retail bosses under earlier names and forms in the NT

| Retail boss | NT name or form | Where |
|---|---|---|
| Vendrick | 年老いた王 "Old King" (514000) and 王（現代）"King (present day)" (514500); King Vanclad 7020 | AI labels, `EnemyParam`, `charaname`, soul |
| Nashandra | Queen Dunashandra 7030; "Shandra" in the castle dialogue (`m20_21` 70004480 / 70004510, retail: デュナシャンドラ) | `charaname`, talk |
| Demon of Song | 美声カエル "sweet-voiced frog" (also retail's script label); 大蛙のデーモン soul | AI, souls |
| Freja | 蜘蛛の女王 "spider queen" | AI, soul |
| The Rotten | ブッチャー "Butcher" (`ai326000`, `BossParam` 326000 / 326010) | AI, boss rows |
| Old Dragonslayer | オーンスタイン "Ornstein" (`ai625000`) | AI |
| Smelter Demon / Old Iron King | デーモンナイト "Demon Knight" (`ai305000`, in both builds) / 熔鉄デーモン "molten-iron demon" (`ai607000`) | AI |
| Looking Glass Knight | **"Mirror Knight"** in the NT's English `charaname` 504000 (added 2026-11-03); 鏡の戦士 "mirror warrior" in the Japanese one (retail 鏡の騎士); scripts ミラーナイト (`ai504000`); soul 鏡の騎士 (60738000). See `mirror-names.md` | `charaname`, AI, soul |
| Ruin Sentinels | 鏡の中の戦士 "warrior in the mirror" (`ai325000`, and the material entry for their model c3250); already in the JP test's base data of 23 Aug 2013, never "knight" (`mirror-names.md`) | AI, material |
| Last Giant | 新・巨人上級兵士 "new elite giant soldier" (`ai309600`, `BossParam` 309600) | AI, boss row |
| Dragonrider | 重騎馬兵：兵士のみ "heavy cavalry, soldier only" (`ai611000/611010`, `BossParam` 611000) | AI, boss row |
| Skeleton Lords | スケルトンの王 "Skeleton King"; English `charaname` 1540 "Skeleton Lord" (singular; retail "Skeleton Lords") | text |
| Royal Rat Vanguard | `BossParam` 226100 (NT only; retail has no boss row for it) | boss row |
| Salamander / Najka / spider queen variants | `BossParam` 601020, 503010, 603010 and the lower / upper scripts | boss rows |
| Executioner's Chariot | シルバーチャリオット "Silver Chariot" | AI, MSB region |

NT `BossParam` has 23 rows, retail 42; 12 NT keys are not in retail's (6090, 6091, 226100, 326010, 503010, 601020, 603010, 609000,
609010, 610000, 611000, 619100): six are lower / upper variants of the retail bosses above, the rest are the Giant Guard Captain
and the c6100 slot. The NT rows are mostly placeholders (the same value in every boss row), so their fields say little.
`BossBattleParam` has two NT rows, 1023000 and 1023010 (Skeleton Lord and the chariot), against 33 in retail.

## 4. Placements

- **NT generators (`GeneratorParam_m10_23_00_00`, 99 generators):** Skeleton Lords 154000 / 154010 / 154020 (one each) and Executioner's
  Chariot 619110. Retail's `m10_23` (105 generators) has the same four. **No other boss is placed anywhere in the NT.**
- **NT event script `event_m10_23_00_00`:** no boss states at all. Retail's version adds the boss fight, BGM start and stop,
  "boss defeated?" tests, `ボス戦_スケルトン貴族` and `ボス戦_シルバーチャリオッツ`. The NT had the bosses standing in the map but not yet
  scripted as fights.
- **Map content shared with retail (so not NT-only):** the boss-music regions 「シルバーチャリオット」 (-586, 47, 276) and 「スケルトン貴族」
  (-750, 28, 610), the three arena variants, the 闘技場 region (`Ghidra Project\export\win\s20, s21`). The NT map also has 30 generator
  positions with no generator ("removed enemies") and a warp to 【炭鉱の村】 "mining village", a map that is not in the demo.
- **NT-only bosses are never placed**: the c6090 stone-giant object, the c6240 and c6245 objects and the crab tower sit in the NT /
  retail layouts as unplaced set dressing; none is tied to a generator.

## 5. Dialogue (NT-only talk text, all Japanese unless noted)

Searched every NT FMG, comparing each line with retail's at the same id. NT-only lines that touch boss material:

| Where | Id | Text | Boss tie |
|---|---|---|---|
| `talk\m10_04` (the file with no language folder; Majula cat NPC 7770) | 77701100 / 110 | "The 'Old Forest' is creepy because the 'sacred tree branch' deep in it is connected to a demon's head. So the whole forest has turned into a demon." | the 古の神木 soul and the NT item 神木の小枝 |
| same | 77701300 / 310 | the 'Egg shrine' enshrines a 'dragon egg fossil', the one proof that the ancient dragons are no legend | Ancient Dragon; NT item 龍卵の化石 |
| same | 77701500 / 520 | the 'cave where souls gather' lets demons grow; the queen tried to hatch a 'dragon egg' there | Nashandra, the dragon egg |
| same | 77700610, 77701310 | "Compared with the age when the Ancient Dragon ruled the world..." | Ancient Dragon |
| same | 77701000-030 | humans and giants fought at the 'sea-side guard'; the giants were crude but their threat was no small thing | giants (Last Giant / Giant Lord / Guard Captain) |
| same | 77700700 | "The humans' King Vanclad had it secretly researched how to go back in time" | Vendrick |
| same (NPC 7220, Velderig, the time researcher) | 72200700-720 | dragons sometimes hold a gem; "it seems related to the souls the 'Ancient Dragon' produces", so he hunts the soul-hungry demons | Ancient Dragon soul |
| `talk\m10_04`, Japanese (NPC 7040, the ladder seller) | 70405330 / 340 | he has heard "something about a Rat King", and would gladly go down to the rats if that king is rich | the Rat King (NT `ai226500`, NPC menu 7560 ねずみの王; retail: Royal Rat Authority) |
| ~~`talk\m20_11` (Shrine of Amana)~~ | 76006000 / 76006010, 76016000 / 76016010 | **Not NT-only (corrected 2026-10-27).** The Milfanito's "Did you calm that cursed singing voice?... take this" is in retail word for word, renumbered to 76007000 / 76007010 and 76017000 / 76017010 (retail moved the death cries to 76005xxx / 76015xxx and "All we can do is sing" to 76006000 / 76016000). The id-by-id comparison missed the move | the Demon of Song's defeat (retail too) |
| ~~`talk\m20_11`~~ | 72600800 | **Corrected 2026-10-27:** NPC 7260 (Lonesome Gavlan) says カエル？ オマエ, which retail's English renders "You? Go home?" (帰る, "leave"), not "Frog? You". Retail has the line in all four of Gavlan's files (m10_17, m10_18, m10_25, m10_33). What is NT-only is that the NT also gives Gavlan's whole dialogue to the Shrine of Amana's file (see `shrine-of-amana-name.md`) | none |
| `talk\m20_21` | 70004480 / 70004510 | "If you go on, Shandra will set upon you." / "Please calm Shandra." (retail: デュナシャンドラ) | Nashandra's earlier name |
| `talk\charaname.fmg` | 6090, 6091 | 巨人親衛隊長 (the only NT health-bar name for an NT-only boss) | Giant Guard Captain |
| `talk\charaname.fmg` | 7020, 7030, 504000 | ヴァンクラッド王, デュナシャンドラ王妃, 鏡の戦士 (English 504000: "Mirror Knight") | Vendrick, Nashandra, Looking Glass Knight |
| `DetailedExplanation.fmg` | 40030000, 892002000-003 | descriptions of a ring and a four-piece "Drangleic traditional" armour that praise King Vanclad as a just king; retail rewrote them | Vendrick |

Also NT-only items that belong to boss ideas: **神木の小枝** "sacred tree twig" (50360000, key-item range), **仮・地竜の怒り** "(tentative)
Earth Dragon's Wrath" (spell 33181000), **龍卵の化石** "dragon-egg fossil" (50120000). None is in retail.

The 98 cut `m20_04` lines (girl and dealer) and the Shanalotte lines are not boss dialogue; see `cut-dialogue-translation.md`.

## 6. What retail kept of the NT's boss material

- **Stray rows** for c5070, c5080, c5130, c5140, c5145, c6090, c6091, c6210-c6245: `EnemyAttackParam` and `EnemyDefaultLogicParam`
  (`cnumtrace.py` output); c5145 also keeps behaviour rows and lot 514500 (Soul of the King).
- **Unreferenced AI scripts**: Lich and Wraith (above), plus the knights of life and death (`ai335000/336000`) and the rat king
  (`ai226500`) (`cut-content-pc.md`). The knights and the rat king have no NT enemy row either, so they are scripts only in both builds.
- **Objects**: `o10_9200`, `o00_9111`, `o10_9230` and the crab tower stand-in.
- **The Soul of the King lot keyed to the NT's King (present day)**, and the Old King label on Vendrick's script.
- Everything else (the souls, test scripts, `charaname` 6090 / 6091, the Agdayne scripts, the 年老いた王 / 王（現代） scripts, the older
  chariot scripts, the dialogue above) is gone from retail.

## 7. The developers' own names (PC pre-release, 2026-10-28)

The PC builds of 31 March 2014 (pre-release), 1.01 and 1.02 keep the developers' Japanese row names in their params
(UTF-16; `paramdef.py` reads them since 2026-10-28). That covers the regulation and the per-map params in `GameDataEbl`.
PS3 builds and later PC builds strip them. The rows themselves are the same in PS3 v1.00, so the names apply there.
(This is where the known "Jabba the Hutt" and "Silver Chariot" nicknames came from.)

### Every NT-only slot, named

Every slot below has param rows in both the NT and v1.00 (`presence.py`, scratch), and none has a model in any build.
The exceptions are c5146 and c6191, the retail bodies, listed for comparison.

| Body | Developers' name | Translation | Notes |
|---|---|---|---|
| c5050 | 恐獣 | dread beast | attack + weapon rows only |
| c5070 | リッチ | Lich | grab 150700410 "behaviour 14, throw: absorb into the body"; size class 超特大 "extra-large" |
| c5080 | レイス | Wraith | five spells: アビス・クラーケン "Abyss Kraken", インフェルニティ・ガン "Infernity Gun", イービル・ブラスト "Evil Blast", ダークシェイド "Dark Shade", スフィアボム "Sphere Bomb"; size class 超超特大 |
| c5130 | ヴァンクラッド王（過去） | King Vanclad (past) | the young Vendrick |
| c5140 | ヴァンクラッド王（現代）＿カットシーン用 | King Vanclad (present), for cutscenes | the NT's script for it, `ai514000`, is labelled 年老いた王 "Old King" |
| c5145 | ヴァンクラッド王（現代）＿インゲーム用 | King Vanclad (present), in-game | lot 514500 不死廟：王（現代）, "Undead Crypt: King (present)" |
| c5146 | 年老いた王：不死廟 | Old King: Undead Crypt | the retail Vendrick |
| c6040 | 蜘蛛の女王：左 | spider queen: left | Freja (c6030 蜘蛛の女王; weapon row 60310000 "on the cliff") |
| c6060 | ヤドカリ城 / ヤドカリ城（仮）/ ヤドカリ城（石化後） | Hermit Crab Castle / (provisional) / (after petrification) | the NT soul 宿り大蟹 |
| c6090 | 巨人族親衛隊長 / （石化後） | Giant-clan Guard Captain / (after petrification) | BossParts: head, **four arms** (front and back, left and right), two legs. The petrified form fits the stone-giant object `o10_9200` |
| c6091 | 巨人族親衛隊長（幻影） | Guard Captain (phantom) | not a second captain: a phantom double |
| c6100 | 重騎馬兵：重騎士（生者） | heavy cavalry: heavy knight (living) | Dragonrider family |
| c6130 | 脳幹の神木 | Brainstem Sacred Tree | the NT soul 古の神木; debug row 934 "effect check (Brainstem Sacred Tree's spear)" |
| c6140 | 双頭のワイバーン | Two-headed Wyvern | the NT soul 双頭竜 |
| c6150 | リヴァイアサン | Leviathan | |
| c6160 | ヴァンクラッド王（変形後） | King Vanclad (transformed) | a monstrous Vendrick |
| c6170 / c6180 / c6190 | シルバーチャリオット：本体 / 処刑人（亡者） / 馬（亡者） | chariot: main body / executioner (Hollow) / horse (Hollow) | the chariot as three parts before c6191 |
| c6191 | シルバーチャリオット：馬のみ / 合体 / 崖しがみつき | horse only / combined / clinging to the cliff | retail; the horse breathes from both heads (bullets 1619106xx-9xx); 161919000 "crashes into the gate and damages itself" |
| c6210 | 達磨 | daruma (a limbless figure) | weapon row 62000000 |
| c6220 / c6230 | 腐食虫（本体） / 腐食虫（子） | corrosive insect (main body / young) | the Gutter ant's other parts (`gutter-ant-c6080.md`) |
| c6240 | 巨人王　※没 | Giant King ※scrapped | object `o10_9230` (40 m rocky mass with a toothed mouth) |
| c6245 | 巨人王 | Giant King | object `o00_9111` (42 m emaciated giant) |

Agdayne's boss rows (c5065; `agdayne-darklurker.md`) are named too:
- EnemyParam 506500 墓守：不死廟 "grave warden: Undead Crypt";
- 506501 墓守：幽霊：汎用 "ghost";
- **506509 墓守：白霊：不死廟, "grave warden: white phantom: Undead Crypt"**;
- his spells トリプルスペル（闇術） "Triple Spell (hex)", three stages, and 黒い稲妻（闇術） "Black Lightning (hex)";
- grab 150650310 "parry-break stab".

**The Giant King:** the soul list below calls the Last Giant's soul 巨人王亡者 "Giant King (Hollow)" and the Giant
Lord's 巨人王生者 "Giant King (living)". The Giant Lord's bullets are named "c3097 巨人王生者". So the Giant King
began as its own boss slot (c6240, then c6245, each with a giant object) and was rebuilt in the giant family as c3096 /
c3097.

### The developers' boss names (`ItemUseCheckDialogParam`, the soul-use dialog rows)

Each retail soul row is named "<shipped name>（<working name>のソウル）". The working names:

| Soul | Boss | Working name |
|---|---|---|
| 64000000 | Pursuer | ベイグラント (Vagrant) |
| 64010000 | Last Giant | 巨人王亡者 (Giant King, Hollow) |
| 64020000 | Dragonrider | 重騎馬兵 (heavy cavalry) |
| 64030000 | Old Dragonslayer | オーンスタイン (Ornstein) |
| 64040000 | Flexile Sentry | 両面戦士：ヘビ頭 (two-faced warrior: snake head) |
| 64050000 | Ruin Sentinels | 鏡の中の戦士 (warrior in the mirror) |
| 64060000 | Lost Sinner | 忘れられた罪人 (Lost Sinner) |
| 64070000 | Executioner's Chariot | シルバーチャリオット (Silver Chariot) |
| 64080000 | Skeleton Lords | スケルトン貴族 (skeleton nobles) |
| 64090000 | Covetous Demon | ジャバザハット (Jabba the Hutt) |
| 64100000 | Mytha | ナーガ (Naga) |
| 64110000 | Smelter Demon | デーモンナイト (Demon Knight) |
| 64120000 | Old Iron King | 熔鉄デーモン (molten-iron demon) |
| 64130000 | Royal Rat Vanguard | 犬ネズミ：色② (dog-rat, colour 2) |
| 64140000 | The Rotten | ブッチャー (Butcher) |
| 64150000 | Najka | マンスコーピオン♀ (man-scorpion ♀) |
| 64160000 | Royal Rat Authority | 巨大ネズミ (giant rat) |
| 64170000 | Freja | 蜘蛛の女王 (spider queen) |
| 64180000 | Looking Glass Knight | ミラーナイト (Mirror Knight) |
| 64190000 | Demon of Song | 美声カエル (sweet-voiced frog) |
| 64200000 | Velstadt | 王妃の騎士B (queen's knight B) |
| 64210000 | Vendrick | 年老いた王 (Old King) |
| 64220000 | Guardian Dragon | ワイバーン (wyvern) |
| 64230000 | Ancient Dragon | 古の龍 (ancient dragon) |
| 64240000 | Giant Lord | 巨人王生者 (Giant King, living) |
| 64250000 | Nashandra | 王妃の鎧形態 (the queen's armoured form) |
| 64260000 | Throne Defender | 王妃の騎士A (queen's knight A) |
| 64270000 | Throne Watcher | 王妃の騎士C (queen's knight C) |
| 64280000 | Darklurker | アガドゥランのスタンド (Agdayne's Stand) |
| 64290000 | — | ★名称未設定（ガーゴイルのソウル）, "★ no name set (Gargoyle soul)": a Belfry Gargoyle soul never named |
| 64300000-64330000 | Old Witch / Old King / Old Dead One / Old Paledrake | イザリス / グウィン / ニト / シース (Izalith, Gwyn, Nito, Seath) |

So Velstadt and the two throne guardians were "the queen's knights A, B and C", and Nashandra's boss form is "the
queen's armoured form".

### Boss weapons on the debug list (`DbgPlayerStatusParam`)

Rows 900-949 are "bullet check" and "effect check" loadouts for testing weapon effects. Next to retail weapons (the
spider queen's curved sword, the queen's knights' weapons, the Mirror Knight's sword and shield, Ornstein's spear, King
"Vangrad"'s two swords), they test:
- 934 **脳幹の神木の槍** "the Brainstem Sacred Tree's spear";
- 935 **ヤドカリの爪** "the hermit crab's claw";
- 936 長老の鞭 "the elder's whip";
- 937 "the Silver Chariot's one-handed bow";
- 938 妖精の巫女の杖 "the fairy shrine maiden's staff" (the Staff of Amana);
- 931 家政婦のおたま "the housekeeper's ladle";
- the bell catalysts of the dragon, the queen and the hexer, and a shield-bell.

So the crab and the sacred tree were far enough along to have planned soul weapons.

## Not known or not checked

- The NT-only bosses' stats and attacks: the NT's `EnemyParam` and behaviour rows have a different layout from retail's (320 against
  356 bytes), the shared decoder does not fit, and nothing here was decoded beyond which rows exist.
- Whether c6210 (達磨), c5050, c6150, the Lich and the Wraith were bosses or ordinary enemies. Section 7 names them but
  doesn't say. The Lich and Wraith's damage-reaction size classes are 超特大 / 超超特大, "extra-large" (the Gutter ant's
  class too), which says they were big but not that they were bosses.
- Which giant the soul 岩の巨人カノッソフ "rock giant Kanossof" belongs to: the petrified Guard Captain and the
  Giant King are both candidates. The names don't settle it, and nothing names 地竜, Hell Scorpion, Death Stalker,
  the mayfly dragon, the hands or the witch Lidirea.
- Every "probable" and "guess" in tables 1 to 3 is an inference from names and ids; no file ties a soul to a body.
  (2026-11-04: the 27 souls have no NT `ItemParam` rows and no NT param refers to them. Soul pairs and the past /
  present question are in `two-soul-bosses.md`.)
- The JP network test was not scanned; the NT's other-language text is almost empty, so only Japanese was read.
- NT audio was not re-read (`network-test-music.md` has the `bossout` jingle and no boss tracks identified).

## Notes this draws on

`agdayne-darklurker.md` (soul list, Agdayne), `unused-map-pieces.md` (crab tower), `unused-models.md` (c6090, c6240, c6245, c7030,
c6180), `boss-region-names.md` (working names of retail bosses), `cut-content-scholar.md` and `cut-dialogue-translation.md` (soul list not revived;
Shandra), `cut-content-pc.md` / `cut_pc_ai.tsv` (unreferenced AI scripts), `unused-attacks-pc.md` (the attacks the NT's boss scripts
requested: Last Giant 15, Ruin Sentinel 16, Demon of Song 15 and 16), `nt-ai-labels-retail-enemies.md`, `developer-signs.md`,
`network-test-music.md`, and the NT session findings `Ghidra Project\export\win\s20` and `s21` (the Copse map).

Method (scratch scripts, session scratchpad, not kept): `gamedata.py` reads of the NT and retail archives; param ids and rows by
`paramdef` and raw words; AI labels from each `ai*.esd` header; `fmg.py` over every FMG of both builds with a same-id comparison;
`cnumtrace.py` for each character number; generators from `bosses_pc.generators` for both builds.
