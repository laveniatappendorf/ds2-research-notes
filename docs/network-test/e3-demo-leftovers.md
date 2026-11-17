# The E3 2013 demo build: what the Network Test keeps of it

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-11-03, revised the same day. The author asked whether the Network Test (NT) holds evidence of the convention
demo From Software showed before it (E3 June 2013 and later events), in which players fought the Mirror Knight and
saw small, slightly different parts of the Forest of Fallen Giants. The demo's map data is known not to be in the NT.
Builds: the JP NT's base data (NPJB90625 01.00, packed 23 Aug 2013; the US NT has the same), v1.00 (BLUS41045), the
PS3 patches and Scholar, PC 1.02 for retail sound banks (the same sizes as the PS3 disc's), current PC, and the
developer row names of the PC builds of 31 Mar to 28 Apr 2014 (`..\..\DS2 Vanilla PC\BUILDS.md`). Translations are
mine. Tools: `tools\e3_leftovers.py` (rows, banks, HP, classes, files; `builds` and `pc` modes) and `tools\fsb_music.py`.

**Revision.** The first version read every row named 抵抗の砦 or 岩峰城 as belonging to the E3 maps. The retail map list
`map\WorldMapList\WorldMapList.list` shows those are the developers' working names for the Forest of Fallen Giants and
Drangleic Castle. Only rows prefixed "E3" are the demo maps'. Sections 1 and 7 are corrected accordingly.

What the press saw at E3 2013, from the hands-on reports: four classes on the title screen (Warrior, Temple Knight,
Sorcerer, Dual Swordsman; Warrior and Sorcerer playable), a torch you carry instead of a shield, a ladder down into a
castle-like area, and the Mirror Knight, who planted his shield and brought AI knights out of it. The demo was offline,
so those were scripted spawns, not players.

## Short answer

- **The demo's two maps were E3 copies of two real areas.** The developers' rows call them **m20_62 E3抵抗の砦（現代）
  "E3 Fort of Resistance (present)"** and **m20_63 E3岩峰城（過去） "E3 Rock-peak Castle (past)"**. In the developers'
  map list, 抵抗の砦 is the Forest of Fallen Giants (m10_10, its past version m20_10) and 岩峰城 is the castle (m20_21,
  the map retail ships as Drangleic Castle, and an m10_21).
- **The NT has no map data for them, but it keeps:** both maps in its world map list (retail's no longer lists them),
  all 22 rows labelled "E3", their sound banks (no retail build has them), and placeholder Japanese names for their
  bonfires and areas.
- **The fort matches the Forest** beyond its name: the same ambience stream byte for byte, and bonfires 02 主塔 "keep"
  and 03 地割れ広場 "cracked-ground plaza", the Forest's names for what retail calls Cardinal Tower and Soldiers' Rest.
- **The castle matches Drangleic Castle:** its music is retail Drangleic Castle's first track byte for byte, and its
  bank holds two Mirror Knight sounds.
- **The NT's Mirror Knight is an earlier version** than retail's: a larger AI script, about a quarter of retail's HP,
  different movement, and summoning that spawns a servant on a timer rather than drawing for a player phantom.
- **The classes are the demo's four**: the NT's title menu has Warrior, Temple Knight, Sorcerer and Dual Swordsman.

## 1. The two E3 maps and their parent areas

**The developers' map list** (`WorldMapList.list`; names in v1.00 and PC Scholar, ids only in the NT):

| Map | Working name | Retail |
|---|---|---|
| m10_10 / m20_10 | 抵抗の砦 "Fort of Resistance" | Forest of Fallen Giants / the giant memories |
| m10_21 / m20_21 | 岩峰城 "Rock-peak Castle" | (no m10_21 map in v1.00) / Drangleic Castle |
| m20_62, m20_63 | listed in the NT only | none |

Other working names in the list: 化石の森 "petrified forest" (Things Betwixt), 廃墟の城 "ruined castle" (Lost Bastille),
拷問場 "torture ground" (Huntsman's Copse), 龍の研究所 "dragon laboratory" (Aldia's Keep), 水没城 "sunken castle"
(Undead Crypt). The m10 / m20 pairs are present and past versions of one area (a material is labelled "m20_10
抵抗の砦(過去)海用", Fort of Resistance (past), for the sea). So the E3 fort is a copy of the present Forest and the E3
castle a copy of the past castle, which is the one retail kept as Drangleic Castle.

**The 22 E3 rows** (all in the NT and in v1.00):

| | m20_62 E3 Fort of Resistance (present) | m20_63 E3 Rock-peak Castle (past) |
|---|---|---|
| Area | `AreaParam` / `NetworkAreaParam` 20620000, `PlayerLight` 206200 | `AreaParam` / `NetworkAreaParam` 20630000, `PlayerLight` 206300 |
| Bonfires | 62650 篝火01_ボス撃破出現 "bonfire 01, appears when the boss is beaten"; 62651 篝火02_主塔 "02, keep"; 62652 篝火03_地割れ広場 "03, cracked-ground plaza"; 62653 篝火04_地下ボス部屋後 "04, after the underground boss room"; key prompts "warp" and "light" | none |
| Boss door | | white door 206300 E3岩峰城_ボス白扉(入口) "boss fog door (entrance)" and its two key prompts |
| Item | | lot 120635500 【宝死体】E3スタート地点からギリギリ見える "corpse, only just visible from the E3 start point" |

Plus six test rows `MapObjectInstanceTypeParam` 29-36 marked 【テスト】【E3対応】 "[test][for E3]": 龍の研究所：巨人属性
"dragon laboratory: giant attribute", "breaks in one hit: rolling / ornament", and three スタチューナイト "statue knight"
rows, one 清水さん仮 "Shimizu's provisional".

**The working-name rows** (249 more, named 抵抗の砦 or 岩峰城 without "E3") belong to the Forest and the castle
themselves, so they show what the areas held around the demo's time, not necessarily what the E3 copies held. Among them:
- **Fort of Resistance:** hollow lower-rank soldiers "(present)" in five loadouts; a stone guardian; the spell seller
  "(Fort of Resistance, present)", Carhillion, whom retail moved to No-man's Wharf; 129 objects of the Forest's `o10`
  series, among them 【汎用】抵抗の砦：親衛隊長_石化 "[generic] Fort of Resistance: the Guard Captain, petrified" (four
  versions, `o00_4900`-`4903`), the phantom Giant Guard Captain (`o10_9200`) and the Giant King (`o10_9230`). The NT's AI
  debug states also name the past fort: 上位ステート：抵抗の砦（過去）_王国兵士用 / _巨人下級兵士 "Fort of Resistance (past):
  for kingdom soldiers / giant lower soldiers".
- **Rock-peak Castle:** a stone guardian, a door-haunter, hollow jailers, hollow demon armour with katana and bow; 40
  objects of the castle's `o21` series, among them the castle for distant views (`o00_9110`), the golem gate, a throne
  and the queen's chair, a Dullahan-head puzzle (one right head, four wrong), and **the pillar the Mirror Knight can
  break** (`o21_2000`) and **a mirror** (`o21_9051`).

The Giant Guard Captain (`nt-bosses-not-in-retail.md`, the NT's one named boss retail lacks) was therefore a Forest of
Fallen Giants boss. Whether the E3 fort used him (its bonfire 01 appears after a boss) is not recorded.

## 2. What the NT carries

- **The map list:** the NT's `WorldMapList.list` ends with m99_93, **m20_62 and m20_63**; v1.00's drops all three (and
  adds m20_26 and the names above).
- **The 22 E3 rows**, and 199 of the 249 working-name rows (retail adds the rest).
- **Placeholder names:** Japanese `BonfireName` 62650-62653 read "Bonfire" and `MapName` 20620000 / 20630000 read "Map",
  in the NT and still in v1.00 (section 6).
- **Sound banks** (`sound\`): `frpg2_m2062` / `m2063` (map sounds), `sm2062` / `sm2063` (streams) and `xm2062` / `xm2063`
  (ambience). Neither PC 1.02 nor the US disc has any `2062` / `2063` bank, nor do the current PC installs.
  - `m2062`: footstep, door and creak sounds plus `o102250000` and `o102370000`, the sounds of objects `o10_2250` and
    `o10_2370`, both placed in retail's Forest.
  - `sm2063`: ambience `a206310001` / `002`, the music `m206300001`, and **`c504007010` / `c504007777`, Mirror Knight sounds**
    (his own bank `frpg2_c5040` has 19 others).
- **Object models:** the NT ships the four petrified Guard Captain models and the distant-view castle, none placed in the
  Copse. They are the Forest's and the castle's objects (above), not specific to the E3 maps.
- **The Mirror Knight's second servant, c7310:** model, textures and timing files (TAE). Its model and textures differ
  from retail's (804,064 / 1,549,184 bytes against 925,312 / 875,392). Its animation entry is empty in both archives.

## 3. The fort and the Forest of Fallen Giants

- **Name:** E3 Fort of Resistance (present) = the E3 copy of m10_10, the present Forest.
- **Ambience:** the fort's `y206200000` stream is byte-identical to the Forest's `xm1010` stream (and m10_30's).
- **Bonfires:** the Forest's developer bonfire names are 10655 篝火02_主塔, 10660 篝火03_地割れ広場, 10670 篝火05 "the
  riverbank after the link to Majula" and 10675 篝火06 "after the King's door"; retail names them Cardinal Tower (Japanese
  still 主塔), Soldiers' Rest, The Crestfallen's Retreat and The Place Unbeknownst. The fort's bonfires 02 and 03 have the
  same numbers and descriptions as Cardinal Tower and Soldiers' Rest. Its 01 and 04 have no Forest counterpart in the rows.
- **Music:** the NT's Forest bank `sm1010` has two tracks. The first is byte-identical to retail's; the second is the same
  piece in another cut: not identical, about 4 % longer, loudness-envelope match 0.82 (different pieces score 0.05-0.3).
- **NPCs:** the NT's English `talk\m10_10` (Pate "Pait", the bag-and-belt craftswoman Chloanne, Carhillion "Karion";
  `nt-draft-talk-folder.md`) has the Forest cast the NT's params set up: Carhillion's spawn row is the Fort of
  Resistance's spell seller, and Pate (744000) and Chloanne (762000, 762010) have spawn rows only the NT has. The NT does
  not need Forest text for the Copse, and apart from the one-line system file m90_00 no other map it doesn't play has
  English dialogue, so this was probably translated for the English-language demo. The 2014 PC rows still call Chloanne
  カバン屋, "the bag shop".
- The NT's Japanese `bloodmes\m10_10` hint 『敵の背後でR1：致命の一撃』 "R1 behind an enemy: critical hit", which names
  the button in plain text (`developer-messages-nt-vs-retail.md`), fits a show-floor demo too.

## 4. The castle and the Mirror Knight

- **Name:** E3 Rock-peak Castle (past) = the E3 copy of m20_21, the past castle, which retail ships as Drangleic Castle.
- **Music:** `sm2063`'s `m206300001` (2:44) is byte-identical to retail Drangleic Castle's `m202100001`. The castle's
  only boss in the demo was the Mirror Knight, so this is very likely his theme; I have not traced which retail fight
  plays it. The NT's own Drangleic Castle track (`sm2021`, the same audio as the unknown map m10_22's `sm1022`, 3:03) is not
  byte-identical to any retail track. Its best envelope match is 0.63, with retail's `m102800001` (4:34) in the same
  bank: below the 0.8-1.0 of same-piece pairs here, above unrelated pieces (0.05-0.12). Unconfirmed; listen to compare.
- **Objects:** 17 of the 40 castle objects are placed in retail's Drangleic Castle, among them **the pillar the Mirror
  Knight can break** (`o21_2000`), the kingdom flag, a pit-trap floor, a floor lever and two masks (retail's castle has a
  仮面の間 "hall of masks" developer sign, `developer-signs.md`). The Dullahan-head puzzle, the throne and the queen's
  chair are placed nowhere.

**The NT's Mirror Knight against retail's:**

| | Network Test | v1.00 |
|---|---|---|
| HP (`EnemyParam` +0x28) | 1,440; servants 225; Ruin Sentinel 560 | 6,540; servants 1,090; Ruin Sentinel 2,330 |
| HP baselines | Skeleton Lord 605, goblin 295 | 780, 390 |
| Movement (`EnemyMoveParam` 504000) | jump height 6.0, jump length 7-7 | 8.0, 6-8 (23 fields differ) |
| Attacks (`EnemyAttackParam`) | identical | |
| Weapons (`EnemyEquipWeaponParam` 50400000) | one field 65535 | 5396 |
| Boss spawn (`BossEnemyGenerateParam` 504000) | generator 5002 | 5001 |
| Projectiles (`EnemyBulletParam`) | his Thunder Break rows 150400820-873 and the mirror's glass shards 150400930 | the same rows (later layout) |
| Boss rows | no `BossParam` or `BossBattleParam` row | `BossParam` 504000, `BossBattleParam` 1021010 |
| Main AI `ai504000` | 74,384 bytes | 63,734 bytes |
| Upper AI `ai504010` | 5,690 bytes | 5,692 bytes |
| Phantom type | `ChrNetworkPhantomParam` 14 (2014 name ミラーナイト霊 "Mirror Knight phantom") | the same |

Against a goblin or Skeleton Lord the NT's HP is about 0.75x retail's; the Mirror Knight family's is about 0.22x.

**AI.** State names only in the NT's main script: lightning attacks named 雷撃回転攻撃 "lightning spin attack" and 雷撃縦衝撃波
"lightning vertical shockwave" (retail calls them コンボ7：サンダーブレイク "combo 7: Thunder Break"), 下位ステート：雷撃攻撃
"lower state: lightning attack", jump slashes for 8, 10 and 12 m, a BOSS用（STEP2） jump attack, reactions to the player
using an item (サンブレ, short for Thunder Break, a jump or a thrust), and the generic ladder-climbing states. The NT's
upper script watches the mirror with ミラーリング監視：開始 / 生成 / クールタイム / 待機 / 停止中 "start / spawn / cooldown /
wait / stopped". Retail's replaces that with 召喚抽選開始 / ミラーナイト霊生成 / 召喚抽選終了 "summon draw start / Mirror Knight
phantom spawn / summon draw end" and checks "is the Mirror Knight phantom ready" and "can the Mirror Knight NPC be
summoned". Retail also adds オンライン敵生成 "online enemy spawning" states, and gives 417 hollow scripts shared
【ミラーナイト】召還抽選操作 "Mirror Knight summon draw" states the NT doesn't have. The NT's timed spawn fits the demo's offline AI knights.

The NT also has the test script `ai009983` (下位：吉田テスト用, "for Yoshida's tests") with 【テスト】ミラーナイト and
【テスト】ミラーナイトバレットテスト "Mirror Knight bullet test" states, gone from retail; and its English boss name is
"Mirror Knight" (`mirror-names.md`).

## 5. The classes

NT `TitleMenu` 1000020001-004 (and 2000020001-004, `ingamemenu` 2401610-2401640): Warrior, Temple Knight, Sorcerer,
Dual Swordsman, exactly the demo's four. Japanese: 戦士, 神殿騎士, 魔術師, 双剣士.

## 6. Names and bonfire numbers in the release builds

- **Developer row names** (the source of "E3 Fort of Resistance (present)" and every name above) are in no PS3 build.
  PS3 params carry no row names, except PS3 1.02, which ships them in 15 balance params (weapons, damage, items,
  enemies, armour, spells; none of the E3 or working-name rows). **They did ship on PC:** the launch build 1.01 (exe 22 Apr
  2014) and the 1.02 patch (28 Apr) carry all 271 names, identical to the 31 Mar pre-release. The regulation-only update
  of 2 Jun 2014 (regulation 1.06) stripped them: none of the 271 is named there, nor any row of the four largest
  params, and no later PC build has them.
- **The NT has no row names at all**, so it has none of these names: only the E3 rows' ids, the map-list ids and the
  placeholder text. The builds with the "E3" names are the PC pre-release of 31 Mar 2014 and the two public PC releases
  of April 2014, 1.01 and 1.02 (all 33,099 pre-release names, plus 187 more in 1.01 / 1.02).
- **Bonfire descriptions** come from two places:
  - the `MapObjectBonfireParam` row names (all 64 bonfires, the four E3 ones included): the three PC builds above only;
  - the event scripts' debug files (`ezstate\event_<map>.edd`), whose state names number and describe each bonfire as
    再生の種火NN_<description>, e.g. m10_10's 再生の種火01_主塔 / 02_地割れ広場 / 04_マデューラとの接続後の川岸 (numbered per
    file, not as in the params). These ship in every retail build: 59 in 23 maps in v1.00 (and the Japan disc), 78 in 26
    maps in PS3 Scholar and PC Scholar (adding the DLC maps), and in current PC vanilla. The NT's event debug files have
    none, and the E3 maps, having no event files, appear in none.
- **Working map names** (抵抗の砦, 岩峰城 and the rest, section 1) are in `WorldMapList.list` in every retail build:
  34 names in v1.00 and the Japan disc, 38 in PS3 Scholar, current PC vanilla and PC Scholar (the four DLC maps:
  地下マップ "underground map", 灰マップ "ash map", 氷マップ "ice map", ヴァンクラッドの記憶 "Vanclad's memory"). The NT's
  list has the ids only. Outside that file, a working name ships only in one material label (`m20_10 抵抗の砦(過去)海用`,
  NT to PC Scholar) and in two NT AI state names (section 9).
- **Text a player could see:** none names the E3 maps. The Japanese `BonfireName` slots 62650-62653 hold the placeholder
  "Bonfire" and `MapName` 20620000 / 20630000 the placeholder "Map", in the NT and in v1.00. English has no entries for
  them; PC has no Japanese text.
- **The map list:** v1.00's and PC Scholar's `WorldMapList.list` name the Forest 抵抗の砦 and the castle 岩峰城, but no
  longer list m20_62 or m20_63.
- **Bonfire numbers.** A bonfire's id is its map number, 6, then a two-digit slot: the Forest's are 10655-10675, the
  Copse's 23650-23665, the E3 fort's 62650-62653. The four E3 rows are in every build from the NT to current PC Scholar,
  unchanged since v1.00: area 20620000, no lit flag (0), the ascetic and effigy flags set, and menu sort ids 252-255, the
  last four slots (the Forest's are 10-14, the Copse's 50-54). The NT's rows are in its older 32-byte layout.

## 7. The rows build by build

`e3_leftovers.py builds` counts the 271 rows named E3, 抵抗の砦 or 岩峰城 (22 of them E3):

| Build | Of the 271 | Notes |
|---|---|---|
| JP NT 01.00, US NT, JP NT 01.02 | 221 (all 22 E3) | the 50 missing are working-name rows in `EnemyDefaultLogicParam` (12; the param is new after the NT), `MapObjectParam` (19), `MapObjectGDParam` (15), `MapObjectDefaultInstanceParam` (4) |
| US v1.00, Asia disc, Japan disc | **271** | identical in all three |
| PS3 1.01, 1.02 | **271** | every row byte-identical to v1.00 |
| PS3 1.03 | 270 | 1.03 drops the old item-lot params wholesale (`ItemLotParam`, `ItemLotParam2`, `ItemLotTicketParam`, `ItemLotWeightParam`; `..\..\DS2 Vanilla Patches\findings\1.01-to-1.03.md`), and the E3 corpse lot 120635500 goes with them. Nothing else changes |
| PS3 1.08, 1.09 | 270 | 163 map-object rows differ from v1.00, but only in one field that changed in 1,353 of 1,435 `MapObjectParam` rows: a global change |
| PS3 1.10, PS3 Scholar | 270 | the old `ItemLotParam` is back (with the E3 lot), but `EnemyAttackParam` is not in the regulation, so Carhillion's Fort of Resistance row 766010 is missing |
| PC pre-release, 1.01, 1.02, reg 1.06, 1.03 | 271 | |
| PC 1.12 / PC Scholar | 270 / 269 | `EnemyAttackParam` absent; PC Scholar also lacks `ItemLotParam` |

Between the NT and v1.00 the shared rows were edited too (843 of the 903 shared `MapObjectParam` rows differ, mostly
in one field), so that is also a global change. The object models stay: patches add or replace archive files and
never delete them (`..\..\DS2 Network Test\tools\games.py`), and 1.01-1.03 ship no sound files. The Japan and Asia
discs' sound folders are not copied here, so their banks were not checked.

**Current PC** (`e3_leftovers.py pc`; vanilla 1.12 of Nov 2022 and Scholar 1.0.3.0 of Sep 2022, both the latest):

- **Rows.** PC 1.12 keeps 270 and PC Scholar 269. Compared field by field, PC 1.12 matches PS3 1.09 on 263 rows and
  PC Scholar matches PS3 Scholar on 260. The few that differ are two area rows, four ground-material rows and, in
  Scholar, three object-instance rows (their projectile ids).
- **Models.** PC 1.12 still ships models for at least 159 of the 169 named map objects (named entries only), including
  the petrified Guard Captain (`o00_4900`-`4903`), the phantom Guard Captain and the Giant King (`o10_9200` / `9230`),
  and the distant castle (`o00_9110`). PC Scholar ships 64; every entry in its index is named, so the other 105 are
  gone, among them all of those. Both keep the Mirror Knight's pillar and mirror (`o21_2000`, `o21_9051`).
- **Sound.** Neither install has an m20_62 / m20_63 bank.

## 8. Not in the NT

No map, collision, navigation mesh, MSB or event script for m20_62 or m20_63, and no Mirror Knight model. The NT's
program never names either map.

## 9. Past and present: the other time-labelled areas (added 2026-11-03)

The author asked which other maps carry a past or present label (time travel was a known development plan). Sources: the
2014 PC developer row names (PC 1.02: 215 rows with 過去 "past", 現代 "present" or 未来 "future"; 現在 "current" only
in unrelated rows such as "current HP"), the map list's numbering, and the shipped files (one material label, two NT AI
states).

**The map numbering.** In `WorldMapList.list` the areas come in m10 / m20 pairs with one working name, and every camera
row for an m20 map is labelled past: 2010000 巨人の記憶（過去）, 2011000 アマナの祭壇（過去）, 2021000 王城ドラングレイグ（過去）,
2024000 不死廟（過去）. So m10 is the present and m20 the past:

| Working name | Present | Past | Retail ships |
|---|---|---|---|
| 抵抗の砦 Fort of Resistance | m10_10 | m20_10 | both: the Forest of Fallen Giants and its giant memories |
| 妖精のほとり "the fairy waterside" | m10_11 | m20_11 | past only: the Shrine of Amana |
| 龍の研究所 "dragon laboratory" | m10_15 | m20_15 | present only: Aldia's Keep |
| 岩峰城 Rock-peak Castle | m10_21 | m20_21 | past only: Drangleic Castle |
| 龍の祠祭場 "dragon shrine festival ground" | m10_27 | m20_27 | present only: Dragon Aerie / Shrine |
| 龍の住処 "dragon's lair" | | m20_03 (and m40_03) | m40_03 only, the Dark Chasm of Old |
| 水没城 "sunken castle" | | m20_24 | the Undead Crypt (labelled past as 不死廟（過去）) |
| E3 copies | m20_62 E3 Fort of Resistance (present) | m20_63 E3 Rock-peak Castle (past) | none |

**Area labels in the row names:**

| Area | Labels and rows |
|---|---|
| Fort of Resistance (Forest) | (present): Carhillion as spell seller, hollow soldiers' logic rows; (past): debug loadout 300114000, material `m20_10 抵抗の砦(過去)海用` "for the sea" (shipped NT to PC Scholar), and, in the NT only, AI states 上位ステート：抵抗の砦（過去）_王国兵士用 / _巨人下級兵士 "Fort of Resistance (past): for kingdom soldiers / giant lower soldiers" |
| Rock-peak Castle | (present): debug loadout 300112000 岩峰城（現代）; (past): Drangleic Castle's camera row 2021000, light 202100 ■王城ドラングレイグ（過去）：屋外明るい空間汎用 "Drangleic Castle (past): bright outdoor space" |
| Shrine of Amana | (past): its four bonfires 11650-11670 アマナの祭壇_過去_篝火01-05, camera row 2011000, key prompt 20119000 妖精のほとり過去_祭壇お供え "fairy waterside, past: altar offering" |
| Undead Crypt / sunken castle | 水没城：半身（過去） "half-submerged (past)" (stone guardian 213006, door-haunter 216001) against 水没城：満水（現代） "full of water (present)" (debug loadout 300110000; stone guardian 213009 満水) and 全貌 "the whole" (Dark Wraith 136001, door-haunter 216003) |
| Majula | (present): 33 rows, the shops and NPCs "in Majula (present)"; (past): debug loadout マデューラ（過去） |
| Dragon's lair | (present): hollows in sick-person's clothes 147000, debug loadout 300102000; (past): one debug loadout; its objects (`o03` series) include 古龍の竜眼のまぶた "the Ancient Dragon's eyelid" and a ring "for an in-engine cutscene" |
| 霧の城下町 "the misty castle town" | (present): a Dark Wizard (118110); (past): a Dark Wraith (136000). Not in the map list; no retail area is known by this name |
| 生きている森 / 生きてる森 "the living forest" | (past): a stone guardian (213005), debug loadout 300113000; also 前半 / 後半 "first / second half" rows (a giant basilisk 222001). Not in the map list; the NT's dialogue about a forest that became a demon (`nt-bosses-not-in-retail.md` section 5) may be the same place, unconfirmed. **2026-11-04: probably the Shaded Woods.** Basilisk 222001 is named 虚ろな影の森 in `EnemyParam` and 生きている森（後半） in `EnemyDefaultLogicParam`, and v1.00 places it in m10_32 (`two-soul-bosses.md` section 3) |

**Characters and the player:**
- King Vanclad (Vendrick) in both: ヴァンクラッド王（過去） and （現代）, each with cutscene rows, and armour 5130
  【ヴァングラット王（過去）】 "King Vangrad (past)" (head to legs).
- The queen: 王妃（過去） / （現代）, both with cutscene rows.
- 王国軍師 "the kingdom's strategist": armour 7210 【王国軍師（過去）】 and behaviour rows （現代）.
- Pate: armour 7440 （作成中）【パッチ（現代）】 "(in progress) Pate (present)".
- The shops: armour shop, spell sellers 1 & 4 and 2 & 3, the general store, the materials shop and weapon smith 1, each
  "(present)".
- **The player in two times:** 800000 現在のプレイヤー（ポリ劇用） "the present player (for in-engine cutscenes)" and 800100
  未来のプレイヤー "the player of the future", with behaviour, logic and weapon rows.

These are all row names, so they exist only in the three 2014 PC builds; the rows themselves are in retail.

## Open points

- Which retail fight plays `m202100001`, from the event or sound-event data.
- What m10_22 was: an NT-only music and ambience bank whose track is also the NT's Drangleic Castle track.
- What m10_21, the present Rock-peak Castle, was: listed in the map list and in the NT's talk files, but no map in v1.00.
- Decoded durations (FFmpeg via PyAV) differ from `network-test-music.md`'s frame counts for some tracks: `sm2063`
  decodes to 2:44 there listed 4:22 (retail's identical bytes also decode to 2:44); the two `sm1010` tracks decode to 2:55 /
  2:24, listed 1:27 / 1:11. Not resolved.

## Reproduce

From this folder's parent:
- `uv run --no-project --python 3.12 python -I -X utf8 tools\e3_leftovers.py` (about 5 minutes); add `builds` or `pc` for
  section 7;
- `uv run --no-project --python 3.12 --with numpy --with av python -X utf8 tools\fsb_music.py compare <NT bank> <retail bank>`
  for the envelope matches (`same` for byte identity, `names` for sample names).
- The map list: `map\WorldMapList\WorldMapList.list` (Shift-JIS text) via `gamedata.py x WorldMapList <dir>`.

Sources for the demo: [Siliconera's hands-on](https://www.siliconera.com/dark-souls-ii-hands-on-the-fight-with-the-mirror-knight/),
[Entertainium](https://entertainium.co/2013/06/17/e3-2013-dark-souls-2-hands-on-preview/),
[Co-Optimus](https://co-optimus.com/article/10385/e3-2013-dark-souls-2-hands-on-impressions.html).
