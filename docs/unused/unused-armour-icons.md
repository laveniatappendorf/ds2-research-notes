# The wiki's "Unknown Headpiece" and the Network Test (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author pasted the community wiki's "Unknown Headpiece" (a brown hood, description "Unknown"). Question: does the
Network Test point to what it is?

Image: `unused-armour-icons.png` (wiki crop left of the bar, game files right; crop in `unused-armour-icons\`).
Tool: `tools\wiki_icons.py compare armour`.

## Answer

It is **the head of a whole unused armour set on model 2481**, the sister model of the **Peasant** set (model 2480:
Peasant Hat / Attire / Long Gloves / Trousers). Its icon shows the Peasant Hat's hessian hood without the straw hat
on top. Retail names it nowhere. The Network Test planned it as an item but never gave it armour stats.

- **Icon:** `IC_0022481100` (match 0.89), and a copy `IC_1022481100` (0.93). Retail keeps such 10-prefixed copies for
  many armour icons; the Peasant Hat has one too. Both are in v1.00, 1.01, 1.03, 1.10, PS3 Scholar and PC. Every
  other icon scored 0.44 or lower. Only the head has an icon: there is no `IC_0022481101`-`103`. **Update
  2026-11-09:** vanilla PC alone does have `IC_0022481101`-`103` and their female `IC_10` versions, but they are
  placeholder cards reading "Body / Arm / Leg 120", not art (`hidden-pc-icons.md`). The `IC_10` prefix marks the
  female icon, so `IC_1022481100` is the hood's female icon.
- **Network Test:** ItemParam has **items 22481100-22481103 for all four pieces**. They point at armour ids
  12481100-12481103, with the head / body / arms / legs kinds (2-5), but **ArmorParam has no such rows**. They are
  among the demo's items that point at armour it doesn't have (`armour-rings-spells-nt-vs-retail.md`, "Missing
  pieces"). They have no name and no text, because the demo names armour only through its test items. The demo
  ships neither model 2480 nor 2481; it carries models for only 59 of its 412 armour pieces.
- **Retail:** no item or armour rows for 2481, no text, no lot. But it ships the **complete model set**:
  `HD / BD / AM / LG_2481`, male and female, plus their `_L` versions. These are separate meshes with their own
  textures, larger than the Peasant files (head 551 KB against 417 KB, body 3.3 MB against 2.6 MB). The 2481 body has
  no cloth simulation, where the Peasant Attire has one. `unused-models.md` already lists 2481 as one of two whole
  unused armour sets. 2481 has since been rendered (see "Renders" below); 3190 too, on 2026-11-16 (end of this file).
- **Nothing equips it:** no param in the Network Test, v1.00 or PC holds 12481100-103 or 22481100-103 apart from the
  demo's four item rows. The Peasant set itself (2480) has ArmorReinforceParam and EquipInterfereParam rows in all
  three builds, and retail's NpcPlayerStatusParam dresses an NPC in the Peasant Hat and Attire.

So it was planned as a wearable, obtainable set from at least the Network Test onwards (the items exist). It was
modelled in full but never given stats, and it was dropped from the item list before v1.00. The renders below show
what the whole set looks like.

## Renders (2026-11-09)

Images in `unused-armour-2481\`: `2480-vs-2481-male.png`, `2480-vs-2481-female.png`, `2480-vs-2481-heads.png`
(Peasant 2480 top row, 2481 bottom row; front, right side, back, three-quarter) and `2481-male-armour-only.png`
(without the bare-skin meshes the arm, leg and body pieces carry). Tool: `tools\unused_models\armourview.py`.

They are drawn straight from the retail v1.00 parts files, so no game or mod is needed. The pieces are in their
modelled pose (a T-pose) with the colour texture only and simple lighting. The bare-skin texture isn't in the armour
files, so skin is a flat colour.

What they show, piece by piece against the Peasant set:

- **Head:** a pointed hessian hood with an open face and a drawstring at the throat, and no straw hat. It matches
  the icon. Shape and textures are entirely its own: half or more of its vertices sit away from any Peasant Hat
  vertex.
- **Body:** the same kind of patched tunic and belt with a ragged hem. The Peasant Attire's moving cloth pieces (cape,
  bag, skirt, strap) are part of the solid model here, and there is no separate collar piece. That is why it has no
  cloth simulation. About 35-50 % of its vertices are away from any Peasant vertex (male 35 %, female 52 %).
- **Arms and legs:** almost exactly the Peasant shapes (0-4 % of vertices moved) but repainted. The colour textures
  differ, while the surface-detail (normal) maps are nearly the same. As with the Peasant set, one arm has a dark
  sleeve and the other bare skin with wraps.
- **File sizes:** every 2481 shine (specular) texture is twice the Peasant resolution (head 512 against 256, body
  1024 against 512). That accounts for the bigger files noted above.

So 2481 reads as a hooded variant of the Peasant outfit: a new hood and a reworked body, with the Peasant gloves and
trousers repainted to match.

Technical notes for the tool: retail parts FLVERs are version 0x20010, whose UVs are short pairs divided by 2048
(not 1024); the Peasant body's cloth meshes keep positions and UVs in two separate vertex buffers.

## Where both sets come from: the peasant enemy c1480 (2026-11-09)

The author's screenshot of the peasant hollows (both wearing a hood, no hat) raised the question of whether enemies wear
the player's armour. They don't, and here the armour was built from the enemy.

Image: `unused-armour-2481\c1480-enemy-variants.png` (all meshes, then the hood and hat looks without weapons). Tool:
`tools\unused_models\c1480view.py`.

- **The enemy is its own model.** `model\chr\c1480.bnd` holds one FLVER (18 meshes) and `c1480.texbnd` its
  textures. The single model carries the body, the arms and face (`AM_FC`), **two heads** and the weapons:
  - `HD1_c1480`: a close hood with the hollow's face showing (the look in the screenshot).
  - `HD2_c1480`: the same hood with the straw hat on top.
  - Weapons `WP1_c1480` and `WP2_c1480` (one a scythe), plus an ordinary weapon model, `WP_6000_M`.
  - There are six EnemyParam rows (148000-148050), and the mesh materials carry variant tags (`col1`-`col3`,
    `cull3`-`cull5`). So the head and weapon are probably chosen per enemy; not checked.
- **The player sets keep the enemy's names.** Set 2481's head material is `HD1_c1480` (the hood), the Peasant Hat's
  (2480) is `HD2_c1480` (the hat), and both bodies use `BD_c1480_1` / `_2`, the arms `AM_c1480` and the legs
  `LG_c1480`. So both sets were made from this enemy, one per head.
- **Peasant Hat = the enemy's hat texture.** Its colour texture is the enemy's `HD2_c1480_d` at twice the size
  (256 -> 512); scaled back down, the two differ by 3.7 out of 255 on average.
- **2481's hood was redone.** Its colour texture matches neither enemy head (28 against HD1, 33 against HD2). The
  shape is also its own: a looser, pointed hood with a drawstring, where the enemy's is a close hood. That fits a hood
  that has to cover the player's own head, where the enemy shows its hollow face.
- **Both sets were refitted to the player body.** Their head and body vertices sit a median 2-5 cm from the
  enemy's nearest ones, with only 2-10 % within 1 cm.
- **Ordinary enemies never wear armour pieces.** Each enemy type is a self-contained model like this one. Armour
  pieces (`model\parts\`) are worn only by characters built on the player body `c0001`. The 97 generators that place
  `c0001` are all NPCs and phantoms, summons and NPC invaders included (`unused-models.md`, characters section).

So the developers turned both enemy looks into player sets. The hat look shipped as the Peasant set, and the
hood-only look (2481) was dropped.

## Not established

- **Its name.** Nothing in any build names it.
- **How it looks in the game.** The renders are static. Seeing it move, lit as in the game, would need armour rows
  added and a live test.
- **How c1480 picks its look.** Which param or flag chooses the hood or the hat, and which weapon, for each
  placed peasant hasn't been traced.
- **The other whole unused set, model 3190.** It has retail icons for all four pieces (`IC_0023190100`-`103`; the
  wiki calls it the "Shiny Alonne Knight set") and retail models, but no rows or items in either the Network Test or
  v1.00. So unlike 2481, the Network Test has no trace of it. Rendered 2026-11-16, see below.

## Set 3190, the shiny Alonne Knight set (2026-11-16)

Images in `unused-armour-3190\`: `3061-vs-3190-male.png` and `3061-vs-3190-female.png` (Alonne Knight set 3061 top
row, 3190 bottom row; front, right side, back, three-quarter). Same tool and limits as the 2481 renders: v1.00 parts
files, bind pose, colour texture only.

- **Shape:** the Alonne Knight set's silhouette, piece for piece. Slightly fewer triangles (male 7577 against 7959).
- **Textures:** 3190's colour and normal maps are twice the resolution of 3061's; its specular (shine) maps are four
  times the resolution and brighter. That is the "shiny" in the wiki's name. The renders draw colour only, so the
  extra shine doesn't show in them.
- **Cloth:** the 3190 body has no cloth simulation, where the Alonne Knight body has one.

So 3190 reads as a higher-detail, shinier remake of the Alonne Knight set, dropped before v1.00 with no rows at all.
Its name and purpose (an upgraded version, a boss or NPC look) aren't in any data. Added to the wiki-team Google Doc
("Renders of the unused models", R8) the same day.
