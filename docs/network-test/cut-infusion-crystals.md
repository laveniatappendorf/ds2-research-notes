# The Network Test's two infusion crystals (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Two items in the Network Test's infusion set that retail dropped: **五色の結晶** ("five-coloured crystal", 61040000)
and **虹色の原結晶** ("rainbow raw crystal" / "rainbow source crystal", 61050000). Background:
`scaling-letters-and-infusion.md` (infusion in the Network Test) and `break-element.md` (the cut Break stone).
Images: `cut-infusion-crystals-icons.png` (tool `tools\icon_sheet.py`), `cut-infusion-crystals-vs-magic-stone.png`.

## What the Network Test has

- **Text:** Japanese only (the US and JP Network Tests, every JP update; no English or other language). Both share one
  short description, 装備に属性を付与するための触媒となる石 "a stone that acts as the catalyst for giving equipment an
  attribute"; no long description. The attribute stones beside them each say "gives equipment the X attribute".
- **Item rows:** both have ItemParam rows, but every row in that block (titanite, dragon bone, crystals, stones) is
  the same template with only the id changed, so price, use and sort order tell nothing. The block, in order:
  鋼石の欠片 / 塊 / 結晶 (Titanite Shard / Chunk / Slab), 純粋な黒鋼石 (Twinkling Titanite), **古代の銀鋼石 "ancient
  silver titanite" (cut)**, **龍の骨片 "dragon bone fragment" (cut)**, 龍の化石 (Petrified Dragon Bone), the two
  crystals, then the seven attribute stones (magic, lightning, fire, dark, poison, bleed, Break).
- **Where they come from:** the five-coloured crystal has no source anywhere. The rainbow crystal has one: item lot
  **10180**, one crystal at 100 %, developer name **鐘守灰霊時-成功帰還（ホスト死亡）** "as a **Bell Keeper** grey
  spirit: successful return (the host died)" (SmithBox DS2S developer row names). Its neighbours (10000-10240) are
  the other online rewards (Awestone, Sunlight Medal, Tokens of Fidelity / Spite, Dragon Scale, Bonfire Ascetic,
  cracked eye orbs). Nothing else refers to either id: no shop, treasure, event, talk or map entry (searched every
  regulation param and 1,295 param / ezstate / map / event files in GameData).

## What retail keeps

- No item rows, names or lots. Lot 10180 (Bell Keeper reward) became **1 Titanite Chunk** in v1.00; PC widens it to
  Titanite Chunk 90 / Slab 5 / Twinkling Titanite 3 / Petrified Dragon Bone 2.
- Infusion in retail costs one stone and nothing else (CustomAttrCostParam: one item + amount per infusion), so
  there is no catalyst step.
- **The icons survive** (v1.00 PS3 and PC GameData; the Network Test itself doesn't ship them): `IC_0061040000`,
  `IC_0061050000`, plus the Break stone `IC_0061120000` and the two cut materials `IC_0061010000` / `IC_0061020000`.
  The five-coloured crystal is a small iridescent pink / blue / green gem; the rainbow crystal a larger opal-like slab.
- **The rainbow crystal's picture is the retail Magic Stone's** (61140000, the Enchanted infusion): the same artwork
  one pixel apart (mean colour difference 74 as stored, 26 after a one-pixel shift, the rest re-encoding noise). The
  Break stone's icon is a clear quartz cluster.

## Guesses (not established)

- A two-tier catalyst: a common five-coloured crystal and a rarer rainbow one (a PvP covenant reward), each perhaps
  setting how strong the infusion is. That would fit the Network Test's WeaponReinforceParam, which has **two**
  element-share blocks where retail has one, the second always double the first (30 / 60 on most weapons, 25 / 50
  staves, 20 / 40 bells). Not traced in code.
- A rainbow is traditionally seven colours in Japanese (七色), and the Network Test has seven attribute stones; what
  the "five colours" stand for is unclear.
- Whether the Magic Stone took the crystal's art, or both were cut from one source image, can't be dated from the
  files (the PS3 TPFs carry no timestamps).
