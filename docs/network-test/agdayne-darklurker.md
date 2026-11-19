# Agdayne and the Darklurker in the data

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-21. Retail BLUS41045 v1.00 and both network tests (NPUB90979, NPJB90625). Names in the game's own text:
Grave Warden Agdayne = 墓守アガドゥラン; Darklurker = 闇潜み (yami-hisomi).

## In the developer messages

- **The Darklurker's name, yes.** Four No-man's Wharf signs (`developer-signs.md`) use **闇ひそみ**, the same word as
  the boss's 闇潜み spelled in kana, for the creatures that hide in the Wharf's dark houses:

  | Sign | Japanese | English |
  |---|---|---|
  | m10_18 004 | 周りの壁も壊れる / 扉入って闇ひそみに気付いたプレイヤーが / 逃げようと扉から出ても壊して襲ってくる | The walls around break too / if a player goes in, notices the dark lurker / and runs back out of the door, it smashes through and attacks |
  | m10_18 006 | 闇ひそみの巣になっている家 / 暗闇の部屋があり迷い込むと非常に危険 / ２階の壊れた壁から出てくる | House that's a dark lurkers' nest / there's a pitch-dark room, very dangerous if you stray in / they come out of the broken wall upstairs |
  | m10_18 015 | この家だけは常に暗い / 闇ひそみが逃げる場所 | This one house is always dark / it's where the dark lurkers flee to |
  | m10_18 016 | 灯火台は目印の役割もある / この明かりの範囲だと闇ひそみが / 来ない どう戦うか？ | The beacon is also a landmark / dark lurkers won't come inside its light / so how will you fight? |

  These are the Wharf's dark-house enemies, not the Dark Chasm boss, so whether the designers meant the same creature
  is open. (`developer-signs.md` first rendered the word as "Darkdweller"; corrected.)
- **Agdayne: no.** No sign, label or in-game message names him or the grave wardens (墓守). The Undead Crypt's 11
  signs don't mention him, and the Dark Chasm's 15 don't name the Darklurker.

## Elsewhere in the data

**One character family.** Character models are numbered in families of ten:

| Number | What | Evidence |
|---|---|---|
| c5060 | no model; the family's base number | sound bank `sound\frpg2_c5060.bnd`; Agdayne's textures and NPC ids |
| c5061 | **the Darklurker**: winged, 4.4 m tall, "star" materials | `model\chr\c5061.bnd`; boss health-bar name 5061 闇潜み in `talk\charaname.fmg` |
| c5062 | an empty model (no geometry), enemy 506200: **the Darklurker's wormholes** | spawned by at least seven generators in the Dark Chasm (`GeneratorParam_m40_03`); its AI scripts are `ai506200_下位：ワームホール` / `ai506210_上位：ワームホール` ("wormhole"), see below |
| c5065 | **Agdayne**: a 1.87 m robed human with weapon `WP_5280` | `model\chr\c5065.bnd`; its body, face, hair, skin and eye materials and textures are all named **c5060** (`BD_c5060_M`, `FC_c5060_M`, `HR_c5060_M` ...) |

Agdayne's other ids sit on the base number too: NPC menu names 5060 and 5065 (墓守アガドゥラン), 43 talk lines
5060xxxx in `talk\m20_24_00_00.fmg` (opening "Halt. / Human. Put that light out. / Human. Do not produce light."),
`NpcTalkParam_m20_24` 5060xxxx, `NpcEventParam` 50600000-1, `ShopLineupParam` 506001xx-506003xx. One sound bank
serving a family of ten is the normal pattern (Vendrick, c5146, uses `frpg2_c5140`), and families group related
characters (Ruin Sentinels c3250-3253; Throne Defender, Velstadt and Throne Watcher c3320-3340). So the Darklurker
being numbered inside Agdayne's family is suggestive of a shared origin, not proof.

