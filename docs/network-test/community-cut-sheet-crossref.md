# The community cut-content sheet, checked against our findings (2026-10-28)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author asked for a cross-reference of a community Google Sheet ("Untitled spreadsheet",
`docs.google.com/spreadsheets/d/1_imlYCft7skFb4Asez1I9tV5iX-eOlx5wQy0akehYZM`, read 2026-10-28, not edited) against
what we know. The sheet lists 56 cut enemies, bosses and characters by id, Japanese name, source param and notes, then
the Network Test's 26 boss souls. Translations of new Japanese below are mine.

For the boss slots, the full table of developer names is [Network Test bosses not in retail](nt-bosses-not-in-retail.md) section 7, written the same day
from the same PC rows (its translations differ slightly: "dread beast", "Hermit Crab Castle", "Brainstem Sacred
Tree"). This file follows the sheet's rows and adds the characters, files and text that section does not cover.

## Short answer

- **The sheet's Japanese names are the developers' own param row names.** Every one matches a row name in the PC
  pre-release build of 31 March 2014 (`DS2 Vanilla PC\2014-03-31\regulation`, read with `paramdef.py`), which is
  also where SmithBox's community row names come from. So the names are sound. Its "Source" column names one param
  per row, but most ids have rows in several.
- **Most of its claims hold**, and five were new to our notes (section 2). They are three cut cutscenes for the Lich and Wraith,
  one for the Giant Guard Captain and one with Cale (all vanilla PC only), the flower mace `WP_2450_M`, and The
  Imperfect's `c5050` mesh. All checked and confirmed.
- **Four nameless rows on the sheet are ids we can name.** The Network Test's speaker list (`talk\charaname.fmg`) gives
  **7220 = Velderick** (ベルデリッグ; the time scholar of `nt-draft-talk-folder.md`), **7400 = Lanafast**,
  **7550 = Wickley** and **7780 = Colmcille**. It also names 7210 Wellager (ベラガー) and 7410 Saulden (ソダン). The
  sheet lists Velderick, Lanafast, Wickley and Colmcille as separate, id-less names.
- **Six of its readings need correcting** (section 3). For example, 7410 *is* Saulden's own id, not an early version of him. The boss
  Vendrick is `c5146` (`c6840` is the Scholar DLC memory figure). And the three sisters 7570-7590 already carry
  the DLC names on the v1.00 disc of January 2014.
- **The developers' names add a lot the sheet leaves out**: the Giant Guard Captain's four arms and petrified form, a
  petrified hermit crab, "Giant King ※没 (cut)", a child Herald placed in Aldia's Keep, and a whole set of
  past / present pairs (section 4).

## 1. Row by row

"NT" = Network Test (Aug 2013), "v1.00" = PS3 retail disc, "PC 3/14" = the PC pre-release build (31 Mar 2014), "PC 1.02"
= vanilla PC of 28 Apr 2014. Verdict: **OK** = the sheet is right as far as we can check; **+** = we add something;
**fix** = we read the data differently.

### Enemies

| Sheet | What we have | Verdict |
|---|---|---|
| 1190 獅子黒魔道士（生者） Lion Black Mage | NT and v1.00 both have `EnemyParam 119000` and AI `ai119000/119010` with this label. Armour rows 2219010x are named 【獅子黒魔道士】 (Lion Mage set). PC 3/14 names its two enemy rows **獅子黒魔道士：こけし背負いデモ** "lion black mage: demo, carrying a kokeshi doll" and **119001 炎を吐く鎧の騎士デモ** "fire-breathing armoured knight demo". 119001 was added to PS3 in 1.03 ([patches 1.01 to 1.03](../builds/1.01-to-1.03.md)). Scholar and PC place neither, and PC ships no `ai119000` | OK +: in the developers' naming these are **demo** rows |
| 1220 バーニスの騎士 Knight of Berenike | NT: AI `ai122000/122010` and four `EnemyParam` rows, one per weapon (greatsword, lance, one-handed hammer, crossbow). PC 3/14 adds colour-variant logic rows 1221x0 (色②). Armour rows named 【バーニスの騎士】 = Steel set | OK + (AI and rows, not just sound) |
| 1360 ダークレイス Darkwraith | NT `EnemyParam` 136000 / 136001 / 136020. Logic rows in PC 3/14 name two map variants: 霧の城下町（過去）"misty castle town (past)" and 水没城（全貌）"flooded castle (whole)". Dark set rows named 【ダークレイス】 | OK +: maps |
| 2180-2182 人間性（敵）Humanity L / M / S | NT: AI `ai218000-218210` 人間性（大／中／小）and `EnemyParam` rows. PC 3/14: small = 水没場：半身（過去）"flooded place, half body (past)" / 水没場：全貌 "flooded place, whole" | OK + |
| 2190 ベイグラント鷹 Vagrant's Hawk | NT `EnemyParam 219000`. The giant animated eagle `o00_6090` (textures `c2190`) is in both the NT and retail. The Bastille's developer sign reads ベイグラント鷹タクシー "Vagrant hawk taxi". The Pursuer's boss region is ベイグラント ([hidden map parts](../unused/hidden-map-parts.md), [boss working names](boss-region-names.md)) | OK +: the hawk survives as an object |
| 2250 光虫 Bright Bug | NT: AI `ai225000/225010` + test `ai225090`, `EnemyParam 225000`. PC 3/14 `EnemyCommonParam 225000` = **光虫（NPCで流用）"bright bug (reused for NPCs)"**, which confirms the sheet's empty NPC model (see [Dyna & Tillo's invisible body](sparkling-girl-dyna-tillo.md)). The Amana signs call the fireflies 光虫 (`developer-signs.md`). **`WP_2450_M` is in PC 1.02 only** (not in any PS3 build or PC Scholar); its mesh is named `WP_c2250_M`, as the sheet says | OK, confirmed |
| 2320* Iron Warrior precursor | Scholar `c6500.texbnd` holds `WP_2320_M` textures, as the sheet says. No 2320 rows anywhere. The PC pre-release already has draft Iron Warrior AI ([earliest PC builds](../builds/early-pc-builds.md)) | OK (concept art not checked) |

