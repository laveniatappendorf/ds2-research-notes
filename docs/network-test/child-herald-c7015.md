# The child Emerald Herald model (c7015), and whether it is the "sparkling girl" (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author's question: retail v1.00 reportedly ships a model of a child thought to be the Emerald Herald. Is it the same
character as the Network Test's "sparkling girl", which might share the Herald's id?

## Short answer

- **Yes, retail has the model: `c7015`**, a 1.21 m girl. It is one of the three character models no enemy places
  (`unused-models.md`). Its textures are named `c7010` (`BD_ / FC_ / HR_ / EYE_c7010_F`).
- **It is the Herald's child version by every id the files use.** The Herald's numbers come in a matching pair: adult
  NPC 7000 -> model c7005 -> enemy rows 7005xx, and child NPC 7010 -> model c7015 -> enemy row 701500. Retail's own
  NPC name list calls **both** 7000 and 7010 "Emerald Herald" (緑衣の巡礼).
- **It is not the sparkling girl.** キラキラ娘 is NPC 7860, retail's Dyna & Tillo, with an invisible body (c2250)
  (`sparkling-girl-dyna-tillo.md`). The Network Test character in the Herald's child slot is 龍の巫女（子供） "dragon's
  shrine maiden (child)". The earlier memory that the sparkling girl had the Herald's id mixed the two up.
- **The Network Test already had this character, but not the model.** It has her name, her AI scripts and her enemy
  row pointing at body 7015, but no `c70xx` model file at all. Retail v1.00 adds the model, textures, animations
  and sound bank.
- **She is very likely the cut `m20_04` girl** (the child the "slave merchant" sells; `..\..\DS2 Scholar\findings\cut-content\cut-dialogue-translation.md`).
  That girl's talk ids are 70100xxx, i.e. NPC 7010, the same slot. This is by the id rule only: nothing places c7015
  in a map, so no file shows the model speaking those lines.
  **A text link too (2026-10-27):** the Network Test's language-less draft talk folder has the *adult* Herald recognise
  the player ("could you be the one from that time!?") and recite the same incantation the `m20_04` girl teaches "the
  one who saved me", worded for "this cursed one" (`herald-lines-to-1.03.md` section 9).

## The id pair, build by build

| | Network Test (US) | retail v1.00 |
|---|---|---|
| `npcMenu` 7000 | 龍の巫女 "dragon's shrine maiden" | 緑衣の巡礼 / Emerald Herald |
| `npcMenu` 7010 | 龍の巫女（子供） "dragon's shrine maiden (child)" | 緑衣の巡礼 / Emerald Herald |
| AI scripts, adult | ai700000 / 700010 巫女 "shrine maiden", ai700500 龍の巫女, ai700510 "…in-game version: general" | ai700000 / 700010 only |
| AI scripts, child | ai701000 / 701010 巫女（子供）, **ai701500 龍の巫女（子供）** | ai701000 / 701010 only (701500 gone) |
| `EnemyParam`, adult | 700000 (body 7000), 700500 (body 7005) | 700500-700504 (body 7005) |
| `EnemyParam`, child | 701000 (body 7010), **701500 (body 7015)** | **701500 (body 7015)** |
| other child rows | EnemyBehaviour* / Move / Common / Attack 701000, 701500; LogicCom 701010; EnemyEquipWeapon 70100000; NpcEvent 70100000 | the same, plus EnemyDefaultLogic 701000 / 701500, EnemyEquipWeapon 70100000; NpcEvent 70100000 dropped |
| child model | **none** (the NT ships only 18 character models, no `c70xx`) | `c7015` model, textures, `c7015.anibnd`, `c701500` TAE, `sound\frpg2_c7010.bnd` |
| child held item | `WeaponParam` 7000000 only | `WeaponParam` 7010000, model `WP_7010_M` |
| face parts | `FG_7010_M` (+ `_FG`, `_L`), icon `IC_FG_7010_M` | the same |

"Body" = the first word of the `EnemyParam` row (the model number). Neither build's `EnemyParam` fits the shared
decoder (NT 320-byte rows, v1.00 356), so only the first words were read.

## The model

`models_c7005_c7015.png` and `textures_c7000_vs_c7010.png` in `child-herald\`:

| | c7005 (adult Herald, placed) | c7015 (child, unplaced) |
|---|---|---|
| size | 1.14 x **1.58** x 0.36 m | 0.84 x **1.21** x 0.46 m |
| triangles / bones | 9,254 / 182 | 6,943 / 111 |
| cloth physics | yes (`c7005_Cloth`) | yes (`c7015_Cloth`) |
| textures | `c7000` set + `feather_c7000_F` | `c7010` set |
| animations | 60 | **79** |

- **The textures match the adult's.** Both have auburn hair, and the face sheets share a layout. Both eye sheets carry
  a brown iris **and a second violet one** in the same arrangement. In the material-colour render she wears a short
  cape with a ruffled collar, a skirt and boots (the collar's real look needs the textured model; the render uses flat
  colours).
- **She was fully rigged and animated**: idle, walk, jog, side-steps, turns, roll landing, 40-odd action slots
  (`a65_*`), and a second set of walk / jog / side-step animations "holding item 1" (`Wep01`). That is more than the
  adult Herald has. So she was built as a working NPC, not as a sketch.
- **Her held item** `WP_7010_M` is a 10 cm object (material `o02_8500_00`): a round body with a small knob on top and
  a fringe of strands below. A pouch or small doll, perhaps; not identified. The adult's `WP_7000_M` is a 0.8 m
  fan- or leaf-shaped blade on a thin stem (`held_items_WP_7000_7010.png`).

## Later builds

PS3 Scholar keeps c7015 unchanged (`..\..\DS2 Scholar\findings\cut-content-scholar.md`). PC Scholar dropped it, with
c2020 and c6031, because no enemy uses it (`..\..\DS2 Scholar PC\findings\cut-content-pc.md`). The `m20_04` girl's
lines left the game in patch 1.03 (`..\..\DS2 Vanilla Patches\findings\cut-talk-timeline.md`).

## Not known or not checked

- Whether any map ever placed her: no generator in v1.00 names enemy 701500 (`unused-models.md`). The NT places neither row.
  **Intended map found 2026-10-28**: the PC pre-release names EnemyParam 701500 龍の巫女（子供）＿インゲーム用：アン・ディールの館,
  "in-game version: **Aldia's Keep**" (701000 is the ＿カットシーン用 cutscene version) (`community-cut-sheet-crossref.md`).
- What `sound\frpg2_c7010.bnd` holds (child voice or foley); not opened. **The credits are confirmed (later the same
  day, `herald-lines-to-1.03.md`)**: v1.00 `Staffroll.fmg` lists "Shanalotte (Child)" (Amy Nolan) apart from
  "Shanalotte" (Ruth Negga), and the Network Test's speaker list `talk\charaname.fmg` names both 7000 and 7010
  シャナロット. That ties the cut `m20_04` girl's lines (speaker 7010) to Shanalotte by name.
- The NT's `o23_7000` / `o23_7005` / `o23_7010` objects share the numbers; not looked at, and possibly coincidental.
- The JP network test was not checked (its params are expected to match the US NT's).
- The c7015 animation names say what slots exist, not what the motions look like; none were played.

Scratch scripts (session scratchpad, not kept): a FLVER survey of the c70xx models, an id trace across archive names /
params / AI labels / `npcMenu`, `chrsheet.py` for the model render, `ps3tpf.py gamedata` for the textures, and a
held-item render.
