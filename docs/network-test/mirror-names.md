# "Mirror" names: the Looking Glass Knight, his servants and the Ruin Sentinels

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-11-03. The author asked whether the Network Test (NT) called the Looking Glass Knight "Mirror Knight", what the
earliest reference to the Ruin Sentinels as "knight in the mirror" is, and whether any other enemy is "in the mirror".
Builds searched: the JP NT (NPJB90625; every NT name below is in its base GameData, packed 23 Aug 2013, not only in the
2 Oct update's loose files), the US NT (NPUB90979, 17 Sep 2013; the same hits), v1.00 (BLUS41045), PS3 Scholar
(BLUS31528) and PC Scholar (1.0.3.0), plus the developer row names of the PC pre-release build of 31 Mar 2014.
Translations are mine. `tools\mirror_names.py` prints everything below.

## Short answer

- **Yes, "Mirror Knight".** The NT's English `charaname` 504000 reads "Mirror Knight", the only English name the NT gives
  him. The Japanese entry for the same id is 鏡の戦士 "mirror warrior"; his NT soul is 鏡の騎士 "mirror knight", the
  Japanese name retail kept. The developers' own name, in scripts and rows from the NT to the 2014 PC build, is ミラーナイト.
- **The Ruin Sentinels are 鏡の中の戦士, "warrior in the mirror"**: 戦士 is warrior or fighter, not 騎士 knight. No build
  has 鏡の中の騎士, and no English text in any build says "in the mirror". The earliest is the JP NT's base data
  (23 Aug 2013): two AI labels and the material entry of their model c3250.
- **No other enemy is "in the mirror".** The only other such name is 鏡の中の剣士 "swordsman in the mirror", a behaviour
  mode in the AI of the Mirror Knight's servants, listed next to a mode also called 鏡の中の戦士 (section 3).

## 1. The Looking Glass Knight (c5040)

| Where | Network Test | Retail v1.00 |
|---|---|---|
| English boss name (`talk\charaname`) | 504000 **"Mirror Knight"** | 5040 "Looking Glass Knight" |
| Japanese boss name | 504000 鏡の戦士 "mirror warrior" | 5040 鏡の騎士 "mirror knight" |
| Soul (`ItemName`) | 60738000 鏡の騎士のソウル, Japanese only, in no item lot | 64180000 鏡の騎士のソウル / "Looking Glass Knight Soul" |
| AI scripts | `ai504000` ミラーナイト, `ai504010` 上位：ミラーナイト | the same |
| Material | "c5040　ミラーナイトの盾用" (for the Mirror Knight's shield) | the same |
| Test AI | `ai009983` (下位：吉田テスト用, "for Yoshida's tests"): states 【テスト】ミラーナイト and 【テスト】ミラーナイトバレットテスト "Mirror Knight bullet test" | gone |
| Armour, Japanese | none | 鏡の戦士の仮面 / 鎧 / 手甲 / 足甲, "the mirror warrior's mask" etc.; the descriptions say the trial is to fight the 鏡の戦士 (English: Looking Glass set) |

The NT does not place him (its one map is Huntsman's Copse). Retail keeps ミラーナイト in developer names: the m20_21
boss-music region and developer signs (`boss-region-names.md`, `developer-signs.md`), the m20_21 event state
ボス：ミラーナイト_戦闘, the Aldia's Keep events ミラーナイトの間 "the Mirror Knight's room" and ミラーナイトの鏡_1 / 4 / 5
"the Mirror Knight's mirror", and in the 2014 PC rows: `BossParam` 504000, `ChrNetworkPhantomParam` 14 ミラーナイト霊
"Mirror Knight phantom" (the summoned player's phantom type; retail English "mirror squire", Japanese 鏡の従者), the soul-
weapon shop rows 鏡の騎士：両手剣 / 大盾, and two map objects 【岩峰城】ミラーナイトが壊せる柱 "[rock-peak castle] pillar the
Mirror Knight can break" (`MapObjectParam` 212000) and 【岩峰城】鏡 "mirror" (219051). 岩峰城 "Rock-peak Castle" is the
developers' working name for Drangleic Castle (m20_21); the E3 2013 demo's castle map m20_63 was "E3 Rock-peak Castle
(past)" (`e3-demo-leftovers.md`, which also compares the NT's Mirror Knight with retail's).

The old text dump `..\DS2NT\Unpacking stuff\English_Text\English\talk\charaname.txt` lists "Mirror Knight" under 7026,
7404 and 9150. `fmg.py` reads the file with "Mirror Knight" at 504000 only, so that dump's ids look misaligned.

## 2. The Ruin Sentinels (c3250): 鏡の中の戦士

| Build | Where the name appears |
|---|---|
| JP NT, 23 Aug 2013 (earliest) | `ai325000` 鏡の中の戦士; `ai325010` 上位：鏡の中の戦士：汎用 "upper logic: generic"; material "c3250　鏡の中の戦士　金色" (golden) |
| US NT, 17 Sep 2013 | the same |
| v1.00 | also `ai325020` 上位：鏡の中の戦士：通常戦用 "for ordinary fights"; m10_16 boss-music region ボス戦BGM範囲_鏡の中の戦士; m10_16 event states ボス：鏡の中の戦士_戦闘 and 再生の種火06_ボス鏡の中の戦士後の食堂 (bonfire 06, the dining hall after the boss) |
| PC pre-release, 31 Mar 2014 | developer row names: `BossParam` 325000 / 325010 (Lost Bastille / Drangleic Castle), `ChrModelParam` 3250, the soul 虚ろの衛兵のソウル（鏡の中の戦士のソウル）, four armour rows 13250100-103 marked （作成中） "in progress", attack rows (light / heavy sweep, light / heavy overhead, thrust, shield throw), chest and corpse lots in the 鏡の中の戦士部屋 "warrior-in-the-mirror room", and a drop row for an ordinary Sentinel in Drangleic Castle (【王城：通常】鏡の中の戦士-ザコ) |
| PS3 Scholar | as v1.00 |
| PC Scholar | as v1.00, plus the Drangleic Castle event 小白霊：鏡の中の戦士 "small white phantom: warrior in the mirror" |

Shipped names: English "Ruin Sentinel", Japanese 虚ろの衛兵 "hollow guards". A web search on 2026-11-03 found no community
source calling them "knight in the mirror".

## 3. The only other "in the mirror" name: 鏡の中の剣士 "swordsman in the mirror"

It is a lower logic (behaviour mode) in `ai730000.edd`, the debug-name file of the servants' lower script, with its own
target-behaviour and attack-behaviour states. The script's lower logics, in file order: 特大剣・中盾 (ultra greatsword,
medium shield), 長斧・中盾 (long axe, medium shield; its target state says 長斧・盾無, no shield), **鏡の中の剣士**,
**鏡の中の戦士**, 特大剣・中盾_2, then 橋専用 "bridge only" (NT) or 狩猟の森 "Huntsman's Copse" (v1.00), and generic
ones (start, settings, navigation dash, search, lock-on dash, guard). Present from the JP NT to PS3 Scholar; no AI
debug-name file turned up on PC Scholar at all. No text, model or row is named after it.

**The servants.** The 2014 PC rows name ミラーナイトの下僕１ / ２ "the Mirror Knight's servant 1 / 2" (NT `npcMenu` 7300 /
7310, Japanese only), each in three loadouts: 特大剣 ultra greatsword (730000 / 731000), 両手長斧 two-handed long axe
(730010 / 731010) and 片手剣 one-handed sword (730020 / 731020), with attack, behaviour, logic and weapon rows. Enemy rows
exist only for servant 1 with the greatsword and servant 2 with the long axe: 730000 / 731010 in the Mirror Knight fight
(m20_21, upper script `ai730010`) and 730001 / 731011 in Aldia's Keep (m10_15, `ai730030` 上位：ミラーナイトの下僕（アンディール）),
the same in v1.00, PS3 Scholar and PC Scholar.

**The NT's Copse swordsman.** NT spawner 81000000 in Huntsman's Copse places enemy 810000 with the upper script
`ai730020` 上位：ミラーナイトの下僕（狩猟の森） "the Mirror Knight's servant (Huntsman's Copse)" over `ai730000`. The NT's
`npcMenu` 8100 is マルチ用NPC：剣士（男） "multiplayer NPC: swordsman (male)". Retail's Copse has no such spawner (retail 8100
is Masterless Glencour), and PC Scholar has no `ai730020` (one of the 51 PS3 scripts it dropped, `..\..\DS2 Scholar
PC\findings\cut-content-pc.md`).

**Reading (not confirmed):** "swordsman in the mirror" is probably the one-handed-sword loadout, perhaps the NT's Copse
swordsman. The script picks its lower logic by number, with no label at the choice, so this is not tied down. Whether
the servants' "warrior in the mirror" mode reuses the Sentinels' behaviour was not checked.

## Reproduce

`uv run --no-project --python 3.12 python -I -X utf8 tools\mirror_names.py [game ...]` (from this folder's parent;
default all five builds, about 10 minutes, mostly v1.00 and PS3 Scholar). It prints each build's AI labels, every
archive string holding one of the mirror terms (the shared summon states of the 417 hollow scripts are only counted),
the map spawners of the mirror families, and the pre-release developer row names.
