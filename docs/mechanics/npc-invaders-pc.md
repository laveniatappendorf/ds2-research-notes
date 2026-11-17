# NPCs who also invade (PC Scholar 1.0.3.0)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-11-04. Tool `tools\npc_invaders_pc.py` (table `npc_invaders_pc.tsv`, one row per spirit spawn); new shared reader
`..\DS2 Network Test\tools\edd.py` (developer names for event commands and events). PC only; data, not live-tested.

## 1. Answer

Five named characters invade the player and also appear in another role (a friendly NPC in the world or a summon
sign):

| Character | Other role | Invades | Notes |
|---|---|---|---|
| **Royal Sorcerer Navlaan** | Friendly NPC in Aldia's Keep (EnemyParam 771000) | 6 spawns: Forest of Fallen Giants, Brightstone Cove Tseldora, Aldia's Keep, The Gutter, Dragon Aerie, Drangleic Castle (771004-771010) | Same model (c7710) and weapon. Only after quest flag 102810, which the Aldia's Keep event "【実行】封じられた人" ("sealed one") sets when the object sealing him opens. Stops once flag 104340 is set (not decoded). The Forest spawn also needs flag 100972 |
| **Melinda the Butcher** | White summon sign, Dragon Aerie / Dragon Shrine map (814000) | The Gutter (814009) | Same NpcPlayerStatusParam row 8140 (Prisoner's set, same weapons). Sign HP 8,060, invader 1,860 (+4,004 souls). The spawn sits at height +15 with the Gutter's own enemies; Black Gulch spawns sit at -60 to -80 |
| **Jester Thomas** | White summon sign, Earthen Peak (849100) | Crown of the Sunken King DLC map m50_35 (861000; community sheet: Dragon's Sanctum) | Same Jester's set, same 10 pyromancies, same death drop lot 84910000; only the right-hand NPC weapon id differs |
| **Transcendent Edde** | White summon sign, DLC map m50_35 (868000; community: Dragon's Rest) | Dragon Shrine (868100) | Edde's name text (8680) and Black Dragon armour; two swords and no Black Dragon Shield. Not a plain dark spirit: generator kind 0 and phantom colour row 12 (ChrNetworkPhantomParam rows 11-13 use it), so probably shown as a dragon spirit. Developer event name "マルチ用NPC：エディラ：黒ファントム出現：オンライン". The community sheet calls it "Black Dragon Knight Phantom" |
| **Bell Keeper** | The covenant NPCs in Belfry Luna (753000) and Belfry Sol (753002) | Belfry Luna (753009) and Belfry Sol (753008), **offline only** | Same name, model c7530 and weapons as the covenant NPCs. The data doesn't say whether they're the same old men or other bell keepers. Developer names "NPC：鐘守：黒ファントム出現：オフライン" (the Sol one adds "ヴェイン") |

Not on the list:
- **Lone Hunter Schmidt** never invades on PC. He has a white sign in Black Gulch and a shade sign in Amana. The "NPC
  invader" label in `shrine-of-amana-pc.md` came from the community sheet and was wrong (corrected there).
- **Sign / shade only**: Bashful Ray, Sellsword Luet, Bradley of the Old Guard, Manhunter O'Harrah, Devotee Scarlett,
  Masterless Glencour, Pilgrim Bellclaire, Felicia the Brave, Ashen Knight Boyd, Drifter Swordsman Aidel and the other
  DLC summons. Several of these have generator HostileEventIDs, so they can turn hostile, but they never spawn as
  invaders.
- **Quest NPCs** (Lucatiel, Benhart, Pate, Creighton and the rest): no invader spawn at all.
- **Aslatiel of Mirrah** invades in Aldia's Keep. He is Lucatiel's brother, not Lucatiel.

Invader-only named characters: Armorer Dennis (Forest of Fallen Giants, Iron Keep), Fencer Sharron (Iron Keep, Dragon
Aerie), Oliver the Collector (Iron Keep), Bowman Guthry (Tseldora; Doors of Pharros offline only), Rhoy the Explorer
(Grave of Saints, offline only), Merciless Roenna (Huntsman's Copse map), Vorgel the Sinner (Belfry Luna map), Aslatiel
(Aldia's Keep), Woodland Child Victor and Gully (Black Gulch), the Gutter Denizen, Dragonfang Villard (Dragon Shrine),
Peculiar Kindalur (Amana), the Nameless Usurper (Drangleic Castle, Undead Crypt; tied to Licia, section 5), the Forlorn (15 areas), and in the DLC
Rockshield Baldyr, Quicksword Rachel, Maldron the Assassin, Castaway Witch Donna, Hexer Nicholai and Holy Knight Aurheim.

## 2. How an NPC spirit is made

- **Command**: map event scripts (`ezstate\event_<map>.esd`) call **1:131741**, which the `.edd` names
  **【ジェネレータ】NPC霊体生成** ("create NPC spirit from generator"). Its argument is a generator id, usually passed
  through a compiler sub-machine. 1:131742 removes the spirit. The same command makes summon signs and invaders.
- **Kind**: set by the generator. `GeneratorParam_<map>` field **Unk84** (SmithBox name, undocumented): 1 = white summon
  sign, 3 = small white (shade) sign, 2 = dark spirit, 0 = Edde's Dragon Shrine spirit only. EnemyParam
  `chr_phantom_param_id` is not the switch: Navlaan's and Melinda's invader rows have colour 0.
- **Cross-check**: the developers' event names in the `.edd` say 白ファントムサイン表示 / 白霊 / 小白霊 for signs and
  黒ファントム出現 / 黒霊 / 闇霊 / 放浪者 for invaders. They agree with Unk84 on all 157 spawn calls (119 generators)
  bar one wording slip: Navlaan's Tseldora event says "黒ファントムサイン表示" but runs the invasion template.
- **Triggers**: invasion events wait for the player to enter a trigger region, then a short timer, then spawn. Sign
  events place the sign when the map loads. "Offline only" means the event checks 1:140671
  【グループ条件：ネット】オフライン判定; online, real players fill those covenant roles. Every spawn skips guests
  (f130602 ゲストユーザー判定).
- **Names**: EnemyParam `name_text_id` -> `npcMenu.fmg`.

## 3. The .edd files

Each `.esd` has a `.edd` beside it: an ezstate debug file with a string table of names (layout in `edd.py`). It names:
- every **command** (325 in m10_25) and **function** (131), e.g. 130101 【フラグ】フラグを設定 "set flag", 140301
  イベントフラグ比較 "compare event flag", 140132 プレイヤーポイント内判定 "player inside point";
- every **event group**, including the compiler's sub-machines (【実行】 / 【プリセット】 variants).

Until now the project read only loose strings from these files. With `edd.py`, `esd.py` dumps can be labelled with
these names.

## 4. Side notes

- The Forlorn also sit directly in Aldia's Keep (4 generators, 694009 / 695004) and No-man's Wharf (694001) without a
  spirit event. Answered 2026-11-05 in `forlorn-pc.md`: Aldia's Keep ones are brazier-linked ordinary enemies; the
  Wharf one is never spawned.
- Two NG+ black phantoms in Drangleic Castle (853200, 853500) reuse Bowman Guthry's name text. This is harmless, since
  plain enemies show no name.
- Open: what the game shows for Edde's kind-0 spirit (dragon spirit banner?); what flags 104340 / 100972 mean; a live
  check of any of these.

## 5. The Nameless Usurper and Licia of Lindeldt (added 2026-11-04)

The author asked whether the Usurper is the early miracle vendor (Licia; dev name 奇跡の人 "the miracle person", NPC 7690)
or whether the link is lore only. **There are mechanical links, but the game keeps them as two characters.**

Links:
- **Licia's death ends the Usurper's invasions.** Both Usurper events (Drangleic Castle 118140, Undead Crypt 118220)
  end, without spawning, once flag **104320** is set. 104320 is Licia's death flag: NpcInfoParam 7690
  `npc_death_flag`, and the DeathEventID of all her generators (Majula 150 / 152, Heide's Tower 150). No script sets
  it, so only her death does (by any means, including the fight below). It doesn't work the other way round: the
  Usurper's generators have no death flag, and killing the Usurper changes nothing for Licia. Each event also has an
  older first state without the check, which is never reached (state 0 jumps straight to state 2).
- **The Usurper wears Licia's own armour.** NpcPlayerStatusParam 8490 = the full Saint's set (17690100-103). Its ids
  follow the NPC-number rule (Lucatiel's Mask 27520100, Creighton's Steel Mask 27420100, ...), and the text says
  "Belonged to Licia of Lindelt". The Dress adds "Even clerics aren't always who they seem". The same set sits in
  Licia's own death lot 76900000 (at 0 %) and in the reward for killing her in her world (below).
- **Miracles.** The Usurper casts Heavenly Thunder, Lightning Spear, Great Lightning Spear, Sunlight Spear, Blinding
  Bolt, Soothing Sunlight and Sacred Oath (NPC copies 72xx0000 = player miracle id + 40000000). Lightning Spear is in
  Licia's shop; Soothing Sunlight is in her world-invasion reward.

Not linked:
- A separate NPC number (8490), name ("Nameless Usurper") and drop (Black Witch set, lot 84900000). Licia is her own
  model (c7690); the Usurper is a player-type human with face preset 8120, the same as Pilgrim Bellclaire's. Weapons:
  NPC copies on the Bandit's Knife, Shortsword and Pyromancy Flame models.
- The developer event names for the Usurper are copied template labels (肉断ち（女）, 魔導士（男）) and don't name
  Licia.

**Invading Licia's world** (Majula, events 3010 / 3020; talk group 7691 奇跡の人（戦闘イベント） "battle event"). This
is a separate PC mechanic, and the one place you fight Licia herself:
- **Conditions**: you hold the **Crushed Eye Orb** (item 51000000, "Invade world of a traitor"); Licia is alive
  (104320 off), living in Majula (102765 on: the AppearanceEventID of her Majula generator) and not angered (103821
  off: her HostileEventID in Heide's Tower); and flag 102761 is on. Licia's "boss kill counter"
  (111415) sets 102761 once **3 of the 4 Great Soul bosses** are dead (map-clear flags of Freja 100951, Lost Sinner
  100963, Old Iron King 100952, The Rotten 100966).
- **Trigger**: standing on hit group 2 shows "The crushed eye stirs..." (mapevent 2100) and sets 102770. The orb is
  only usable while 102770 is on (ItemUsageParam 2702).
- **Result**: you face Licia (Majula generator 152, EnemyParam 769000, DeathEventID 104320). Killing her gives reward
  lot 60006000 (Rotunda Lockstone, Soothing Sunlight, Idol's Chime, Saint's set), sets 104322, shows "Returning to
  your world...", warps you back, and sets her death flag. **That also stops the Usurper's invasions.**
- Not live-tested.

## 6. Magerold of Lanafir (added 2026-11-04)

**Magerold never invades on PC.** Dev name 珍品屋 ("curio dealer"). He has two EnemyParam rows: 772000, alive (Iron
Keep generator 175), and 772001, dead (176). Neither is spawned by 1:131741, and no other map places a 772xxx row or
his name text 7720. His generator flags are the usual NPC ones: HostileEventID 103850 (he turns on you if attacked)
and death 104350. Only his own talk, grave and key-guide events use them. His Traveling Merchant set
(NpcPlayerStatusParam 7720) is also worn by Rapacious Andrei (8690), a DLC summon, not an invader.
