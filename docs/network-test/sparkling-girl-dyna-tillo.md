# "Sparkling girl" (キラキラ娘) is the Network Test's name for Dyna & Tillo

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-10-25. The author's questions: what other references to "sparkling girl" exist, and could she be related to the cut
girl-and-dealer text (talk file `m20_04`)? Translations are mine unless a column says "retail English" (FromSoftware's own
text from retail v1.00's English talk file).

## Short answer

- **キラキラ娘 is the Network Test's working name for NPC 7860, who is Dyna & Tillo (ドネ＆ティロ) in retail**: the pair in Things
  Betwixt (map `m10_02`) who ask for a Smooth & Silky Stone, a Small Smooth & Silky Stone or a Petrified Egg and answer with a
  random draw (the talk script's own state name is ガチャ, "gacha"). The Network Test has only her two AI scripts and her
  entry in the NPC name list; retail added the talk script, the lines and the placement, and renamed her.
- **The developers' old name survives in retail**: the Things Betwixt talk script (`talk_m10_02_00_00.edd`) still labels her
  states キラキラ娘, in v1.00, v1.10, PS3 Scholar and PC Scholar.
- **No data link to the cut girl and dealer.** By id they are other NPCs (7010 and 7040, not 7860) in another map. The only
  overlap is thematic (see the last section), and that is my reading, not something the files say.

## Every place the name appears

Scan for キラキラ (UTF-16 both ways, Shift-JIS, UTF-8) over every non-model file of the Network Test (3,114 files) and
retail v1.00 (9,047), and over the `ezstate`, `talk` and `npcMenu` files of v1.10, PS3 Scholar and PC Scholar. Hits:

| Where | Builds | What it says |
|---|---|---|
| `ezstate\ai786000.esd` / `ai786010.esd` (header label) | Network Test, v1.00, v1.10, PS3 Scholar. **Not on PC** | `ai786000_下位：キラキラ娘`, `ai786010_上位：キラキラ娘` ("lower / upper logic: sparkling girl"). 1.10 touched both (the `.edd` of 786000 grew 58,530 -> 59,320 bytes; the `.esd` kept its size) |
| `menu\Text\Japanese\npcMenu.fmg`, id 7860 | **Network Test only** | キラキラ娘. Retail (all languages) has ドネ＆ティロ / "Dyna & Tillo" at that id; no later build has キラキラ in `npcMenu` |
| `ezstate\talk_m10_02_00_00.edd` (state names) | v1.00, v1.10, PS3 Scholar, PC Scholar | the state names below. The Network Test archive has no `m10_02` talk script at all (its only talk script is `m10_23`) |
| nowhere else | | no hit in item names, maps, event scripts, param names or any other text, in any build scanned |

(A stray hit on the single kanji 煌 and on half-width ｷﾗｷﾗ turned up in model / collision binaries: byte noise, ignored.)

## What the character is in the data

**Enemy row 786000** (v1.00): body `c2250`, HP 10, stamina 20, 0 souls, defence 0 against physical, petrify 100,
`name_text_id` 7860, spawn limit 1, death item lot 78600000. `c2250` has a 28 KB model file and **no textures**, so she is
invisible ([the PC cut-content note](cut-content-pc.md#5-ai-scripts-51-unreferenced-ones-are-gone) already calls it "the invisible c2250"). Later patches touched the row: 1.02
`chr_round_damage_param_id` 1 -> 21, 1.10 `coop_time_reduction` 90 -> 60.

**Placement**: one generator, id 320 in `m10_02_00_00` (Things Betwixt), regist row 78600000 (enemy 786000, logic ids -1),
AI think id 10, spawn limits 0. The same generator is in v1.00, v1.10 and PC. Position (360.82, -5.44, 136.00) is identical on
PS3 Scholar (1.10) and PC; v1.00's position was not compared. The Network Test has **no** generator naming enemy 786000
(and I did not decode the Network Test's own row 786000: its layout does not fit the shared decoder).

**The talk script**, `talk_m10_02_00_00`, holds her next to the three retired fire-keepers (retail: Strowen, Griant, Morrel)
and Milibeth. Her states (the file's own labels):

| Japanese | Translation |
|---|---|
| キラキラ娘：エリア未侵入 | sparkling girl: area not yet entered |
| キラキラ娘：エリア変数：リセット | area variable: reset |
| キラキラ娘：ガチャ実行 | **gacha execution** (a random draw) |
| キラキラ娘：取得後：待機 | wait after receiving |
| たまごの化石 or 小 or つるすべ石を置いた | placed the egg fossil, or the small one, or the smooth stone (小 read as "small": a guess) |
| 交換対象外アイテムを置いた | placed an item that is not eligible for exchange |
| キラキラ娘：エリア侵入 / 侵入：開始 | area entered / entry: start |
| キラキラ娘：ランダム再生 (開始 / 分岐 / 再生) | random playback (start / branch / play) |
| 一定領域に侵入：ランダム１ / ２ (連続再生) | enters a fixed zone: random 1 / 2 (continuous play) |
| 初回会話 / 次回会話 | first conversation / next conversation |

Items (retail ItemName): たまごの化石 **Petrified Egg** (62190000), つるすべ石 **Smooth & Silky Stone** (50880000),
小さなつるすべ石 **Small Smooth & Silky Stone** (50885000). 何かの化石 Petrified Something (60511000) is not named in the
state labels.

**The lines** (retail `m10_02` talk text, ids 786000xx; all 18; English is retail's official text). Two voices alternate: the
ids ending 0 say つるつる ("smooth"), the ids ending 10 say スベスベ ("silky"), the two halves of つるすべ ("smooth and silky"),
so presumably one line per twin:

| id | Japanese | Retail English | Role (my reading) |
|---|---|---|---|
| 78600000 / 010 | ねえ、ねえ / つるつる・スベスベちょうだい | You, you, give us smooth. / Yes, you, give us silky! | asks |
| 78600100 / 110 | ねえ！ねえ！ / つるつる・スベスベちょうだい！ | You! You! Give us smooth! / Yes! You! Give us silky! | asks again |
| 78600200 / 210 | ねえ、ねえ / つるつる・スベスベじゃないね | You, you, that not smooth. / No, you, that not silky. | wrong item |
| 78600220 / 230 | ちょっとがっかり / すこしがっかり | Oh, pooh. / Oh, phoo. | disappointed |
| 78600300 / 310 | うん、うん / つるつる・スベスベだね | Yah, yah. So nice, so smooth. / Yah, yah. Silky smooth. | right item |
| 78600400 / 410 | ねえ、ねえ / あれ、ちがうね・ちがうね | You, you, not like that. / No, you, not at all. | wrong item |
| 78600500 / 510 | ねえ、ねえ / あれ、いらないね・もう、いらないね | You, you, no, not that. / Yes, you, No need, no more. | refuses |
| 78600600 / 610 | それ、それ / おれいだよ・おかえしだよ | There, there. Thanks for you. / Yes, there. Gift for you. | gives the return gift |
| 78600700 / 710 | ぜんぶありがと・うん、うん / たくさん、いっぱい | Thank you. We have plenty. / Yah, yah. We get many. | done |

(78600000-78600710 are 18 ids; the table pairs them.) The row's death item lot 78600000 reads, in its raw words, Human Effigy
then Rubbish filler (ItemLotParam2_Chr); I did not decode that lot's layout, so treat it as a stub.

## Which builds have what

| | AI scripts | NPC name, id 7860 | `m10_02` talk script with her states | her lines | generator 320 |
|---|---|---|---|---|---|
| Network Test (US) | yes | キラキラ娘 | **none in the archive** | none (its `m10_02` talk text has 59 lines, none hers) | none |
| v1.00 | yes | Dyna & Tillo | yes | yes (126 English lines in the file) | yes |
| v1.10 | yes (touched) | not キラキラ | yes | not checked | yes |
| PS3 Scholar | yes | not キラキラ | yes | not checked | yes |
| PC Scholar | **no** | not キラキラ | yes | not checked | yes |

So the PC runs her without the AI script that the PS3 builds ship; the talk script and placement carry her.

## Is she related to the cut girl and dealer (`m20_04`)?

**By id: no.** In the NPC tables a talk id's first four digits are the NPC id (checked on the Emerald Herald 7000 ->
70002xxx, Strowen 7050 -> 70500xxx, Griant 7051 -> 70510xxx, Milibeth 7230 -> 72300xxx, Dyna & Tillo 7860 -> 78600xxx).
By that rule:

| NPC id | Network Test name (`npcMenu`) | Retail name | Talk ids |
|---|---|---|---|
| 7010 | 龍の巫女（子供） "dragon's shrine maiden (child)" | Emerald Herald | 70100xxx = **the cut girl** (`m20_04`) |
| 7040 | 梯子屋 "ladder seller" | Laddersmith Gilligan | 70400xxx = **the cut dealer** (`m20_04`) |
| 7860 | キラキラ娘 "sparkling girl" | Dyna & Tillo | 78600xxx (`m10_02`, retail) |

- The cut girl sits in the slot that became the Emerald Herald (as a child), and the dealer in the slot that became Gilligan.
  Retail's unused child model c7015 belongs to that 7010 slot, not to the sparkling girl ([child Emerald Herald (c7015)](child-herald-c7015.md), 2026-10-26).
  Neither is the sparkling girl's slot, and `m20_04` is a village with no map in any build, while she stands in Things
  Betwixt.
- Timing differs: the cut text is in the Network Test, the JP test and v1.00 unchanged (removed in patch 1.03,
  [cut talk timeline](../builds/cut-talk-timeline.md)); her lines and states exist only from v1.00, her AI and name from the Network Test.
- More NT traces of the two cut NPCs (found 2026-10-25 while cross-referencing bosses, [Network Test bosses not in retail](nt-bosses-not-in-retail.md)): NT-only AI
  scripts `ai700500` / `ai700510` (龍の巫女 "dragon's shrine maiden", 「インゲーム用」 "in-game version") and `ai701500`
  (龍の巫女（子供） "...(child)") for NPCs 7000 / 7010, and `ai704500` / `ai704510` **奴隷商人 "slave merchant"** (in-game
  version) for NPC 7040. So the NPC whose talk ids carry the dealer's lines also had a "slave merchant" script, which fits the
  `m20_04` dealer who sells the girl; the "ladder seller" label is the same NPC's other script.

**Thematically, there is an echo (inference only).** Both are childlike characters who ask the player for particular items
and react to whether the right one is brought, with disappointed and pleased lines: the cut girl wants an Estintia's Feather
(an item only the network tests have), forgives you if you bring one, and says she owns "lots and lots of pretty things";
Dyna & Tillo want smooth stones, say "that not smooth... oh, pooh" and give a gift back. It may be one design habit reused;
nothing in the files ties the two characters together.

## Not known or not checked

- What the gacha gives (the reward tables were not traced), and which twin says which line.
- Why キラキラ ("sparkling") was chosen: no file says. A lead (found 2026-10-25): in the Network Test the body c2250 belongs to
  **光虫 "light insect"** (AI `ai225000` / `225010` / test `225090`, enemy 225000, no retail equivalent), and retail gives c2250
  (no textures) to enemy 786000. The body that was a light insect became her invisible body, which fits "sparkling"; no file says
  it was reused on purpose (but see [the developers' row name 光虫（NPCで流用）](community-cut-sheet-crossref.md#enemies)).
- The JP network test (NPJB90625) was not scanned; the v1.10 / Scholar / PC talk text for her was not read.
- Network Test row 786000: its body is c8000 (first word of the row; most NT NPC rows use that body), not c2250. The rest of the
  row was not decoded (the NT's 320-byte layout does not fit the shared decoder).

Related: [Network Test AI labels](nt-ai-labels-retail-enemies.md) (the 306 labels; this NPC is `ai786000` / `786010`), [cut content in PS3 Scholar](cut-content-scholar.md)
and `cut-content\cut-dialogue-translation.md` (the girl and dealer), [cut talk timeline](../builds/cut-talk-timeline.md).
Scratch scripts used (session scratchpad, not kept): a byte scan for the name over each build's archive, a state-label dump of the
talk `.edd`, `fmg.py` reads of `npcMenu`, `ItemName` and the talk text, `bosses_pc.generators` for the placement.