### Bosses

| Sheet | What we have | Verdict |
|---|---|---|
| 5050 恐獣 Terrifying Beast | Attack and weapon rows only, in NT, v1.00 and PC. Scholar's `c6830` (AI `ai683000` なりそこない "the failed one") has a mesh named **`c5050`**. `o99_0037` (10.5 m, `c5050`) is PC Scholar only ([cut content in PC Scholar](cut-content-pc.md)) | OK, confirmed |
| 5070 リッチ Lich | NT `EnemyParam 507000`. Retail-only AI `ai507000/507010`, gone on PC ([Network Test bosses not in retail](nt-bosses-not-in-retail.md)). Item lot rows 50700000. Olenford's Staff = オランフォードの杖（DS：リッチ：片手杖触媒（奇蹟闇術））. **"DS:" tags soul-trade weapons** (the Chariot Lance and Nashandra's Chime / Bow of Want carry it too), so the Lich was meant to have a tradeable soul | OK + |
| 5080 レイス Wraith | NT `EnemyParam 508000` and AI `ai508000`. **The three cutscenes exist in PC 1.02 only.** `scn102530`, `scn102540` and `scn102560` all cast `c5070` and `c5080`; scn1025xx = map m10_25 (The Gutter's map). Effect labels confirmed: ミイラから異質な妖気, レイスを纏う炎, レイスが持つソウル, plus **鎌の軌跡 A-D "scythe trail"** and **リッチの爆発 "Lich's explosion"**. `o00_6060` has bones named `c5080_レイスのカラス` in **NT and v1.00** (not just "the original DS2"), and PC Scholar no longer ships it | OK, confirmed +: a scythe |
| 5100 ファントムプリースト Phantom Priest | PC 3/14 has only weapon row 51000000 ファントムプリースト（霊体）"(spirit)". Its siblings are full enemies: ファントムナイト 511000 and ファントムメイジ 512000, both placed in 不死廟 (Undead Crypt). The Insolent set, mace and shield are named for the Priest. **The NT soul list has 幻影の騎士 and 幻影の術士 (phantom knight / phantom sorcerer)**, which supports a group fight; there is no priest soul | OK + |
| 5130 ヴァンクラッド王（過去）King Vanclad (past) | NT `EnemyParam 513000`. Retail Vendrick `c5146` uses **material `c5130`**. But `m0101` is a **97 m statue** of a crowned king (Gutter m10_25 and Grave of Saints m10_34), not a character model. Its body texture is `BD_c7020`, not c5130 ([unused map pieces](../unused/unused-map-pieces.md)) | partly: a statue, other textures |
| 5140 ヴァンクラッド王（現代）King Vanclad (present) | PC 3/14 attack and weapon rows: **＿カットシーン用 "for cutscenes"**. NT: `EnemyParam 514000`, NpcEvent 51400000, AI 年老いた王 "old king", npcMenu 王. **The boss Vendrick is `c5146` in v1.00 and in Scholar** (charaname 5146, model, AI `ai514600` 年老いた王). His head is `HD_c5140_M`. **`c6840` is Scholar only**: enemy 684000, no AI, one generator in **Memory of the King (m50_38)**, built entirely on the `c5140` texture set | fix: c6840 = the DLC memory figure |
| 6060 ヤドカリ城 Castle Hermit Crab | NT and v1.00 rows. PC 3/14 adds **60601000 ヤドカリ城（石化後）"after petrification"** and logic row 6060 ヤドカリ城（仮）. Malformed Shell / Claws are tagged DS:. **The NT soul 宿り大蟹 (60733000)** and the crab-shaped tower in the early Bastille `m0000` (memory: crab tower) fit a crab that ends up as stone | OK +: petrified; soul |
| 6090 巨人族親衛隊長 Giant Guard Captain | NT: `BossParam`, charaname 6090 / 6091 巨人親衛隊長, test AI `ai009988`. PC 3/14 `BossPartsParam` lists **head, left / right front arm, left / right rear arm, two legs**, so four arms are confirmed. Variants: **609010（石化後）"after petrification"** (the stone giant `o10_9200`, textures `BossStone` / `c6090`) and **609100（幻影）"phantom"**. **`scn201000` exists in PC 1.02 only and casts `c0001` (the player) and `c6090`**. scn2010xx = map m20_10, probably the giants' Memories, the series retail's `scn201010` belongs to | OK +: petrified and phantom forms |
| 6130 脳幹の神木 Brainstem Divine Tree | `MapObjectParam` 139000 and 229000 both 脳幹の神木; Santier's Spear also has a row 3251000 "after the tree's destruction". **NT ties**: soul 古の神木 (60743000), key item 神木の小枝 "sacred-tree twig", and the Majula cat's line that the Old Forest's sacred-tree branch "is connected to a demon's head", so the whole forest became a demon ([Network Test bosses not in retail](nt-bosses-not-in-retail.md#5-dialogue-nt-only-talk-text-all-japanese-unless-noted) s5). A brain*stem* tree attached to a head fits that line | OK +: NT soul and lore |
| 6140 双頭のワイバーン Twin-Headed Wyvern | Rows only. NT soul **双頭竜 "two-headed dragon" (60744000)** is the likely match | OK +: soul |
| 6150 リヴァイアサン Leviathan | Rows only; no NT link. `h04_8100` not checked | OK |
| 6160 ヴァンクラッド王（変形後）| Rows only | OK |
| 6170-6190 シルバーチャリオット | NT AI `ai618000/618010` シルバーチャリオット (the c6180 numbering). PC 3/14 adds 619100 馬のみ "horse only (for debugging)" and 619110 合体 "combined", whose 8th part is 処刑人 "executioner", plus 61912000 崖しがみつき "clinging to the cliff". **`o23_9050` (static chariot, `c6180` textures) = MapObjectParam 239050 【拷問場】シルバーチャリオット（仮）"[torture ground] Silver Chariot (temporary)"**. Executioner armour = （作成中）"being made" 【シルバーチャリオット】 with model 6180 | OK + |
| 6210 達磨 | Rows in NT and later; no more known | OK |
| 6220 / 6230 腐食虫 | Same as [Gutter's giant ant](gutter-ant-c6080.md) (today): shell 608000 is the placed Gutter ant, with 16 antidote-gas vents; body and young never got models | OK + |
| 6240 巨人王 Giant King | PC 3/14 names 624000 **巨人王　※没 "Giant King (cut)"** and 624500 巨人王. `MapObjectParam 109230` 【抵抗の砦】巨人王 = `o10_9230`, and `109250` 巨人王登場地割れ瓦礫 "rubble from the crack the Giant King appears through" (抵抗の砦 is the Forest of Fallen Giants: Pate's trap objects carry the same tag). **Retail souls: 64010000 最後の巨人 (Last Giant) = 巨人王亡者 "hollow Giant King"; 64240000 巨人の王 (Giant Lord) = 巨人王生者 "living Giant King"** | OK +: [the Last Giant is the Giant King too](two-soul-bosses.md#1-retail-who-has-two-souls) |
| 7020 王妃（現代）Queen (present) | NT AI: 下位：巫女（死骸）/ 上位：**王妃（死骸）"queen (corpse)"**. NT npcMenu 王妃, **but NT charaname 7020 = ヴァンクラッド王**. The `c7020` *male* textures are the king's: `BD_c7020` on the m0101 king statue, `HR_ / WP_c7020_M` on retail Vendrick. **`o21_9200` is textured `c7030_F`** and is named 【王城】王妃NPC仮モデル "[castle] queen NPC temporary model", so it belongs with 7030 rather than 7020 | fix: model id |
| 7035 王妃（過去）Queen (past) | 7030 = 王妃（過去）＿カットシーン用 (cutscene), 7035 = ＿インゲーム用 (in-game), 7036 = 新王妃 "new queen" (retail Nashandra), all textured `c7030_F` (checked). NT charaname 7030 = デュナシャンドラ王妃 | OK |

### Characters

| Sheet | What we have | Verdict |
|---|---|---|
| 7015 龍の巫女（子供）| [Child Emerald Herald (c7015)](child-herald-c7015.md). PC 3/14 adds **EnemyParam 701500 ＿インゲーム用：アン・ディールの館 "in-game version: Aldia's Keep"**; she was placed there by design. The model is in v1.00 and PS3 Scholar, gone on PC Scholar; the credits list "Shanalotte (Child)" | OK +: Aldia's Keep |
| 7210 王国軍師（現代）| **This is Wellager's own number**: NT charaname 7210 ベラガー, NT npcMenu 宰相 "chancellor", v1.00 npcMenu 7210 = Chancellor Wellager. Retail places 721100 = 王国軍師（**過去**）"(past)" (AI 軍師の霊 "strategist's spirit") in Drangleic Castle (see [his Llewellyn kit](llewellyn-references.md#cut-and-leftover)). The unplaced 721000 is the "present" form, with a full NT row and NpcEvent row. Nothing says alive or dead | fix: Wellager, present-day form |
| 7220 刻の研究者 Time Researcher | **= Velderick** (row 38 of the sheet). NT charaname 7220 ベルデリッグ; 57 Majula lines on crossing "the wall of time and space into the past", the Pendulum of Time and the Drop of Time (`nt-draft-talk-folder.md`). NT AI `ai722000` 下位：刻の研究者 | merge rows 37 + 38 |
| Velderick | see 7220 | as above |
| 7290 忠犬 Loyal Dog | NT: AI `ai729000/729010`, npcMenu 忠犬, NpcEvent 72900000. Reward lot 1729000 忠犬-？ | OK + |
| 7400 刻印の在りし者 | **= Lanafast** (NT charaname 7400 ラナファスト). NT `EnemyParam 740000` | merge rows 40 + 41 |
| Lanafast | see 7400 | as above |
| 7410 心折れた青教戦士 | **Saulden's own id**, not an early version: NT charaname ソダン, v1.00 npcMenu Crestfallen Saulden, PC 3/14 EnemyParam 741000 "…：マデューラ" (Majula). His corpse object is 【汎用】宝死体 c7410 "treasure corpse c7410". Whether he had a unique model was not checked | fix |
| 7440 パッチ Patches | NT AI 下位 / 上位：パッチ, npcMenu パッチ, English draft "Pait". PC 3/14 has Patches rows for the Forest of Fallen Giants, Tseldora, 溜まりの谷 and a white phantom. Armour 2744010x = **Iron Helm** + Hard Leather, labelled 【パッチ（現代）】"Patches (present)". Pate's Spear and Shield are tagged "NPC：パッチ"; the Tseldora Den key = パッチの家の鍵 "Patches' house key" | OK + |
| 7460 追跡する戦士 | NT `EnemyParam 746000`; PC 3/14 adds the location ：廃墟の城 "ruined castle" (see [the early Bastille's labels](../unused/unused-map-pieces.md#lost-bastille-m10_16-an-earlier-bastille-with-character-labels)) | OK |
| 7540 囚われの姫 | OK: Melentia. NT npcMenu already reads 行商メレンティラ / "Merchant Hag Melentia", and the NT places her at the Copse bonfire (s12 / s19). Only the weapon, behaviour and attack rows say 囚われの姫; EnemyParam / logic say よろずババア "general-store hag" | OK |
| 7550 ストーカー戦士 | **= Wickley**: AI `ai755000` ストーカー戦士**ウイックリー** (NT and v1.00; gone on PC), NT charaname 7550 ウィックリー. Three weapon variants (default, two-handed, dual-wield). Mad Warrior set = 狂戦士 armour labelled 【ストーカー戦士】 | merge rows 46 + 47 |
| Wickley | see 7550 | as above |
| 7570-7590 三姉妹 Three Sisters | NT npcMenu 三姉妹Ａ／Ｂ／Ｃ; NpcEvent rows in every build; reward lots 1757000-1759000 "event 1". **The v1.00 disc (Jan 2014) already names 7570-7590 Nadalia, Alsanna and Elana**, months before the DLC. So the data shows the DLC used the same three slots, not different characters. v1.00's English already has "Elana, **Queen of Ruin**". The DLC boss is "Elana, the **Squalid Queen**" (Scholar charaname 6820), and npcMenu 7590 was never updated. Scholar's item text calls all three a "child of Dark" | fix |
| 7603 / 7604 バンシーのブラフ | There is also **7602 バンシーのブラフ１** (weapon row, NpcNameSwap). The Amana singers are enemy 304000 / 304010 バンシー（亡者）：怖い顔 / 美女顔 "banshee (hollow): scary face / beautiful face" ([Shrine of Amana across builds](../builds/shrine-of-amana-pc.md)) | OK + |
| 7650 道具屋 Tool Shop | NT AI `ai765000/765010` and `EnemyParam 765000` | OK + |
| 7780 よろず屋（現代）| **= Colmcille** (NT charaname 7780 コルムキル); PC 3/14 name has （現代）"(present)" | merge rows 54 + 55 |
| Colmcille | see 7780 | as above |
| 7870 刻のダンジョンの妖精 | NT `EnemyParam 787000` | OK |
| 7890 Cale's sister? | **`scn101520` exists in PC 1.02 only**; it casts **`c7510`** (Cale: PC 3/14 names 7510 妹を探す兄 "brother looking for his sister") and `c7890`. scn1015xx = m10_15 = Aldia's Keep. No 7890 rows in any build. **Note**: the child Herald was also meant for Aldia's Keep (7015 above). It is a different id, so this is only a coincidence worth a look | OK, confirmed |
| 9061 フラムト Frampt | There is also 9062 フラムト(環境マップ無し) "(no environment map)"; NT weapon row 906100 | OK + |
| 城下町の長老 Castle Town Elder | Old Whip = （NPC：城下町の長老：鞭）. NT Majula cat 77701200: "Dran Castle Town (ドラン城下街) is closed off because barbarians attacked it long ago" (`nt-draft-talk-folder.md`). The Darkwraith's "misty castle town (past)" may be the same town | OK + |

### Boss souls (sheet rows 62-88)

Same 26 ids and Japanese as our list; ours has a 27th, 50140000 古の龍 (Ancient Dragon). Our table in
[Network Test bosses not in retail](nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000) s2 gives a retail counterpart for each. Ties to the sheet's own rows: 宿り大蟹 ↔ 6060 hermit
crab; 古の神木 ↔ 6130 brainstem tree; 双頭竜 ↔ 6140 twin-headed wyvern; 幻影の騎士 / 幻影の術士 ↔ the 5100 group fight;
ヴァンクラッド王 ↔ 5130 / 5140; デュナシャンドラ ↔ 7030; 処刑人 ↔ 6180. 岩の巨人カノッソフ "rock giant Kanossof" and the
petrified Giant Guard Captain are both stone giants, but no file links them. (Sheet typo: row 65 has a stray "The".)

## 2. New to our notes, from the sheet (all checked 2026-10-28)

1. Cut cutscenes in **vanilla PC 1.02 only**: `scn102530/102540/102560` (Lich + Wraith), `scn201000` (Giant Guard
   Captain + player), `scn101520` (Cale + c7890). No PS3 build and not PC Scholar. Our PC name dictionary
   doesn't know them (they list as hashes `#f11cc333`, `#c133d614`, `#6161fbd6`, `#fe514e8f`, `#55a1bb19`).
2. `WP_2450_M`, the blue flower mace with mesh `WP_c2250_M`: also PC 1.02 only (hash `#4d5d5b28`). **Correction
   2026-11-09:** it is in every vanilla PC build (31 March 2014 pre-release, 1.02, 1.12) as an unnamed file; "only"
   meant only vanilla PC, not any PS3 build or PC Scholar ([Network Test-only equipment models](../unused/nt-only-equipment-models.md)).
3. The Imperfect `c6830`'s mesh is named `c5050`.
4. `c7036` Nashandra's textures are `c7030_F`. Vendrick `c5146` borrows `HD_c5140_M`, `HR_ / WP_c7020_M`, `LG_c3370` and the
   `c5130` material.
5. `c6500`'s textures include `WP_2320_M`.

Not checked: the concept art and Design Works interview, `h04_8100`, the King's set model id.

## 3. Corrections to the sheet

- **7410** is Crestfallen Saulden's id in every build, not an earlier character.
- **7210** is Wellager by name. The retail ghost is the "past" row; the cut one is the "present" row.
- **5140**: boss Vendrick = `c5146`; `c6840` = Scholar's Memory of the King figure (textures `c5140`).
- **5130**: `m0101` is a 97 m statue with `BD_c7020` textures, not a c5130 model.
- **7020**: `o21_9200` is textured `c7030_F`; the `c7020` male textures belong to the king.
- **7570-7590**: the v1.00 disc already uses the DLC names; same characters, not different ones.
- **Merge** Velderick into 7220, Lanafast into 7400, Wickley into 7550, Colmcille into 7780.

## 4. What the developers' names add (PC 3/14), beyond both

- **Past and present pairs everywhere**: King Vanclad （過去）5130 / （現代）5140; Queen （過去）7030 / 7035 / （現代）7020
  (the present-day queen's AI is "queen (corpse)"); strategist （過去）721100 / （現代）721000; general store （現代）7780;
  Patches （現代） armour; Darkwraith in "misty castle town (past)"; Humanity in "flooded place (past)". With the time
  researcher, the "dungeon of time" fairy and the NT's "King Vanclad researched going back in time" (77700700), this is
  a designed two-era structure, stated in the developers' own names rather than inferred from dialogue (see [the past and present areas, map by map](e3-demo-leftovers.md#9-past-and-present-the-other-time-labelled-areas-added-2026-11-03)).
- **Petrified forms**: hermit crab （石化後）, Giant Guard Captain （石化後）.
- **Developer "cut" marker**: 巨人王　※没.
- **"DS:" item tag** = boss-soul trade weapons: Hermit Crab, Brainstem Tree, Twin-Headed Wyvern, Leviathan, Lich,
  Silver Chariot and the queen each had soul weapons planned. "NPC：" tags NPC weapons (Castle Town Elder,
  strategist, Patches, Stalker Warrior).

## 5. Is the sheet's "Bosses" section all bosses? (asked 2026-10-28)

**No.** The data supports about half of its 21 rows as bosses. Markers checked:
- boss params (`BossParam`, `BossPartsParam`, `LockOnParam`, `DeadTypeParam`, boss camera) in the NT, v1.00 and PC 3/14;
- an NT boss soul;
- a **"DS：" weapon**. All 24 items with this tag in PC 3/14's `ItemParam` names are boss-soul weapons as planned. They
  include Mytha's Bent Blade（ナーガ）, Freja's two（蜘蛛の女王）, the Bone Scythe（ジャバザハット）, the Spotted Whip（美声カエル）,
  the Old King's hammer（熔鉄デーモン）, the Dragonrider halberd, the Chariot Lance and the Chime / Bow of Want. So a "DS："
  owner was meant to give a soul.

| Read | Rows | Basis |
|---|---|---|
| boss, by its params | **6090** Giant Guard Captain | `BossParam` 6090 / 609000 / 609010 (NT), `BossPartsParam` in all three builds, health-bar name, boss test AI, cutscene, DS: Giant Stone Axe |
| planned boss, by its soul | **5070** Lich, **6060** hermit crab, **6130** brainstem tree, **6140** twin-headed wyvern, **6150** Leviathan, **6160** Vanclad transformed, **6240** Giant King | DS: weapons (6160's is the **King's Ultra Greatsword**, retail's Soul of the King weapon); NT souls for 6060 / 6130 / 6140; 6240 by the Last Giant / Giant Lord soul names (巨人王亡者 / 生者) |
| parts of one boss | **6170 / 6180 / 6190** | the chariot's body / executioner / horse; the boss rows are 619100 / 619110 (c6191) |
| probably in a boss fight | **5080** Wraith; **5100** Phantom Priest | 5080 shares the three boss-style cutscenes with the Lich and is size 超超特大, but has no soul or DS: weapon, and has a sound-reaction logic variant (`ai508030`) like an ordinary enemy. 5100 has one weapon row; its gear is tagged as its own, not DS:. Its two siblings are ordinary retail enemies with NT souls 幻影の騎士 / 幻影の術士, but there is no priest soul |
| no boss evidence | **5050** 恐獣, **5130** Vanclad (past), **6210** daruma, **6220 / 6230** corrosive insect | rows only. 5050 became The Imperfect, an ordinary Scholar DLC enemy. The insect's shell is the inert Gutter ant, and 6230 is its "young" |
| NPC or cutscene forms | **5140** Vanclad (present), **7020** queen (present), **7035** queen (past) | 5140 is ＿カットシーン用 (the in-game king is 5145, lot 514500 = Soul of the King). 7020's AI is "queen (corpse)", size 極小. The boss Nashandra is a separate row, **627000 王妃の鎧形態 "queen's armoured form"** (`BossParam`, `BossBattleParam` 1021030); 7036 新王妃 is her NPC form |

No row except 6090 has a `BossParam` in any build checked. Two boss slots the sheet leaves out: **[Agdayne](agdayne-darklurker.md)** (c5065: NT
boss AI, soul 60742000, DS: **Darkdrift** 闇朧) and **c6100** 重騎馬兵：重騎士 (NT `BossParam` 610000). The DS: list also
names a salamander (Spitfire Spear) and [a male man-scorpion (Scorpion Stinger)](tark-najka-duo-claim.md#duo-boss).

## Method

Scratch scripts (session scratchpad, not kept): row names by id and by name from `2014-03-31\regulation` with
`paramdef.load`; row presence in NT / v1.00 / PC 3/14 regulations; AI labels from `ezstate\ai*.esd` headers (NT,
v1.00, Scholar); NT and v1.00 `charaname` / `npcMenu` FMGs (`fmg.py`); file presence by name (PS3 `gamedata.py`) and
by path hash (PC 1.02 via `pcindex.index`, PC Scholar via `bhd5`); cutscene, object and model binders unpacked with
`bndx` and searched for `cNNNN` and `sfx` strings.
