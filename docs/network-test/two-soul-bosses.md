# Bosses with two souls, and the past / present plan (2026-11-04)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

After the Tark check ([Tark and Najka duo claim](tark-najka-duo-claim.md)), the author asked whether any other boss has two souls, and whether two
souls could belong to the cut past / present (time travel) mechanic. Sources: the NT soul names ([Network Test bosses not in retail](nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000)
table 2), the 2014 PC pre-release developer row names (rows the same in PS3 v1.00), the NT and v1.00 map lists, and
v1.00 placements. Translations are mine.

## Short answer

- **In the NT soul list, one boss clearly has two souls: Mytha**, as 幼生ナーガ "larval naga" (60734000) and 成体ナーガ
  "adult naga" (60746000). Five more pairs could be one boss in two forms or two bosses fought together (table 2).
- **Retail has exactly one boss with two souls, and it is a past / present pair: the Giant King.** The Last Giant is
  "Giant King (Hollow)", fought in the present Forest (the `is_hollow` flag is off on his v1.00 enemy row: see
  [hollow bosses](../mechanics/hollow-bosses-v1.00.md)). The Giant Lord is "Giant King (living)", fought in the giants'
  memory, which is labelled past. One character, one soul per era.
- **The NT list does not give a soul per era.** The characters the developers did split by era (King Vanclad past /
  present / transformed, the Queen past / present) have one soul each. Mytha's pair is a growth stage, and her area has
  no past version anywhere in the data.
- **New: the Shaded Woods had a planned past stage.** Its working name was probably 生きている森 "the living forest",
  which has first-half, past and second-half debug loadouts (section 3). So past and present scorpions aren't ruled
  out, but nothing ties either scorpion soul, Tark or Najka to that stage.

## 1. Retail: who has two souls

