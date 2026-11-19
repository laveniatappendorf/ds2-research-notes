# The wiki's "unknown" upgrade materials and the Network Test (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author pasted [the community wiki's four cut "Upgrade Materials" icons](../network-test/wiki-unused-content-crossref.md) (three marked "Unknown"; one "appears to be a
smaller fragment of a Magic Stone"). Question: does the Network Test point to what they are?

**Yes: all four are Network Test items.** Retail keeps their icons under the demo's item ids, but nothing else.
Image: `unused-material-icons.png` (wiki crops left of the bar, game files right; crops in `unused-material-icons\`).
Tool: `tools\wiki_icons.py compare materials` (the ring tool, renamed; see [unused ring icons](unused-ring-icons.md)).

| wiki | icon file (v1.00 PS3 and PC) | match | Network Test item | Network Test description |
|---|---|---|---|---|
| 1, grey rock with green | IC_0061010000 | 0.97 | **古代の銀鋼石** "ancient silver titanite" | 装備を強化する "reinforces equipment" |
| 2, bone shard | IC_0061020000 | 0.93 | **龍の骨片** "dragon bone fragment" | デーモン装備を強化する "reinforces **demon** equipment" |
| 3, iridescent gem | IC_0061040000 | 0.98 | **五色の結晶** "five-coloured crystal" | 装備に属性を付与するための触媒となる石 "a stone that acts as the catalyst for giving equipment an attribute" |
| 4, clear quartz cluster | IC_0061120000 | 0.96 | **溶解石** "dissolving stone" | 装備に破壊属性を与える石 "a stone that gives equipment the Break attribute" |

Matching: template match (OpenCV, colour, several scales) of each wiki crop inside all 3,055 distinct item icons of
every dumped build. Each found its file at 0.93-0.98, and the next best was 0.77 or lower. Icon 4 needed a tighter
crop, because the first one took in the table's cell background. All four pairings were checked by eye. The text is
Japanese only (`jp_ItemName` / `jp_SimpleExplanation`); the long descriptions are the placeholder "No Text Settings".

## What the Network Test says about each

**The demo's upgrade-material block** (ItemParam, in id order), with retail's names:

| id | Network Test | retail v1.00 |
|---|---|---|
| 60970000 | 鋼石の欠片 (shard) | Titanite Shard |
| 60975000 | (no row) | Large Titanite Shard (new) |
| 60980000 | 鋼石の塊 (chunk) | Titanite Chunk |
| 60990000 | 鋼石の結晶 (crystal) | Titanite Slab |
| 61000000 | 純粋な黒鋼石 "pure black titanite": reinforces equipment | Twinkling Titanite: "reinforces what cannot normally be reinforced" |
| **61010000** | **古代の銀鋼石 "ancient silver titanite": reinforces equipment** | gone (icon only) |
| **61020000** | **龍の骨片 "dragon bone fragment": reinforces demon equipment** | gone (icon only) |
| 61030000 | 龍の化石 "dragon fossil": reinforces demon equipment | Petrified Dragon Bone: "reinforces special equipment" |
| 61040000 / 61050000 | 五色の結晶 / 虹色の原結晶 (infusion catalysts) | gone (icons only); see [cut infusion crystals](../network-test/cut-infusion-crystals.md) |
| 61100000-61120000 | poison / bleed / **Break** infusion stones | poison / bleed kept; **61120000 gone** (icon only); see [Break element](../network-test/break-element.md) |

- **Dragon bone fragment (wiki 2).** It is the smaller version of the 龍の化石, which became the Petrified Dragon
  Bone. Both carry the same text, "reinforces demon equipment", where retail's Petrified Dragon Bone says "special
  equipment". "Demon equipment" is the demo's name for what retail upgrades with Petrified Dragon Bone, the boss soul
  weapons. [The demo's boss souls](../network-test/nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000) include a 大蛙のデーモン "Great Frog Demon" and the 貪りデーモン "Covetous Demon", and
  デーモン appears nowhere else in its text. So the demo planned two tiers for the boss-weapon path, a fragment and a
  fossil. Retail made it one material, Petrified Dragon Bone ×1-5 from +1 to +5 (50 ReinforceCostParam rows).
- **Ancient silver titanite (wiki 1).** It sits between 純粋な黒鋼石 (which became Twinkling Titanite) and the dragon
  bones, with the same generic text as the black one. By that position, it is most likely the second tier of the
  black / silver pair, mirroring the fragment / fossil pair. Retail again kept one material, Twinkling Titanite ×1-5
  (194 rows). The pairing comes from the order and names. Neither stone's demo text says which equipment it
  upgraded; at the demo stage the black stone's description was still the generic one, not yet "what cannot normally
  be reinforced".
- **Five-coloured crystal (wiki 3).** One of the two infusion catalysts (see [cut infusion crystals](../network-test/cut-infusion-crystals.md)). The wiki's
  "smaller fragment of a Magic Stone" is close: the larger partner, 虹色の原結晶 (61050000), uses the same picture as
  retail's Magic Stone, shifted one pixel. The five-coloured crystal has no source in the demo. The rainbow one was
  a Bell Keeper online reward.
- **Dissolving stone (wiki 4).** The demo's infusion stone for its cut ninth damage type, Break (破壊, equipment
  destruction). See [Break element](../network-test/break-element.md).
- **Overall shape.** The demo had three titanite tiers (shard, chunk, crystal) and two tiers on each special path.
  Retail has four titanite tiers (it added the Large Shard as a new id, 60975000) and one material per special path.

## PC builds (2026-11-09)

No PC build adds anything about these four items: no rows, names, text, lots or extra icons. The pre-release PC
params' developer names do name the three upgrade paths: 通常 (titanite), 光石 "light stone" (Twinkling) and
ドラゴンウェポン "dragon weapon" (Petrified Dragon Bone), each with its own infusion-cost row. They also show that
Twinkling Titanite's working name was the Network Test's 純粋な黒鋼石. See [upgrade paths and PC developer names](../network-test/upgrade-paths-pc-dev-names.md).

## Not established

- **How the demo spent its materials.** It has no ReinforceCostParam (retail's table of souls and materials per
  level), and no param in its regulation refers to any of these ids except ItemParam. So which levels the fragment
  or the silver titanite were for can't be read from data.
- **What "ancient silver" upgraded.** It could have been a second tier for the unique weapons, or a material for a
  weapon class retail dropped. Only the name and position speak.
- **Sources.** No item lot or shop in the demo's regulation gives 61010000 or 61020000. The world's map and event
  files were not searched for these two ids; they were for the crystals.
