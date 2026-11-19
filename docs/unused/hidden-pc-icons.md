# The 345 icons only vanilla PC has (2026-11-09)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Every vanilla PC build, from [the 31 March 2014 pre-release](../builds/early-pc-builds.md#31-march-pre-release) to today's 1.12, carries 345 item-icon files that no other
build has: not the US or JP Network Test, PS3 v1.00, PS3 Scholar or PC Scholar. Nothing uses them; no ItemParam row in
any build points at them. They are byte-identical across the PC builds. Our name lists come from the PS3 builds and
PC Scholar, so the PC indexes show these files only as path hashes. They were named by hashing every possible icon
path (`menu\tex\Icon\IC_00` / `IC_10` + 00000000-99999999) against the ~560 unnamed entries.
Re-run 2026-11-17 on 1.12 (18,867 entries, 569 unnamed; 200,000,000 names): 354 hashes hit, 345 are icon textures
and 9 are other files (chance matches, dropped). Four icon hashes match two names each; the kept one is the Network
Test boss soul (60738000, 60739000, 60748000, 60749000), the other a meaningless shield number (e.g. 11200327).

**One row per icon (2026-11-17):** Google Doc "DS2: the 345 icons only vanilla PC has" (link not published): thumbnail, file name, PC hash, placeholder or drawn, Network Test name with English reading, notes. Built by `..\..\DS2 Vanilla PC\tools\hidden_icons\all345_doc.py` (python-docx; uploaded as .docx, Drive converts).

