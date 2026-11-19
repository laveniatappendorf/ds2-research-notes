# Network Test equipment that retail cut: which pieces still have a model (2026-11-09)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author asked whether we had rendered every Network Test item that retail cut and that has a model at all. Method:
every Network Test weapon, shield and armour item whose model no retail WeaponParam / ArmorParam row uses (84
models), then a search for the model file (male and female; `model`, `model_hq` and `model_lq` folders) in the
Network Test, PS3 v1.00, PS3 Scholar and PC Scholar by name, and in vanilla PC (31 March 2014, 1.02, 1.12) by path
hash, which also finds files no name list has (as with the hidden icons, [hidden PC icons](hidden-pc-icons.md)). Script and table:
`..\..\DS2 Vanilla PC\tools\hidden_icons\survey\ntonly_models2.py`, `ntonly_models.json`.

## Answer: 22 files; every one with a real mesh is now rendered

Renders in `nt-only-equipment-models\` (tool `tools\unused_models\partview.py`, which reads PS3 files and vanilla
PC files by path hash, including PC DDS textures).

| model | Network Test item | where the file is | what it is |
|---|---|---|---|
| HD / BD / AM / LG_2481 | [the Peasant's sister set (hood)](unused-armour-icons.md) | PS3 + PC, M and F | rendered (`unused-armour-2481\`) |
| SD_1095 | small wooden shield (Melinda's) | PS3 + PC | rendered: a round plank shield like a barrel lid |
| WP_3510 | **Shadow Claws** | **vanilla PC only** (hidden) | rendered: five loose finger blades and a curved thumb spike, 0.18 m, no glove. Its one material is `WP4_c1230_M_twoside`, named after enemy c1230, the 異形の影 ("shadow of a deformed one") whose claws the retail text says these are |
| WP_3511 | the claws' twin row | **vanilla PC only** (hidden) | rendered: a full armoured claw gauntlet, 0.4 m, five blades and a hooked thumb, engraved plates |
| WP_2450 | the flower club 2450000 | **vanilla PC only** (hidden, all three builds) | rendered: a 1.06 m teal mace shaped like a closed flower bud, lattice-work and a red gem, spiked tip |
| WP_2510 | the "tongs" hammer 2510000 | PS3 + PC | rendered: smith's tongs, 0.72 m |
| WP_7000 | row 7000000 (bare-fist type) | PS3 + PC | rendered: **a peacock-feather fan** on a short handle, 0.81 m, two feathers with cloth physics; [a held prop](../network-test/child-herald-c7015.md#the-model), not a weapon |
| HD / BD / AM_1380 | set 1380 ([the wiki's red "Dark Souls 1" icons](../network-test/wiki-unused-content-crossref.md)) | PS3 + PC, M and F | **empty files**: every copy (PS3, PC `model` and `model_hq`) has a FLVER with zero meshes |
| LG_1380 (not in the 84: retail keeps an armour row for it, but no item) | set 1380's legs | PS3 + PC, M and F | rendered: a long dark grey tattered skirt with a beaded belt. It doesn't look like the red leg icon `IC_0021380103` |
| AM / LG_1430, HD / LG_1440, HD / AM / LG_1450, HD / AM_1460 | the other pieces of the sets whose Dorgo's Cap (1430), Engraved Gauntlets (1440), Barrel (1450) and Flying Feline Boots (1460) retail kept | **vanilla PC only** (hidden, male) | **no mesh**: texture-only binders. Every head stub holds `HD_1430` (Dorgo's Cap) textures, every arm stub `AM_1440` (Engraved Gauntlets), every leg stub `LG_1460` (Flying Feline Boots), some byte-identical to the kept piece's, some a different version. No body file exists except the Barrel's |

So the cut pieces of the 1430-1460 sets and of set 1380 were never modelled (or their models were emptied). What
survives with a real mesh is the 2481 hood set, the wooden shield, the two Shadow Claws models, the flower club,
the tongs and the peacock fan. The Shadow Claws, which [weapons: Network Test vs retail](../network-test/weapons-nt-vs-retail.md) said had no model, have two.

The four sets 1430-1460 look like one group: retail kept one piece from each (head 1430, arms 1440, body 1450, legs
1460), and the leftover stubs for the other part slots carry exactly those four pieces' textures.

The other 62 Network Test-only models have no file in any build: WP_3401; BD_1430, BD_1440, BD_1460; whole sets
2061, 2150, 2181, 2221, 2231, 2461, 3041, 7901; the Hollow Mage's arms and legs (2180) and set 2182's; HD_2190;
the second Prisoner set's arms and legs (2271); 2351, 2370, 3011 and 3040 pieces; HD_3080, HD_5060, LG_5120,
Llewellyn's head (HD_7210), AM_7510 and [the Despatcher's arms and legs (7830)](../network-test/armour-rings-spells-nt-vs-retail.md#armour).

Earlier renders cover characters and objects ([unused models](unused-models.md)), not equipment; before 2026-11-09 no cut equipment
had been rendered.
