# Hidden map parts in retail v1.00

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-17, follow-on from `developer-signs.md`. Tool: `tools\devsigns\hidden.py` (DS2_GAME picks the build), renders
with `render3d.py` (flat-shaded top / front / oblique views). Renders: `findings\hidden-map-parts\`.

## How drawing works (why hidden things stay hidden)

Every map piece and object in an MSB carries a set of draw groups (128-bit mask at part +0x30). Every floor collision
carries a set of groups to display (+0x40). The game draws only the pieces whose groups overlap the display groups of
the floor you're standing on. That's room-by-room culling, not distance: most floors show two groups (a shared base
group and their own area). So a piece in a group that no floor lists is never drawn, however close you get. The PvP
arenas in m10_23 (groups 70/71/72) are the deliberate case: only each arena's own floor shows its group, and you only
stand on it once the game puts you in the arena. The network test's developer signs (group 96, shown by no floor) are
the "parked" case.

## The scan

For all 24 retail maps: every map piece (type 0) and object (type 1) whose draw groups are shown by no floor collision
(type 3) in the same map, plus parts with no draw group at all. Limits:
- A connect collision (type 5) or a neighbouring map can show a group; those cases are marked.
- Events can switch objects on, so hidden *objects* may still appear in play. Map pieces are static geometry.
- Arena-style hiding (shown only by a floor you can't reach) isn't caught: that needs reachability, not just masks.

The network test (m10_23 only) has just two: the sign model `m0800` (group 96) and object `o23_1000` with no group
(also in retail).

## Map pieces nobody can see

| Map | Piece | Group | What it is |
|---|---|---|---|
| m10_10 Forest of Fallen Giants | `m0640` | 127 | a few crude untextured-looking boxes and a strip spread over 178 m: blockout geometry |
| m10_14 Brightstone Cove Tseldora | `m6010` | 127 | low-poly octagonal building with a conical roof and a short corridor, 43 m across: a blockout or far-view stand-in |
| m10_17 Harvest Valley / Earthen Peak | `m0090`, `m0091` | 121 | rough, low-detail canyon terrain (250-290 m pieces): looks like an early blockout of the valley |
| m10_17 | `m0094` | 121 | a 336 m long bent backdrop wall |
| m10_18 No-man's Wharf | `m0078` | 127 | **a fully modelled three-masted ship** with rigging and hanging cages, about 86 m long. Possibly the "stranded ship" a Wharf sign mentions ("if you want to use this stranded ship, please get in touch"); unconfirmed |
| m10_18 | `m0904` | 100 | a huge open-bottomed box, 355 x 296 m: probably a lighting/darkness cover |
| m10_18 | `m1000` | 86 | a 9.3 km sky dome (not the one in use) |
| m10_30 | `m1008` | 32 | a plain 8 x 11 x 11 m box |
| m10_33 | `m0062` x2 | 96 | two 10.7 m stone pillars |

Not really hidden (a connect collision shows the group, so presumably drawn from the next map): m10_14 `m2020` x5
(small plant-like cluster, group 9), m10_29 `m8000` (a sky dome, group 6).

Group 127, the last group, holds three of these: it looks like a "park it here" group.

## Objects nobody can see (unless an event shows them)

| Map | Object | Group | What it is |
|---|---|---|---|
| m10_10 Forest of Fallen Giants | `o00_6090` | 110 | **a giant eagle**, 19 m wingspan, animated (textures `c2190`). Fits the "hawk taxi" in the Lost Bastille signs and the Pursuer's eagle that carries you to the Bastille |
| m10_10 | `o10_8510` | 110 | a 50 m mass of rubble/rock, at the same spot as the hidden blockout `m0640` |
| m10_16 Lost Bastille | `o16_8501` | 110 | a spiky beetle- or tick-like creature (animated) |
| m10_19 Iron Keep | `o19_8500` | 110 | a flat disc 101 m across (lava or fog plane?) |
| m10_04 Majula | `o00_2000`, `o00_2010`, `o04_2020`, 4 of each | 0 (no Majula floor shows group 0) | barrel, crate and urn (identified by their shadow meshes) stacked in a neat column, 2 m apart, at about (-61, 8-14, -96): looks like a test rack |
| m20_26 | `o00_2010` x4 | 0 | the same kind of stacked crates, 1 m apart |

Objects with no draw group at all: Majula 3 (`o00_2313`, `o00_2315`, `o00_2490`), Lost Bastille `o16_1220`,
m10_23 `o23_1000`, m10_32 five `o32_7000`, m20_10 twelve (mostly `o00_2000`), Drangleic `o00_2580`. Not investigated.

## The bigger pool: map pieces no layout uses

Separately from draw groups, 874 of the 2,863 map-piece models in retail's map archives aren't used by any MSB part
(749 not even listed, 125 listed but never placed; shadow `_s` models excluded). Most per map: Lost Bastille 150,
Huntsman's Copse 110, Dark Chasm 104, Majula 102, Undead Crypt 70, Harvest Valley 63. That's where the developer signs
were found. Surveyed since: `unused-map-pieces.md`. Lists: `tools\devsigns\unplaced.py`.
