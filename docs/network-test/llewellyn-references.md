# Llewellyn (リンド) - every reference, cut and shipped (2026-11-10)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

English spelling in every build: **Llewellyn** (double l at both ends). Japanese: **リンド** ("Lind"); his title is 名工
("master smith"). He exists only in item names and descriptions: no dialogue line, NPC, blood message, map text or
character name mentions him in any build.

Searched: every unpacked FMG under <local path> (all languages) for `lew(e)l` and リンド, plus the game-data FMGs of v1.00,
1.01, 1.03, 1.10 and PS3 Scholar (English + Japanese, including talk, the v1.00 cut talk files and the NT draft talk
folder); geisteel / グラン鋼 for related items; pre-release PC param row names (31 Mar 2014 build). The vanilla PC menu
text was not read directly (encrypted archive), but PS3 v1.00 -> 1.10 -> PS3 Scholar -> PC Scholar text is identical
for all of these ids, so vanilla PC is almost certainly the same.

## Shipped (v1.00 to Scholar, text unchanged)

| Item | Id | What the text says about him | How it is obtained (data) |
|---|---|---|---|
| Royal Dirk | 1130000 | Geisteel was Mirrah's secret alloy; Vendrick had Chancellor Wellager hire the smith Llewellyn on a lifetime contract (JP: on high pay) to bring it to Drangleic | Wellager's talk script, Drangleic Castle (`talk_m20_21_00_00.esd`, lot 1721000 with Espada Ropera, gift flag 102260 (see [PC Scholar's change to this lot](../builds/placements-balance-pc.md#2-items))) |
| Espada Ropera | 1440000 | Wellager's rapier; a Llewellyn masterpiece he brought from Mirrah as a gift | same Wellager hand-over |
| Llewellyn Shield | 11080000 | Small geisteel shield; same "masterpiece brought from Mirrah as a gift" text | Stone Soldiers (117000 / 117010), Drangleic Castle, 2 % (lot 11700000). Stone Knight lot lists it at 0 %. 1.10 / Scholar: both Wall Watchmen in Frozen Eleum Loyce (883000 / 883200) carry it, no drop |
| Llewellyn Armor | 27210101 | Belonged to Wellager; made by the castle's resident master smith for a select few; plain, no-frills style | Stone Knights (150000, 4 placed), Drangleic Castle, 2 % (lot 15000000) |
| Llewellyn Gloves | 27210102 | as the armour | as the armour, 2 % |
| Llewellyn Shoes | 27210103 | as the armour | as the armour, 2 % |

- English slip: the gloves and shoes drop "Belonged to Chancellor Wellager" and keep the old NT sentence about light,
  sturdy gear, so the sentence about light and strong gear appears twice. The Japanese has the Wellager line on all three.
- In Japanese the shield is ドランシールド ("Dran Shield"), not named after him; the set is リンドの鎧 / 篭手 / 靴.
- The set is also worn (Drangleic Helm + set + Greatsword + Orma's Greatshield, NpcPlayerStatusParam 8361 / 8520) by
  the Greatsword Bell Keeper phantom in Belfry Sol (836100, 2 placed) and the Greatsword phantom in Undead Purgatory
  (852000); their lots list the set at 0 %.
- Related but not naming him: Grand Lance 2930000 and Twinblade 5340000 (geisteel gear for high-ranking knights).

## Cut and leftover

1. **Network Test (US and JP NT)**: Llewellyn Armor / Gloves / Shoes under NT ids 892002101-892002103, text "crafted by
   the master smith Llewellyn" plus the no-frills line; no Wellager, Mirrah, Vendrick or castle. In the NT map m10_23
   a treasure corpse (o00_0240_0001, object 10235540) holds lot 110235540 = Defender Helm + the set; the set is also in
   the NT full-kit row PlayerStatusItemParam 4120 (used by PlayerStatusParam 7 and 8). The NT Royal Dirk (900008401)
   says only that the king gave it to his chancellor (JP 軍師): the Llewellyn / Mirrah backstory was written after the NT.
2. **Planned head piece**: [NT 892002100, JP name 仮・王国軍師・頭](armour-rings-spells-nt-vs-retail.md#renamed-the-45-pieces-the-demo-names) ("placeholder: kingdom strategist, head"), text
   仮テキスト, no English. Retail keeps only ReinforceCostParam 30012400 "7210 【王国軍師（過去）】・頭"; no ItemParam /
   ArmorParam row (27210100 / 17210100) in any retail build (see [its placeholder icon on vanilla PC](../unused/hidden-pc-icons.md#placeholder-cards)).
3. **NT text left in retail**: the English names and descriptions of 892002101-103 survive in v1.00 to Scholar with no
   item rows. Japanese retail keeps the names, but its descriptions in the 8920019xx-8920022xx block are shifted five
   slots: the Lind armour and gloves text sits under 892002202 / 892002203 (named 仮・黒ファントム屋・腕 / 足,
   "placeholder black-phantom shop arms / legs") and the shoes text is gone.
4. **Developer row names** (PC 31 Mar 2014 to 1.02): set = リンドの鎧 / 篭手 / 靴（【王国軍師（過去）】・胸 / 腕 / 足）;
   Royal Dirk = 王の短剣（NPC：王国軍師：短剣）, Espada Ropera = （NPC：王国軍師：刺突剣）, Llewellyn Shield =
   ドランシールド（NPC：王国軍師：小盾）; WeaponParam 王国軍師の短剣 / 刺剣 / 小盾. The whole Llewellyn kit was the
   loadout of [a "past" Wellager (王国軍師（過去）)](community-cut-sheet-crossref.md#characters); the shipped Wellager is his spirit ([EnemyParam 721100 宰相の霊](cut-content-pc.md#3-characters-and-a-correction)). The
   past version survives only as EnemyDefaultLogicParam 721100 and EnemyEquipWeaponParam 72110000; no enemy row or
   generator places him.

Print a text: `fmg.py <ItemName.fmg | DetailedExplanation.fmg> <id> <id>` (DS2 Network Test\tools, `-X utf8` for Japanese).
