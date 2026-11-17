# The wiki's "Unused Content" page against the game files (2026-11-09)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author saved the community wiki's Unused Content page (darksouls2.wiki.gg, last edited 3 September 2026) as a PDF
(`Downloads\Unused Content - Dark Souls II Wiki.pdf`). Question: does any icon on it still lack a known source?

Method: the PDF's 148 distinct images were extracted with their captions and matched (content box, dark background,
32 × 48, mean difference) against 6,398 game icons: vanilla PC 1.12 (named and hidden, `hidden-pc-icons.md`), PS3
Scholar, PC Scholar and the Network Test. Every non-spell match was checked by eye. Scripts:
`..\..\DS2 Vanilla PC\tools\hidden_icons\wiki_page\`.

## Answer

**Every item icon on the page is a game file.** The only images without an item-icon source are the ones already
known:

- the gold 26 × 26 menu icons, unattached even in the Network Test (`unused-menu-icons.md`). **Update
  2026-11-16:** all 15 are cells of the Network Test's older draft sheet `menu\tex\02\Icon_tex.tpf`, and the 10
  "Cut Icons" are cells of the shared sheet `Icon_tex` (`wiki-cut-icons.md`);
- the "Hornet" ring, which is the Network Test's menu placeholder `yubiwa_test`, not an item icon
  (`unused-ring-icons.md`).

(The two blank grey boxes in the PDF are the wiki's page-header buttons, which didn't render; four more images are
page decoration.)

## Where the wiki's unknown or uncertain icons come from

| wiki entry | icon file | what it is |
|---|---|---|
| "Unknown" weapon (p. 3) | `IC_0007000000` | the Network Test's weapon row 7000000 (bare-fists type, model `WP_7000`, no name in any build; `weapons-nt-vs-retail.md`) |
| Blacksmithing Tongs | `IC_0002510000` | unused model `WP_2510` (`weapons-nt-vs-retail.md`) |
| wooden shield | `IC_0011095000` | the cut wooden shield `SD_1095` in Melinda the Butcher's loadout, which retail's NPCGenerateParam still equips (`weapons-nt-vs-retail.md`) |
| "Unknown" hood | `IC_0022481100` | the unused Peasant sister set 2481 (`unused-armour-icons.md`) |
| shiny Alonne Knight set | `IC_00231901xx` | the other whole unused set, model 3190 |
| Dark Souls 1 red set | `IC_00213801xx` | set 1380, a Network Test item row (`21380100`) |
| Gren's hood / robes | `IC_0892002200` / `201` | the Network Test's test items 892002200-201 (`armour-rings-spells-nt-vs-retail.md`) |
| barrel | `IC_0021450101` | set 1450's body |
| starstone rings, rainbow ring | `IC_0040170000`-`40200000`, `IC_0040270000` | vanilla PC only (`unused-ring-icons.md`) |
| "Unknown Serum" 60300000 | `IC_0060300000` | the Network Test's **龍の紅液 "dragon's crimson fluid", the Break resin**: "temporarily gives the right-hand weapon a break effect". The wiki's stronger-bleed guess is off (`break-element.md`) |
| "Rhoy's Stone of Knowledge" | `IC_0060460000` | the Network Test's text-only 導きの護符 "guiding talisman", not Rhoy's stone (`cut-item-descriptions.md`) |
| Rhoy's Stone (cyan shard) | `IC_0060330000` | correct: retail's Rhoy's Stone, the Network Test's クィンドニーの魔石 "Quindony's magic stone" |
| lantern, "strengthen nearby foes" | `IC_0060527000` | the Bonfire Ascetic's own (used) icon |
| three "Unknown" materials and the "Magic Stone fragment" | `IC_0061010000` / `61020000` / `61120000` / `61040000` | ancient silver titanite, dragon bone fragment, dissolving stone (Break), five-coloured crystal (`unused-material-icons.md`) |
| all the spell scrolls | `IC_0031112000`-`35141000` etc. | the Network Test's spell list; 58 of them exist only in vanilla PC (`hidden-pc-icons.md`) |

The Caestus, Greataxe and Rapier "reskins" are three of seven exact weapon copies with new models that patch 1.08 added
and nothing hands out (`patch-1.08-weapon-copies.md`). The rest ( the Dark Sword, the Blue Flowery Club, greatarrows,
Illusory Rings, Ring of War, smithboxes, Yellow Sea Troches and so on) match the files the wiki already names.

Except for the vanilla-PC-only files (starstones, Rainbow Tearstone, 58 spells), all of these icons are in PS3
v1.00, PS3 Scholar and PC Scholar too.
