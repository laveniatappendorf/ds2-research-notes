# Boss working names in the map layouts, and the Pursuer ("Vagrant")

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-21, retail BLUS41045 v1.00. Each map's MSB has boss-music regions (`POINT_PARAM_ST` entries named
`ボス戦BGM範囲_<name>` or `ボスBGM範囲_<name>`, "boss battle BGM range: <name>"). The names are the designers' working
names. Each region was matched to the boss-list characters (`talk\charaname.fmg`) whose spawn points lie within 45 m
of its centre (spawn point = `GeneratorLocation_<map>` + the map origin; enemy = `GeneratorRegistParam` +0, model =
enemy / 100). Tools (`tools\devsigns`): `boss_regions.py` (this table), `spawns.py <game> <map> <enemy id range>...`, `msb_regions.py <game> [name substrings]`.

## Every region

| Map | Region name | Literal | Boss spawning in it (distance, count) |
|---|---|---|---|
| m10_10 Forest of Fallen Giants | ベイグラント | **Vagrant** | **The Pursuer** (25 m) |
| m10_10 | 新・巨人上級兵士 | new elite giant soldier | The Last Giant (10 m) |
| m10_14 Tseldora | 蜘蛛の女王 | spider queen | The Duke's Dear Freja (24 m) |
| m10_14 | ザコボス | mook boss | Prowling Magus (2 m) |
| m10_15 Aldia's Keep | ワイバーン | wyvern | none within 45 m ([the Guardian Dragon's area, inferred](nt-bosses-not-in-retail.md#7-the-developers-own-names-pc-pre-release-2026-10-28)) |
| m10_16 Lost Bastille | ガーゴイル（ザコボス） | gargoyle (mook boss) | Belfry Gargoyles (41 m, x5) |
| m10_16 | **鏡の中の戦士** | **warrior in the mirror** | **the Ruin Sentinels** (16 m, x3) |
| m10_16 | 忘れられた罪人 | the Lost Sinner (the shipped name) | The Lost Sinner (4 m) |
| m10_17 Earthen Peak | シャバザハット | "Jabba the Hutt" (the developers' nickname for the Covetous Demon, per the author) | Covetous Demon (2 m) |
| m10_17 | ナーガ | naga | Mytha, the Baneful Queen (6 m) |
| m10_18 No-man's Wharf | 中ボス | mid-boss | none within 45 m |
| m10_18 | 両面戦士：ヘビ頭 | two-faced warrior: snake head | Flexile Sentry (9 m) |
| m10_19 Iron Keep | (blank: `ボスBGM範囲_`) | | Smelter Demon (6 m) **and a Pursuer** (6 m) |
| m10_19 | 熔鉄デーモン | Smelter Demon (the shipped name) | none within 45 m |
| m10_23 Huntsman's Copse | シルバーチャリオット | silver chariot | Executioner's Chariot (44 m) |
| m10_23 | スケルトン貴族 | skeleton nobles | Skeleton Lords (22 m, x3) |
| m10_25 The Gutter | ブッチャー | butcher | The Rotten (6 m) |
| m10_27 | 古の龍 | ancient dragon | none within 45 m (the Ancient Dragon, inferred) |
| m10_31 Heide's Tower | オーンスタイン | Ornstein | Old Dragonslayer (16 m) |
| m10_31 | 重騎馬兵：兵士のみ | heavy cavalryman: the soldier only | the Dragonrider (its spawn, enemy 611510, is 30 m away; c6115 isn't in the boss list) |
| m10_32 Shaded Woods | マンスコーピオン♀ | man-scorpion (female) | Scorpioness Najka (6 m) |
| m10_33 Doors of Pharros | 巨大ネズミ | giant rat | none within 45 m (Royal Rat Authority, inferred) |
| m10_34 Grave of Saints | 犬ネズミ | dog rats | Royal Rat Vanguard (11 m) |
| m20_10 | 新・巨人上級兵士（生者） | new elite giant soldier (living) | none within 45 m ([the Giant Lord, inferred](two-soul-bosses.md#1-retail-who-has-two-souls)) |
| m20_11 Shrine of Amana | 美声カエル | sweet-voiced frog | Demon of Song (4 m) |
| m20_21 Drangleic Castle | ミラーナイト | Mirror Knight | none within 45 m (Looking Glass Knight, inferred) |
| m20_21 | 王妃の鎧形態 | the queen's armour form | Throne Defender / Watcher and Nashandra (8-19 m), same spot as the next |
| m20_21 | 王妃の騎士A&C | the queen's knights A & C | Throne Defender (19 m), Throne Watcher (15 m) |
| m20_21 | 重騎馬兵：兵士のみ　２体戦 | heavy cavalryman: the soldier only, two-body fight | the Twin Dragonriders (21 m); two Pursuers 14 m away, 22-28 m lower |
| m20_24 Undead Crypt | 王妃の騎士B | the queen's knight B | Velstadt (12 m) |
| m40_03 Dark Chasm | **アガトゥランのスタンド** | **[Agaturan's (Agdayne's) Stand](agdayne-darklurker.md)** | the Darklurker (32 m, x2: the boss and its split copy) |

Readings: the Covetous Demon was nicknamed after Jabba the Hutt; the Dragonrider is a cavalryman without his mount; the Throne Defender, Watcher and Velstadt were "the
queen's knights" A, B and C; the Ruin Sentinels, three identical warriors, were ["the warrior in the mirror"](mirror-names.md#2-the-ruin-sentinels-c3250). The Last
Giant and the Giant Lord are "elite giant soldiers", the term the Gutter's developer signs use for their giants
(巨人上級兵, `developer-signs.md` m10_25).

## The Pursuer was "Vagrant"

- **Forest of Fallen Giants:** the boss region is ボス戦BGM範囲_**ベイグラント** at (148, -10, -203), with a light
  `環境ライトID6ベイグランドカットシーン用` "environment light ID6, for the Vagrand cutscene" at (143, 10, -200). The
  Pursuer boss (enemy 318010, generator 853, `BossBattleParam` 1010000) spawns at (139.6, 10.0, -191.6): 25 m from the
  region's centre and 9 m from the cutscene light. A second Pursuer spawn (318011, generator 2255) is 100 m away at
  (67.3, 1.7, -261.0). The hidden eagle object `o00_6090` ([hidden map parts](../unused/hidden-map-parts.md)) is 113 m from the arena.
- **Lost Bastille sign 020** "ベイグラント鷹タクシー / 到着地点" (["Vagrant hawk taxi / arrival point"](community-cut-sheet-crossref.md#enemies)) is at
  (-150.8, 13.5, 487.6), 6 m from The Tower Apart bonfire.
- **Drangleic Castle sign 017** "玉座(謁見の間) / ベイグラント" ("Throne (audience chamber) / Vagrant") is at
  (-478.8, 90.2, -363.0), **11 m from the two Pursuers** that spawn there (enemy 318000, generators 8000 and 8010,
  at (-487, 86, -369) and (-487, 86, -358)). These two appear only in NG+ or after burning a Bonfire Ascetic (per
  the author); the generators' own conditions weren't checked (see [NG+ near boss arenas](../modding-world/ng-plus-differences.md#the-other-arenas)).
- **Iron Keep:** a Pursuer (318020, generator 5000) spawns at (-718.5, 176.3, 655.7), inside the boss region whose name
  is left blank, next to a light labelled "mid-boss room" (中ボス部屋), 6 m from the Smelter Demon's spawn.
- **No Pursuer spawns in the Lost Bastille's v1.00 data:** its "warrior in the mirror" region is the Ruin Sentinels'. [The Bastille's multiple-Pursuer fights were added in Scholar of the First Sin](../builds/placements-balance-pc.md#1-enemies) (the PC / PS4 edition; not in the PS3 "Scholar" release), per the author, which fits: there's nothing of them here to find.
- **Text:** the game's text always says 呪縛者 "the Pursuer" (boss name 3180, Soul, Ultra Greatsword, Greatshield).
  The Soul's Japanese description calls him the Pursuer "who **wanders** (放浪する) the land of Drangleic", which
  echoes "Vagrant"; the English has "who lurks in Drangleic". Neither executable contains ベイグラント.
- **Network test:** no Pursuer character (no c3180 files), but its handed-out Pursuer's Greatsword (900008420) is
  already 呪縛者の大剣, and it has the eagle model `o00_6090`. So the shipped name existed by September 2013, while
  the map layouts and signs kept "Vagrant".