**Params** (enemy ids = model x 100): 506100 and 506101 the Darklurker (two boss rows, `BossBattleParam` 4003000 in the
Dark Chasm, generator 811); 506200 the empty c5062; 506500, 506501 and 506509 Agdayne (Undead Crypt generators 240, 241
and 686). Agdayne's spawn records (`GeneratorRegistParam_m20_24`) point at 506000, the family base (no param has that
row), where the Darklurker's point at its own 506100. Other 506xxxxx values in params are unrelated: items 50600000
(Soldier Key) and 50610000 (Key to King's Passage), a Drangleic event location 506000 in a 500000 / 504000 / 505000
sequence.

**Retail text** names each in its own place only: Agdayne's Black Robe, Cuffs and Kilt and their descriptions, the
Grave Warden Agdayne katana description, the staff roll ("-Grave Warden Agdayne-"), the NPC menu and his talk; the
Darklurker Soul ("Soul of what lurks in the Dark Chasm"). Neither text mentions the other.

**Executables:** neither retail's nor the network test's EBOOT contains either name or a c506x string.

## The network test

No c506x models, sound bank or params, and no 闇潜み anywhere. But it already has Agdayne, as a boss:
- `npcMenu.fmg` 5060 = 墓守 "Grave Warden" (no name yet);
- `ItemName.fmg` 60742000 = **アガドゥランのソウル "Agdayne's Soul"**, in [a run of 26 boss souls, 60730000-60755000](nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000)
  (Japanese only, placeholder descriptions 仮テキスト / "No Text Settings"; the first ten were added 2026-10-22):

  | Id | Name | Literal | Retail |
  |---|---|---|---|
  | 60730000 | 紅炎トカゲのソウル | crimson-flame lizard | |
  | 60731000 | 大蛙のデーモンのソウル | giant frog demon | |
  | 60732000 | 蜘蛛の女王のソウル | spider queen | the Duke's Dear Freja: retail's AI ai603020 for her model c6030 is labelled 蜘蛛の女王 |
  | 60733000 | 宿り大蟹のソウル | dwelling (hermit) giant crab | probably the crab tower in the early Bastille `m0000` ([unused map pieces](../unused/unused-map-pieces.md)) |
  | 60734000 | 幼生ナーガのソウル | larval naga | |
  | 60735000 | 地竜の化身のソウル | incarnation of the earth dragon | |
  | 60736000 | ヘル・スコーピオンのソウル | Hell Scorpion | |
  | 60737000 | デス・ストーカーのソウル | Death Stalker | |
  | 60738000 | 鏡の騎士のソウル | mirror knight | the Looking Glass Knight (same Japanese name, retail soul 64180000) |
  | 60739000 | カゲロウ竜のソウル | kagerou dragon (mayfly or heat haze) | |
  | 60740000 | 岩の巨人カノッソフのソウル | soul of Kanossof (?), the rock giant | |
  | 60741000 | 黒炎トカゲのソウル | black-flame lizard | |
  | 60742000 | アガドゥランのソウル | **Agdayne** | no soul in retail |
  | 60743000 | 古の神木のソウル | ancient sacred tree | |
  | 60744000 | 双頭竜のソウル | two-headed dragon | |
  | 60745000 | 竜騎兵のソウル | dragon cavalryman | retail's Japanese name for the Dragonrider |
  | 60746000 | 成体ナーガのソウル | adult naga | |
  | 60747000 | 貪りデーモンのソウル | | the Covetous Demon (same Japanese name) |
  | 60748000 | 魔手のソウル | demon hand | |
  | 60749000 | 呪手のソウル | cursed hand | |
  | 60750000 | 魔女リディーレアのソウル | the witch Lidirea (?) | |
  | 60751000 | 幻影の騎士のソウル | phantom knight | |
  | 60752000 | 幻影の術士のソウル | phantom sorcerer | |
  | 60753000 | ヴァンクラッド王のソウル | King Vanclad | Vendrick (same Japanese name) |
  | 60754000 | 王妃デュナシャンドラのソウル | Queen Dunashandra | Nashandra (same Japanese name) |
  | 60755000 | 処刑人のソウル | executioner | |

  Retail renumbered its boss souls (34 souls, 64000000-64330000) and has no Agdayne soul. The NT's one boss soul
  outside the run is 50140000 古の龍のソウル (Ancient Dragon, retail 64230000 古の竜のソウル, with 竜 for 龍); the
  English text of both network tests names none of them. The only soul tied to the c506x family in
  retail is the Darklurker Soul (64280000).

## The Dark Chasm boss region: "Agaturan's Stand" (found 2026-10-21)

The Dark Chasm's boss-music region is named **ボス戦BGM範囲_アガトゥランのスタンド**, "boss battle BGM range:
Agaturan's Stand", at (-1014, -17, 281). The Darklurker (506100) and its split copy (506101) spawn 32 m from its
centre, inside a ring of the seven empty c5062 "enemies" about 8 m out. アガトゥラン is アガドゥラン (Agdayne) with
ト for ド, and スタンド reads as "Stand", a spirit that a person manifests (the JoJo's Bizarre Adventure sense is
the obvious reading, but that's an interpretation). So the designers named the Darklurker's fight after Agdayne:
**the Darklurker was Agdayne's Stand.** This is the most direct link in the data, and it goes with the shared c506x
model family above and the network test's "Agdayne's Soul". See [boss working names](boss-region-names.md) for the other bosses'
working names.

## The scripts call the Darklurker "Agdayne" (spelling search, 2026-10-21)

Every file in all three games was searched (DCX unpacked, BND3 / BND4 entries and BHD / BDT pairs opened, plus the
EBOOTs) for アガトゥラン / アガドゥラン in UTF-16 BE and LE, Shift-JIS and UTF-8, and for romanised spellings
(agaturan, agaduran, agdayne and variants). Tool: `namescan.py <nt|jpnt|vanilla>`, kept in the session scratchpad.
No romanised spelling appears anywhere.

**"Agaturan" (ト) appears 3 times, all in Dark Chasm files:**

| File | Label | Literal |
|---|---|---|
| `map\m40_03_00_00\m40_03_00_00.msb` | ボス戦BGM範囲_アガトゥランのスタンド | boss battle BGM range: Agaturan's Stand |
| `ezstate\event_m40_03_00_00.edd` | ボス：アガトゥランのスタンド_戦闘 (followed by 終了) | boss: Agaturan's Stand, battle (end) |
| `ezstate\event_m40_03_00_00.edd` | アガトゥランのワームホール生成 | Agaturan's wormhole creation |

**"Agdayne" (ド) is used everywhere else, including for the Darklurker** (retail unless noted):

| File | Label | Literal |
|---|---|---|
| `ezstate\ai506100.esd` (the Darklurker, enemy 506100) | ai506100_下位：**アガドゥランのスタンド** | lower logic: **Agdayne's Stand** |
| `ezstate\ai506110.esd` | ai506110_上位：**アガドゥランのスタンド** | upper logic: **Agdayne's Stand** |
| `ezstate\ai506200.esd` / `ai506210.esd` (c5062) | 下位 / 上位：ワームホール | lower / upper logic: wormhole |
| `ezstate\event_m40_03_00_00.edd` | アガドゥランの分身死亡処理 (and its 【条件】/【再現】/【実行】/【プリセット】 variants) | death handling for Agdayne's double (the split copy, 506101) |
| | アガドゥランのジェネレータID | Agdayne's generator ID |
| | アガドゥランの特殊効果判定 | Agdayne's special-effect check |
| | ボス：アガドゥランを撃破したか？ | boss: has Agdayne been defeated? |
| `ezstate\talk_m10_25_00_00.edd`, `talk_m10_32_00_00.edd`, `talk_m20_21_00_00.edd` | 装備品譲渡（条件：アガドゥラン撃破） | equipment hand-over (condition: Agdayne defeated) |
| NT and JP NT only: `ezstate\ai506500.esd` | ai506500_アガドゥラン | Agdayne |
| NT and JP NT only: `ezstate\ai506510.esd` | ai506510_上位：アガドゥラン＿インゲーム用：汎用 | upper logic: Agdayne, in-game version, generic |
| text (`ItemName`, `DetailedExplanation`, `npcMenu`, `talk\m20_24`) | アガドゥランの黒衣 ... 墓守アガドゥラン ... 我はアガドゥラン | his armour, katana, NPC name, talk ("I am Agdayne, keeper of this mausoleum") |

The three talk scripts belong to **Darkdiver Grandahl** (闇潜りのグランダル), who stands at the Dark Chasm's three
entrances (the Gutter / Black Gulch map m10_25, Shaded Woods m10_32, Drangleic Castle m20_21). His script label is
**深淵の巡礼者 "Pilgrim of the Abyss"**, a working name not used in the shipped text. His gift for beating the
Darklurker is scripted as a reward for **defeating Agdayne**.

What this shows:
- "Agaturan" is a variant (ト for ド) confined to the Dark Chasm's map and event files, which were probably one
  person's work. That person also wrote アガドゥラン in the same event script, so it reads as a slip rather than an
  older name. Every other team (AI, NPC talk, text) wrote アガドゥラン.
- The AI programmers named the Darklurker "Agdayne's Stand" (see also [the soul rows' working names](nt-bosses-not-in-retail.md#7-the-developers-own-names-pc-pre-release-2026-10-28)), and the event and talk scripters went further and simply
  called it Agdayne: its split copy is "Agdayne's double", and killing it is "defeating Agdayne".
- The seven empty c5062 "enemies" round the arena are the boss's wormholes (the Darklurker's teleport points).
- The network tests had Agdayne's own fighting AI (`ai506500` "Agdayne", `ai506510` "in-game version") next to the
  grave-warden AI (`ai506000` / `ai506010` 墓守 "grave warden"). Retail kept the grave-warden pair for the NPC and
  dropped 506500 / 506510, and added the Darklurker's "Agdayne's Stand" pair and the wormholes. So the boss AI moved
  from Agdayne himself (Sept 2013) to his Stand (Jan 2014).
- An aside: the same Dark Chasm event script has 水滴が落ちてきて松明が消える, "water drips down and puts out the
  torch", the area's torch-extinguishing trigger, which fits Agdayne's "Human. Put that light out."

## Reading it together

In the network test (Sept 2013) Agdayne is a grave warden planned as a boss with his own soul. In retail (Jan 2014) he
is a non-hostile NPC who tells you to put your light out, his model is the family's c5065, and the boss in his family
is the Darklurker (c5061), with the soul. The Wharf designers' notes use the Darklurker's name for light-shy lurkers in
the dark. The Dark Chasm's boss region, "Agaturan's Stand", says it outright: the Darklurker was conceived as Agdayne's
Stand, a manifestation of him, rather than an unrelated boss.

Caveats: transcriptions and translations by an AI assistant; the readings of カノッソフ and リディーレア are guesses; the
"family" reading of the model numbers is an inference from the numbering pattern.
