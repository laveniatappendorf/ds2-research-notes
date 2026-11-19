# Dark Souls II PS3: Japanese disc (BLJM61113) vs US disc (BLUS41045)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

First broad pass, 2026-10-27, for the author. Question: is there anything beyond menu localisation? **Yes, but not
localisation at all: the Japanese disc is a later build.** Its game data was packed on 31 Jan 2014 (US: 27 Jan), and
almost everything that differs is an early cut of what the US got in title update 1.01 (data 24 Feb 2014). A few
fixes the US vanilla line only got in later patches, or never (they came back in Scholar of the First Sin).

Tools (`tools\`, run with `uv run --no-project --python 3.12 python -I -X utf8 tools\<tool>`): `build_compare.py`
(every file both discs load; `files_vanilla_vs_jp.tsv`), `timeline.py` (when each Japanese change reached the US
patches; `timeline_jp_changes.tsv`), `text_diff.py`, `mapparam3.py`, `pieces3.py` (pieces inside split map archives),
`piece_detail.py`, `eboot_compare.py`. Regulation: `..\DS2 Vanilla Patches\tools\regulation_compare.py`
(`regulation_vanilla_vs_jp.tsv`, `regulation_jp_vs_1.01.tsv`). Shared tools read this build with `DS2_GAME=jp`.

## 1. The disc

- Same 152 files as the US disc. **136 are byte-identical**: all sound and voice banks (English voices on both), the
  intro movie, every font (both discs carry Japanese, Chinese, Korean and all European fonts), shaders except one,
  the trophy file, the system update.
- The 16 that differ: EBOOT, GameData.bdt / .bhd, InstallData.bdt / .bhd, regulation, the effect bank
  (sfx9999 + its models), SfxShader.bnd, the server list (`JP0506-BLJM61113_00`, own online pool), PARAM.SFO,
  LIC.DAT, PS3_DISC.SFB, and the XMB pictures PIC0 / PIC2 (the logo with the CERO D rating mark).
- **Text: both discs ship all 11 languages.** The US disc already has the full Japanese text; the Japanese disc has
  English, French, German and the rest. Same file list, same 11,452 archive names. Which language shows is not a
  data difference (presumably the console language; not checked in code).
- PARAM.SFO: disc `VERSION` 01.02 (US 01.01, Asian 01.00), `APP_VER` 01.00 on all three; parental level 8; no 576p.
- [Build stamps](../mechanics/agility-timeline.md#1-build-dates-the-binder-stamps): GameData index `14B31Q42` = 31 Jan 2014 16:42 (US `14B27P30` 27 Jan 15:30); InstallData 30 Jan
  21:10 (US 24 Jan 06:16); EBOOT file 31 Jan 08:24.
- **Regulation version 00010100.** US v1.00 = 00010000, patch 1.01 = 00010200: the US patches skip 00010100, and this
  disc is it.

## 2. What changed: 173 of 11,457 files (+1 repack)

| against patch 1.01 | files | meaning |
|---|---|---|
| same as 1.01 | 142 | a change the US got a month later in 1.01, byte for byte |
| 1.01 has a third version | 22 | mostly the same content repacked; a few where 1.01 went further (below) |
| 1.01 still has the US copy | 10 | Japanese-only at the time (section 4) |

And the other way: **80 files 1.01 changed are still the US v1.00 copy here** (section 3).

### Already in the Japanese disc (= 1.01)
- **Balance**: 36 of 1.01's 54 changed rows plus its 4 new rows; 1.01's regulation = this one + 18 more rows. Details
  in [v1.00 vs patch 1.01](v1.00-vs-1.01.md#2-balance-regulation-calibrations-100-102-and-effect-scripts) section 2; names now identified (from npcMenu / charaname,
  matched by id; the attack-id prefix rule 1 + character number is inferred):
  - enemy 769000 (AI label 奇跡の人 "miracle person") = **Licia of Lindeldt** (NPC 7690): HP 2,440 -> 4,580,
    defence 338 -> 581, souls 1,000 -> 2,500, damage x1.68;
  - attacks 160201010-040 = **Demon of Song** (c6020 唄うデーモン): damage cut 10-50 %;
  - projectile 177100510-513 = **Royal Sorcerer Navlaan** (NPC 7710; his AI label is 封じられた人 "sealed person"):
    3-shot homing volley -> one bullet that spawns two child bullets;
  - attacks 112402810 / 910 = 暗殺者（生者） "assassin (living)" (c1240): damage control id changed;
  - **Soul Geyser** (31170010-13) and the Lightning Spear family: BulletParam flags (meaning unknown). Not the
    damage nerf [the early PC build](early-pc-builds.md#31-march-pre-release) has;
  - Ladder Miniature use id; the four gesture Carvings can no longer be sold (200 -> 0 souls).
- **Chloranthy Ring and Green Blossom stamina buffs** (both effect scripts = 1.01's).
- **Bell Keepers' new model** c7530 and all their armour (both sexes, both detail levels).
- AI "evasion cancel" state in **4 of 1.01's 16 AI scripts**: two-sided warrior (snake head), warrior in the mirror,
  spider queen, heavy cavalry (soldier only).
- NPCs: Gilligan's new death event in Majula, Rosabeth's muted line, McDuff's generator field (and the Bastille
  event fix that goes with it), Tark's line swap.
- Player animation network (c0001.nmb); the Memories of the giants objects and filter; No-man's Wharf and Shaded
  Woods layout edits; collision re-bakes of 1.01: Heide's Tower (l10_31) 3 of 3, Drangleic Castle 4 of 4 (+ map
  piece m7000), Lost Bastille 2 of 4, Dragon Aerie 1 of 3, Shaded Woods 1 of 2; the Majula, Aldia's Keep (m10_15) and
  Gutter event scripts.
- Text: every changed English (79) and Japanese (84) entry is word for word 1.01's (other languages 10-78 entries
  each, not read entry by entry): placeholder
  subtitles ("(no line)", "(N/A)", "（字幕なし）") removed, "long ago long ago", "Increases spell uses, but at the cost
  of HP" (Northern Ritual Band), "online ID", Japanese typo fixes, Ornifex's ヴェンドリック -> ヴァンクラッド,
  古龍 -> 古竜 in the Drakewing description.
- Effects: 80 of the 84 changed effects in sfx9999 and all 8 changed point-sprite shaders = 1.01's.
- Program: `invaded_count` added, `InitLadder` and two ladder-move state-machine names gone (both as in 1.01).

## 3. Not yet in the Japanese disc (1.01 only)

- 18 regulation rows: bonfire lit flags (Doors of Pharros, Grave of Saints), character-creation face slider ranges,
  two online timings, EnemyParam Unk02 for 9 enemies, one enemy weapon.
- **Bell Keeper belfry logic**: the Lost Bastille / Iron Keep event scripts and EventLocation rows (so the Japanese disc
  has the new Bell Keeper model but the old belfry behaviour).
- 12 of the 16 "evasion cancel" AI updates: dullahan, Vagrant (Pursuer), man-scorpion (Tark), grave warden,
  kingdom captain (+ its white phantom), moonlight old man, Patches, female knight, **Navlaan (and his ladder
  climbing)**, the generic white-phantom AI (with its "white door failure").
- Animation networks c1020 (hollow soldier) and c1250 (undead citizen); collision in Forest of Fallen Giants
  (l10_10), Iron Keep (l10_19), Memories (l20_10), and the remaining Bastille / Dragon Aerie / Shaded Woods pieces.
- Forest of Fallen Giants event script (event_m10_10).
- Text: the Steam strings, TitleFlow placeholders, 召還 -> 召喚, the 古龍の誓約 covenant message.
- Program: no title-update layer (`patch:`, `gamedata_patch:` ...), so like US v1.00 it can only run disc data; build
  path still `n:/dantelion2_localbuild` (1.01: `Stable_10_0_FRPG2_Title`); online content file **contents_0104.bin**
  (US disc 0101, 1.01 0105).

## 4. Japanese-only changes (not in the US 1.01)

| change | what | US vanilla line |
|---|---|---|
| Throne Defender AI (ai332010, 王妃の騎士Ａ) | one argument 1 -> 0 in four commands (1:7013 / 1:7000) | same change in **1.10** (Nov 2014) |
| Things Betwixt event script | command 1:140111(8, x) -> 1:140113(8, x, 1) | in **1.03** (Apr 2014), with an added 1.5 s wait state |
| Majula collision `l02_3300`, `l02_3700` | low-detail collision pieces rebuilt (+1.1 KB, +1.2 KB) | **never** (1.10 still US) |
| Lost Bastille map pieces `m0060` (by its breakable "Destruction" wall), `m0151` + high-detail collision `h02_0700`, `h02_2500` | a few vertices each: geometry seams patched | **never** |
| Shaded Woods collision `l01_0200` | an intermediate rebuild; 1.01 has a later one | superseded in 1.01 |
| 20 effect models (sfx9999res) | small edits | US kept v1.00's until 1.08 shipped its own |
| Lost Bastille GeneratorParam | byte-identical to 1.03's file (content = 1.01) | - |

**PS3 Scholar has the Japanese versions of all six Majula / Bastille pieces**, so these fixes went into the main
branch on 31 Jan 2014 but the US vanilla patches never shipped them (patches re-ship only what each patch flagged).

## 5. Reading

The Japanese master was cut four days after the US one, from the same tree, and caught the first wave of fixes that
became patch 1.01 (balance, Bell Keeper models, placeholder text), plus a few that the US line only picked up later or
never. No content was added, removed or censored for Japan; no Japan-only text. Practical upshot: an unpatched Japanese
disc plays like a half-way US 1.01.

## Open

- Bounding locations of the Majula / Bastille geometry fixes in game (what gap or stuck spot they close).
- What BulletParam Unk14 / 16 / 17 do (Soul Geyser, Lightning Spear).
- Whether the EBOOT restricts menu languages by region (code not read; no Ghidra import of this EBOOT yet).
- Code-level EBOOT diff (Japanese +12 KB code over US v1.00, 1.01 +46 KB).
- The Asian disc (BLAS50640): done 2026-10-27, [Asian disc vs US disc](asia-disc-vs-us.md): the US v1.00 game byte
  for byte, mastered first (27 Jan). Its "different GameData.bdt" in the first hash pass was the showcase patches in
  the RPCS3 copy of the US ISO, not the Asian disc. The same applies to section 1 here: the hash list compared against
  that patched ISO, but the Japanese GameData.bdt differs from the original as well, and every file comparison above
  used the clean `DS2 Vanilla` copy.

## Related pages

- [V1.00-only content](v1.00-only-content.md): the v1.00 content that shipped only on the US and Asian discs, because this later master already had the 1.01-level files.