How they were found: [unused ring icons](unused-ring-icons.md#vanilla-pc-holds-the-missing-icons-2026-11-09) ("Vanilla PC holds the missing icons"). Sheets: `hidden-pc-icons\`
(`A`-`F`, `C2`); every icon as PNG in `hidden-pc-icons\png\`. Scripts and tables:
`..\..\DS2 Vanilla PC\tools\hidden_icons\survey\` (`info.json` names, `nearest.json` closest in-use icon,
`classify.json` placeholder or art).

## Summary

| group | ids | files | what they are |
|---|---|---|---|
| armour, sets in no build | 21090100-21310103 | 63 + 59 female | **placeholder cards** "Head / Body / Arm / Leg NNN" for 16 sets that exist nowhere else |
| armour, Network Test pieces | 21450103-27770101 | 32 + 26 female | placeholder cards for pieces retail dropped (including **2481's body, arms and legs**), plus 2 real icons that copy retail ones |
| spells | 31112000-35151000 | 59 | **58 real, unique spell icons** for the Network Test's spell list, and one placeholder |
| rings | 40170000-40600000 | 10 | real icons, see [unused ring icons](unused-ring-icons.md); **three are copies** of retail ring icons (below) |
| key items | 50010000-50530000 | 45 | placeholder cards "Item 001-046" |
| gesture scrolls | 53000000-53190000 | 19 | placeholder cards "Item 047-066" |
| items and boss souls | 60080000-60755000 | 32 | placeholder cards "Item 074-164": Humanity ×3, 解呪石, 黒松明, 告罪符, and **all 26 Network Test boss souls** (Item 139-164) |

So 274 of the 345 are numbered placeholder cards (black, white lettering), and 71 are artwork. `IC_10` files are the
**female** versions of armour icons: the placeholders read "Head_F 015" beside the male "Head 017". The same holds for
retail's `IC_10` copies, such as the 2481 hood's `IC_1022481100`.

## Artwork

- **58 spell icons for [the Network Test's spell list](../network-test/armour-rings-spells-nt-vs-retail.md#spells).** They use the Network Test's spell numbers (31112000 etc.;
  retail renumbered its spells to 31010000 and up). The demo itself ships only 11 spell icons. Covered: sorceries
  (落下制御 "fall control", 偽魂 "false soul", 魔術の反転 "sorcery reversal", 衝撃波 "shockwave" and soul arrows /
  spears / beams), miracles (Force, 小さな大地の恵み, 解毒, 清らかな加護, 鎮魂の響き, 断呪の加護, 三女神の護り, 放電,
  聖女の祈り), pyromancies (Fire Serpent, Great Combustion, 火柱, 炎陣, 炎の飛礫, 混沌の嵐, 地竜の怒り, 混沌の大火球),
  hexes (Dark Barrier, Bitter Mist, 死者の活性, 逃れられぬ怨念, 毒の沼, 呪詛, 封術, 闇の刃, 魂の吸収, 死霊の召喚,
  冷たい炎の嵐) and the **35xxxxxx school** (Dark Burden, 再動の念, 硬化の念, 活性の念; 衝撃の念 is the placeholder
  "Spell 069"). Each school has a "reversal" (反転) spell. Matched against every in-use PC spell icon, the closest is a
  different picture every time (`C2-spells-vs-closest-retail.png`), so none of the 58 is retail art.
- **Rings** ([unused ring icons](unused-ring-icons.md)): new art for the four starstones (the red one is the Old Sun Ring's picture),
  the Rainbow Tearstone and the Great Eagle Crest. **Three are pixel-identical to retail ring icons:** Dragon Claw
  40240000 = Ring of Giants (40290000), the demo's Ring of Thorns 40250000 = Old Leo Ring (40295000), Rusted Iron
  40380000 = Stone Ring (40230000, the demo's 仮・大蜘蛛の指輪). 40600000 = the King's Ring.
- **Two armour pieces copy retail icons:** 26220100 (set 6220 head) = Penal Mask (26260100), 27770101 (set 7770
  body) = Hexer's Robes (27700101), plus the female copy of the latter.

Whether those copies are art that moved from a cut ring or piece to a kept one (as the Flame Starstone's did to the
Old Sun Ring), or a kept icon reused as a stand-in, can't be told from the files.

## Placeholder cards

The numbers on the cards run as one list per kind, in id order:

- **Items:** key items 001-046 (50010000 ... 50530000), gesture scrolls 047-066, then 074 (解呪石 60080000), 079-081
  (Humanity, Twin and Triple Humanity), 110 (黒松明 "black torch"), 118 (告罪符) and 139-164 for [the 26 boss souls](../network-test/nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000)
  (60730000-60755000: 紅炎トカゲ, 大蛙のデーモン, 蜘蛛の女王, 宿り大蟹, 幼生ナーガ, 地竜の化身, ヘル・スコーピオン,
  デス・ストーカー, 鏡の騎士, カゲロウ竜, 岩の巨人カノッソフ, 黒炎トカゲ, アガドゥラン, 古の神木, 双頭竜, 竜騎兵, 成体ナーガ,
  貪りデーモン, 魔手, 呪手, 魔女リディーレア, 幻影の騎士, 幻影の術士, ヴァンクラッド王, 王妃デュナシャンドラ, 処刑人). The
  names come from the Network Test's text; none of these ids has an item row in any build. So the boss souls,
  Humanity and the key items still have **no art**, only list numbers.
- **The Network Test's key items** (Item 001-046): 紅蓮の火, 龍の宝珠, 刻の雫, ゆりかごの鍵, 王妃のぬくもり, 王家の鍵,
  王城門の鍵, 龍卵の化石, 石砕の楔, 古の龍のソウル, a run of keys (探究の館裏口, 水車の塔裏通路, 牢屋, 賊, 海辺の護り, 下水路,
  熔鉄工廠第二棟, 卵, 王城の監獄, 客間, 兵舎, 地下牢, 水車の塔４階, 館の地下, 主塔, 倉庫), 石棺, ずた袋 ×5,
  エスティンティアの羽, 霊験の薬草, 古びた魔導書, 上質な種火, 煌めく種火, 城下街の絵図, 刻の霊石, 紋章の楯, 神木の小枝, 感状,
  敗者の魂 and 石ころ ("pebble").
- **Gesture scrolls** (Item 047-066): the Network Test taught gestures through scroll items ("Point", "Bow",
  "Wave", "Hurrah!", "Okay", "No", "Allow me!" Scroll, plus unnamed ones).
- **Armour:** the card number is a set number (the female card is one or two lower). [Network Test pieces](../network-test/armour-rings-spells-nt-vs-retail.md#armour): 1450
  legs 073, 2180 (Hollow Mage) arms / legs 092, 2181 094, 2182 096, 2190 head 098, 2221 100, 2231 104, 2351 110, 2461
  116, **2481 body / arms / legs 120** (the unused hood set, [unused armour icons](unused-armour-icons.md): still no art for those three),
  5120 legs 141, [7210 (Llewellyn) head](../network-test/llewellyn-references.md#cut-and-leftover) 145.
- **16 sets that exist nowhere else:** 1090 (017), 1110 (021), 1120 (023), 1130 (025), 1150 (029), 1170 (032),
  1180 (034), 1190 (036), 1200 (037), 1220 (041), 1240 (045), 1250 (047), 1270 (049), 1280 (051, no head), 1300 (053),
  1310 (055). They fill the gaps in retail's 1xxx series, where the developers' row names are mostly archetypes
  (1010 軽量戦士 light warrior, 1020 盗賊 thief, 1030 放浪者 wanderer, 1040 狩人 hunter, 1050 兵士 soldier, 1060
  魔術兵士, 1070 呪術師, 1080 山賊 bandit, 1100 旅人の服, 1140 アイテム師, 1160 重戦士 ...). No item, armour row, text or
  model file for any of the 16 exists in the Network Test, PS3 or PC. So 16 planned sets in that series never got
  further than a placeholder icon.

## Not established

- Why vanilla PC alone kept these files (most likely a menu-texture folder packed whole for PC and pruned for PS3,
  then rebuilt for PC Scholar).
- What the 16 planned 1xxx sets were meant to be: nothing but their numbers survives.
- Whether any of the 58 spell icons was redrawn for a retail spell under its new number. None matches as the same
  picture, but a reworked one was not looked for.

## Related pages

- [Network Test-only equipment models](nt-only-equipment-models.md): the same path-hash search applied to models: which Network Test-only weapons and armour pieces have a model file in any build.
- [Wiki Unused Content cross-check](../network-test/wiki-unused-content-crossref.md): the community wiki's Unused Content page checked against these files: its star rings, rainbow ring and 58 of its spell scrolls are icons from this set.