From the developers' names in PC 3/14: `BossBattleParam` (fights), `ItemLotParam` (soul lots) and the soul working names
([Network Test bosses not in retail](nt-bosses-not-in-retail.md#7-the-developers-own-names-pc-pre-release-2026-10-28) section 7).

| Boss | Fight row | Soul lot | Soul |
|---|---|---|---|
| Last Giant | 1010010 m10_10 [新・巨人上級兵士（亡者）, "new elite giant soldier (Hollow)"](boss-region-names.md#every-region) | 309600 朽ちた巨人の森：…（亡者）, "Forest of Fallen Giants" | 64010000, working name 巨人王亡者 "Giant King (Hollow)" |
| Giant Lord | 2010000 m20_10 新・巨人上級兵士（生者）, "(living)" | 309700 巨人の記憶：…（生者）, "the giants' memory" | 64240000, working name 巨人王生者 "Giant King (living)" |

The giants' memory's camera row is 2010000 巨人の記憶（過去）, "the giants' memory (past)". m10 maps are the present and m20
maps the past ([E3 2013 demo leftovers](e3-demo-leftovers.md#9-past-and-present-the-other-time-labelled-areas-added-2026-11-03) section 9). So the developers treated the Last Giant and the Giant Lord as one
character in two eras. He has two bodies (c3096 / c3097) and gets a soul in each. No other retail boss does this.

For comparison:
- **Throne Watcher and Defender**, a duo: one lot (332000 火継ぎの玉座１：王妃の騎士A) gives both souls, 64260000 and
  64270000, which are neighbouring ids.
- **The Dragonrider** is fought in both eras: alone in the present (Heide, m10_31, lot 611000) and as the twins in the
  past castle (m20_21, fight row 1021000 重騎馬兵：兵士のみ 2体戦, lot 309610 王城ドラングレイグ：重騎馬兵). Both lots
  give the same soul, 64020000. So the same body in both eras gets one soul.
- **Vendrick** has one soul. Its lot is 514500 不死廟：王（現代）, "Undead Crypt: King (present)", although the crypt's
  camera row is 不死廟（過去）, "(past)". The era labels were already mixed up by 2014.

## 2. The NT soul list

The 27 NT souls are names only. No NT `ItemParam` row exists for any of their ids, and no NT param holds any of them
(word search over every NT regulation param, 2026-11-04). So only the names and their order link a soul to a boss.

| Pair | Ids | Next to each other | Reading |
|---|---|---|---|
| larval naga / adult naga | 60734000 / 60746000 | no | **Mytha** (retail AI label and soul name ナーガ). Two growth stages. Her area, m10_17 炭鉱の村 "mining village", has no past version in the map list, the camera rows or the debug loadouts |
| Hell Scorpion / Death Stalker | 60736000 / 60737000 | yes | unknown. The only scorpions in the game are the man-scorpions |
| demon hand / cursed hand | 60748000 / 60749000 | yes | unknown; no body |
| phantom knight / phantom sorcerer | 60751000 / 60752000 | yes | perhaps the Imperious Knight and Leydia Pyromancer families (retail AI ファントムナイト / ファントムメイジ) |
| crimson-flame / black-flame lizard | 60730000 / 60741000 | no | the salamander family? |
| King Vanclad / Queen Dunashandra | 60753000 / 60754000 | yes | two bosses, a married couple |

**Eras.** King Vanclad has three bodies in the developers' names: past (c5130), present (c5140 / c5145) and
transformed (c6160). He has one NT soul. The Queen has [past and present bodies (7030 / 7035, 7020)](community-cut-sheet-crossref.md#bosses) and one soul. The
Giant King, retail's one two-soul boss, has no NT soul, unless he is 岩の巨人カノッソフ "rock giant Kanossof". So the NT
list was not built as one soul per era.

**A weak pattern in the order.** One boss in two forms gets its second soul further down the list: Mytha's two are 12
places apart, and retail's Giant King's 23. Two bosses fought together get neighbouring souls: retail's Throne Watcher
and Defender. By that pattern, the neighbouring scorpion pair reads more like two bosses than one boss in two eras.
That is two or three cases, so only a hint.

## 3. The Shaded Woods had a past stage

- **The working name.** Giant basilisk row 222001 is named 巨大バジリスク：色②：虚ろな影の森 "Shaded Woods" in
  `EnemyParam` and 巨大バジリスク：色②：生きている森（後半） "living forest (second half)" in `EnemyDefaultLogicParam`. v1.00
  places 222001 once, in m10_32. So 生きている森 "the living forest" is probably the Shaded Woods' working name
  ([E3 2013 demo leftovers](e3-demo-leftovers.md#9-past-and-present-the-other-time-labelled-areas-added-2026-11-03) section 9 had it as an unknown area).
- **A past visit between two halves.** The debug loadouts (`PlayerStatusItemParam`) list it three times. The list
  starts at Things Betwixt (化石の森) and reads like a route through the game:
  - 300111000 生きてる森（前半） "first half";
  - 300112000 岩峰城（現代） Rock-peak Castle (present);
  - 300113000 生きてる森（過去） "**past**";
  - 300114000 抵抗の砦（過去） Fort of Resistance (past);
  - 300115000 生きてる森（後半） "second half".
- **A stone guardian.** Default-logic row 213005 is named 石守：生きている森（過去）. But that family's row ids were
  reused for other maps: 213005's `EnemyParam` name is Iron Keep, and v1.00 places it there. So this row only shows
  that the label existed.
- **Retail kept only the present.** The Shaded Woods camera rows are 1032000 現代-虚ろの影の森-汎用 and 1032010
  現代-虚ろの影の森-霧の森, "present: Shaded Woods: general / misty forest". No map list (NT or v1.00) has an m20_32.
- **Caveat.** The map list calls m10_32 獣人の森 "beast-folk forest", not the living forest. One row (the basilisk)
  links the two names, so this is probable, not certain. The NT's line about a forest that became a demon
  (77701100) may be the same place. Unconfirmed: m10_32's object names include 人面木 A / B "human-faced trees" and
  森の番人（検証用） "forest guardian (test)".

**For the Tark comment.** The forest Tark and Najka live in did have a past stage planned. That is more than the
"no evidence" in [Tark and Najka duo claim](tark-najka-duo-claim.md) allowed for. But nothing puts the man-scorpions in it:
- every man-scorpion row is named 虚ろの影の森 alone;
- no row names one with 過去 "past" or 生きている森;
- the two scorpion souls are tied to no body.

## Method

Scratch scripts (session scratchpad, not kept):
- NT `ItemParam` rows for the 27 soul ids, and a word search for them over every NT param;
- PC 3/14 `BossBattleParam` names, and `WorldMapList.list` for nt and vanilla;
- every PC 3/14 item lot holding a boss soul, 64000000-64330000;
- PC 3/14 row names containing the forest names, and the `CameraExFollowParam` past / present rows;
- 2130xx / 2220xx names, with v1.00 placements from `DS2 Scholar PC\tools\bosses_pc.generators("vanilla")`;
- `PlayerStatusItemParam` names 300090000-300200000.
