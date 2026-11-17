# Network Test AI scripts for enemies the test never places: Japanese names and translations

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Written 2026-10-25 (`tools\nt_ai_labels.py` regenerates it). The author asked whether the Network Test's AI scripts for enemies found only in the main game carry names or only numbers.

**Answer: both.** The file name carries the six-digit enemy / logic id (`ai205000`), and the file's own header holds the developers' working label in Japanese (`ai205000_毒角虫`). No in-game text names these enemies; these labels are the only enemy names in the data (`ainames.py`).

## What is listed

- **306 scripts, 147 families** (first four digits of the id): every script that exists in both the Network Test (631 AI scripts) and retail v1.00 (427) and that no Network Test generator names (only 33 AI ids are used by the test's own maps). So these are enemies and NPCs retail places and the test does not. They are not all enemies: the 700000s are NPCs and merchants and the 600000s are mostly bosses.
- **6 families have a script but no enemy row at all in the Network Test's regulation** (marked ⚑ below): rat king (2265), knight of life (3350), knight of death (3360), gravekeeper (5060), generic NPC (6900) and the weapon seller (7760).
- Not listed: the Network Test's own developer test scripts (`ai0095xx`, `ai0099xx`, some labelled with staff surnames) and the scripts only the Network Test has (variants such as `ai100011`).

## How to read it

- **Script**: `aiNNNNNN`. A pair per enemy: the base script ends `000`, the paired `上位` (upper) script usually `010`; variants use other tens digits (`030` ambush, `040` route patrol ...).
- **Label prefix** 下位 = *lower logic*, 上位 = *upper logic* (as in `agdayne-darklurker.md`). Where 上位 sits inside a name (`上位武器屋`, "high-grade weapon seller") it is part of the name.
- **亡者** (literally "dead person") is the developers' word for a hollow, **生者** ("living person") for the living version of the same enemy; **（イモータル）** = immortal, **（寄生）** = parasitised.
- **Translation** is mine, best effort; a trailing `?` marks a guess, `[sic]` a probable typo in the original.
- **Community sheet name** is what the community's SOTFS enemy sheet (`Misc\...Comprehensive Enemy Defenses V2.xlsx`) calls the enemy with exactly this id, when it has one (that is the retail name, not part of the label). It is a cross-reference only: ids can carry a different name in retail, and the sheet lists one row per variant.

## Developers' names that differ from retail's

| Script | Label | Developers' name | Retail name (community sheet, same id) |
|---|---|---|---|
| `ai625000` | オーンスタイン | Ornstein | Old Dragonslayer (Boss) |
| `ai326000` | ブッチャー | butcher | The Rotten (Boss) |
| `ai305000` | 下位：デーモンナイト | lower logic: demon knight | Smelter Demon (Boss) |
| `ai602000` | 美声カエル | sweet-voiced frog | Demon of Song (Boss) |
| `ai603000` | 蜘蛛の女王 | spider queen | The Duke's Dear Freja (Boss) |
| `ai607000` | 熔鉄デーモン | molten-iron demon | Old Iron King (Boss) |
| `ai318000` | 下位：ベイグランド | lower logic: Beigrand (reading of the name uncertain) | The Pursuer - Drangleic Castle |
| `ai325000` | 鏡の中の戦士 | warrior in the mirror | Ruin Sentinel (Boss) |
| `ai309600` | 下位：新・巨人上級兵士（亡者） | lower logic: new giant senior soldier (hollow) | Last Giant (Boss) |
| `ai611010` | 上位：重騎馬兵：兵士のみ | upper logic: heavy cavalry: soldier only (presumably the rider without the mount)? | Dragonrider (Bow) (Boss) - Drangleic Castle |
| `ai303300` | 下位：両面戦士：ヘビ頭 | lower logic: two-faced warrior: snake head | Flexile Sentry (Boss) |
| `ai332000` | 王妃の騎士A | queen's knight A | Throne Defender (Boss) |
| `ai333000` | 王妃の騎士Ｂ | queen's knight B | Velstadt, the Royal Aegis (Boss) |
| `ai334000` | 王妃の騎士Ｃ | queen's knight C | Throne Watcher (Boss) |
| `ai725000` | 下位：深淵の巡礼者 | lower logic: pilgrim of the abyss | Darkdiver Grandahl - Shaded Woods |
| `ai743000` | 下位：月光爺 | lower logic: moonlight old man | Benhart of Jugo - Forest of Fallen Giants |
| `ai741000` | 下位：心折れた青教戦士 | lower logic: broken-hearted Blue Sect warrior | Crestfallen Saulden |
| `ai742000` | 下位：仇討ちの戦士 | lower logic: warrior seeking vengeance | Creighton the Wanderer - Brightstone Cove Tseldora |
| `ai752000` | 下位：女騎士 | lower logic: woman knight | Lucatiel of Mirrah - Aldia’s Keep |

The three "queen's knights" are A / B / C in the labels; the sheet names them Throne Defender, Velstadt and Throne Watcher. The queen (王妃, scripts 703000 / 702010) is presumably Nashandra, but no label says so.

## Labels that changed between the Network Test and retail (14)

| Script | Network Test label | Translation | Retail v1.00 label | Translation |
|---|---|---|---|---|
| `ai102040` | 上位：王国兵士（亡者）：固定砲台（領域） | upper logic: kingdom soldier (hollow): fixed turret (zone) | 上位：王国兵士（亡者）：弓持ち替え射撃 | upper logic: kingdom soldier (hollow): switches to a bow and shoots |
| `ai102052` | 上位：王国兵士（亡者）：領域＆視線奇襲 | upper logic: kingdom soldier (hollow): zone, ambush from line of sight | 上位：王国兵士（亡者）：死体擬態（グループ無） | upper logic: kingdom soldier (hollow): plays dead (no group) |
| `ai103041` | 上位：下級兵士（亡者）：ランダム徘徊 | upper logic: lower-rank soldier (hollow): wanders at random | 上位：下級兵士（亡者）：ダッシュ追跡 | upper logic: lower-rank soldier (hollow): dash pursuit |
| `ai113020` | 上位：水兵（亡者）：汎用（FC用） | upper logic: sailor (hollow): generic (for 'FC', not explained) | 上位：水兵（亡者）：椅子で寝てる | upper logic: sailor (hollow): asleep in a chair |
| `ai151040` | 上位：王国重兵士（亡者）：グループリーダー | upper logic: kingdom heavy soldier (hollow): group leader | 上位：王国重兵士（亡者）：狭路戦用 | upper logic: kingdom heavy soldier (hollow): for fighting in narrow paths |
| `ai203030` | 上位：パラサイトスパイダー：松明起動判定 | upper logic: parasite spider: torch-activation check | 上位：パラサイトスパイダー：音反応 | upper logic: parasite spider: reacts to sound |
| `ai203050` | 上位：パラサイトスパイダー：接地せずに起動 | upper logic: parasite spider: activates without touching the ground | 上位：パラサイトスパイダー：ボス用 | upper logic: parasite spider: for a boss |
| `ai307000` | 下位：デュラハン | lower logic: dullahan | 下位：デュラハン（亡者） | lower logic: dullahan (hollow) |
| `ai307010` | 上位：デュラハン | upper logic: dullahan | 上位：デュラハン（亡者）：汎用 | upper logic: dullahan (hollow): generic |
| `ai307100` | デュラハン（生者） | dullahan (living) | 下位：デュラハン | lower logic: dullahan |
| `ai331000` | デーモン重兵士 | demon heavy soldier | デーモンの重戦士 | demon's heavy warrior |
| `ai337000` | すごい亡者 | formidable hollow | すごい亡者：クズ底 | formidable hollow: dregs / bottom tier? |
| `ai771000` | 下位：スペルマスター | lower logic: spell master | 下位：封じられた人 | lower logic: sealed person |
| `ai771010` | 上位：スペルマスター | upper logic: spell master | 上位：封じられた人 | upper logic: sealed person |

Seven are variant scripts (102040, 102052, 103041, 113020, 151040, 203030, 203050) whose behaviour label differs; the dullahan trio (307000 / 307010 / 307100) splits hollow and living differently; the other four are renames or longer names (331000, 337000, 771000, 771010). Which build wrote which label first is not known from the labels alone.

## 100000s: soldiers, humanoids and statues (95 scripts)

| Script | Japanese label | Translation | Sheet name (same id) | Note |
|---|---|---|---|---|
| `ai100000` | ゴブリン（生者） | goblin (living) | Forest Grotesque (Unarmed) |  |
| `ai100010` | 上位：ゴブリン（生者）：汎用 | upper logic: goblin (living): generic | Forest Grotesque (Bone Club) |  |
| `ai101000` | コボルト（生者） | kobold (living) | Kobold - Things Betwixt |  |
| `ai101010` | 上位：コボルト（生者）：汎用 | upper logic: kobold (living): generic |  |  |
| `ai102000` | 王国兵士（亡者） | kingdom soldier (hollow) | Hollow Soldier (Spear & Shield) |  |
| `ai102010` | 上位：王国兵士（亡者）：汎用 | upper logic: kingdom soldier (hollow): generic | Hollow Soldier (Bastard Sword) |  |
| `ai102030` | 上位：王国兵士（亡者）：領域＆視認即反応 | upper logic: kingdom soldier (hollow): zone, reacts the moment it sees you | Hollow Soldier (Estoc & Shield) |  |
| `ai102040` | 上位：王国兵士（亡者）：固定砲台（領域） | upper logic: kingdom soldier (hollow): fixed turret (zone) | Hollow Soldier (Bow) |  |
| `ai102052` | 上位：王国兵士（亡者）：領域＆視線奇襲 | upper logic: kingdom soldier (hollow): zone, ambush from line of sight |  |  |
| `ai102053` | 上位：王国兵士（亡者）：死体擬態：近距離 | upper logic: kingdom soldier (hollow): plays dead (corpse mimicry): close range |  |  |
| `ai102054` | 上位：王国兵士（亡者）：死体擬態：宝死体 | upper logic: kingdom soldier (hollow): plays dead (corpse mimicry): treasure corpse |  |  |
| `ai102055` | 上位：王国兵士（亡者）：攻撃待ち（検証用） | upper logic: kingdom soldier (hollow): waits to attack (for verification) |  |  |
| `ai102090` | 上位：王国兵士（亡者）：宝箱罠（ロジックフラグ起動） | upper logic: kingdom soldier (hollow): treasure-chest trap (started by logic flag) |  |  |
| `ai102092` | 上位：王国兵士（亡者）：宝死体（ロジックフラグ起動） | upper logic: kingdom soldier (hollow): treasure corpse (started by logic flag) |  |  |
| `ai102100` | 王国兵士（生者） | kingdom soldier (living) | Royal Soldier (Spear & Shield) |  |
| `ai102110` | 上位：王国兵士（生者）：汎用 | upper logic: kingdom soldier (living): generic | Royal Soldier (Bastard Sword) |  |
| `ai103000` | 下級兵士（亡者） | lower-rank soldier (hollow) | Hollow Infantry (Unarmed) |  |
| `ai103010` | 上位：下級兵士（亡者）：汎用 | upper logic: lower-rank soldier (hollow): generic | Hollow Infantry (Bandit's Knife) |  |
| `ai103030` | 上位：下級兵士（亡者）：領域：視認即反応 | upper logic: lower-rank soldier (hollow): zone: reacts the moment it sees you | Hollow Infantry (Axe) - Forest of Fallen Giants |  |
| `ai103040` | 上位：下級兵士（亡者）：ルート徘徊 | upper logic: lower-rank soldier (hollow): patrols a route | Hollow Infantry (Bow) - Forest of Fallen Giants |  |
| `ai103041` | 上位：下級兵士（亡者）：ランダム徘徊 | upper logic: lower-rank soldier (hollow): wanders at random |  |  |
| `ai103050` | 上位：下級兵士（亡者）：死体擬態 | upper logic: lower-rank soldier (hollow): plays dead (corpse mimicry) |  |  |
| `ai103051` | 上位：下級兵士（亡者）：死体擬態（グループ無） | upper logic: lower-rank soldier (hollow): plays dead (no group) | Hollow Infantry (Bow) - No-man's Wharf |  |
| `ai103055` | 上位：下級兵士（亡者）：攻撃待ち（検証用） | upper logic: lower-rank soldier (hollow): waits to attack (for verification) |  |  |
| `ai103090` | 上位：下級兵士（亡者）：BS待ち | upper logic: lower-rank soldier (hollow): waits for a backstab (BS)? |  |  |
| `ai103100` | 下級兵士（生者） | lower-rank soldier (living) |  |  |
| `ai103110` | 上位：下級兵士 | upper logic: lower-rank soldier |  |  |
| `ai105000` | 妖精の巫女（亡者） | fairy shrine maiden (hollow) | Amana Shrine Maiden (Healer) |  |
| `ai105010` | 上位：妖精の巫女（亡者）：汎用 | upper logic: fairy shrine maiden (hollow): generic | Amana Shrine Maiden (Sorceress) |  |
| `ai105030` | 上位：妖精の巫女（亡者）：待ち伏せ | upper logic: fairy shrine maiden (hollow): ambush |  |  |
| `ai106000` | ダークプリースト（亡者） | dark priest (hollow) |  |  |
| `ai106010` | 上位：ダークプリースト（亡者）：汎用 | upper logic: dark priest (hollow): generic |  |  |
| `ai107000` | パラサイトスパイダー（寄生） | parasite spider (parasitised) | Parasitized Undead |  |
| `ai107010` | 上位：パラサイトスパイダー（寄生）：汎用 | upper logic: parasite spider (parasitised): generic |  |  |
| `ai107030` | 上位：パラサイトスパイダー（寄生）：即起動 | upper logic: parasite spider (parasitised): activates at once |  |  |
| `ai113000` | 水兵（亡者） | sailor (hollow) | Varangian Sailor (Sword & Shield) |  |
| `ai113010` | 上位：水兵（亡者） | upper logic: sailor (hollow) | Varangian Sailor (Scimitars) |  |
| `ai113020` | 上位：水兵（亡者）：汎用（FC用） | upper logic: sailor (hollow): generic (for 'FC', not explained) | Varangian Sailor (Bow) | 'FC' is not expanded anywhere in the data. |
| `ai113030` | 上位：水兵（亡者）：音反応 | upper logic: sailor (hollow): reacts to sound |  |  |
| `ai115000` | 亡者_旅人の服 | hollow in traveller's clothes | Undead Traveler (Unarmed) - Statues |  |
| `ai115010` | 上位：亡者_旅人の服：汎用 | upper logic: hollow in traveller's clothes: generic |  |  |
| `ai117000` | スタチュー：ボーン | statue: bone [sic, see note] | Stone Soldier (Spear & Shield) | ボーン (bone) is almost certainly a typo for ポーン (pawn), spelt correctly in ai117010. The statues carry chess-piece names (pawn, knight). |
| `ai117010` | 上位：スタチュー：ポーン | upper logic: statue: pawn | Stone Soldier (Spear) |  |
| `ai119000` | 獅子黒魔道士（生者） | lion black mage (living)? |  |  |
| `ai119010` | 上位：獅子黒魔道士（生者） | upper logic: lion black mage (living)? |  |  |
| `ai121000` | 巨人下級兵士 | giant lower-rank soldier | Giant (Unarmed) |  |
| `ai121010` | 上位：巨人下級兵士：汎用 | upper logic: giant lower-rank soldier: generic | Giant (Club) |  |
| `ai123000` | 異形の影（生者） | shadow of a deformed one (living)? |  |  |
| `ai123010` | 上位：異形の影（生者） | upper logic: shadow of a deformed one (living)? |  |  |
| `ai124000` | 暗殺者（生者） | assassin (living) | Manikin (Claws) - Harvest Valley/Earthen Peak |  |
| `ai124010` | 上位：暗殺者（生者） | upper logic: assassin (living) | Manikin (Knife & Shield) |  |
| `ai125000` | アンデッド市民 | undead citizen |  |  |
| `ai125010` | 上位：アンデッド市民：汎用 | upper logic: undead citizen: generic | Rupturing Hollow (Red Core) - Lost Bastille |  |
| `ai129000` | 森の番人 | forest keeper | Forest Spirit - Dark Chasm of Old |  |
| `ai129010` | 上位：森の番人 | upper logic: forest keeper |  |  |
| `ai131000` | クレリック（亡者） | cleric (hollow) | Lindelt Cleric |  |
| `ai131010` | 上位：クレリック（亡者）：汎用 | upper logic: cleric (hollow): generic |  |  |
| `ai133010` | 上位：車輪スケルトン | upper logic: wheel skeleton |  |  |
| `ai134000` | ドワーフ（亡者） | dwarf (hollow) | Gyrm |  |
| `ai134010` | 上位：ドワーフ | upper logic: dwarf |  |  |
| `ai135000` | 重ドワーフ（亡者） | heavy dwarf (hollow) | Gyrm Warrior (All Variants) |  |
| `ai135010` | 上位：重ドワーフ | upper logic: heavy dwarf |  |  |
| `ai135090` | 上位：重ドワーフ（亡者）：GDDテスト用 | upper logic: heavy dwarf (hollow): for 'GDD' testing (not explained) |  | 'GDD' is not expanded anywhere in the data. |
| `ai135091` | 上位：重ドワーフ（亡者）：GDD高台砲台 | upper logic: heavy dwarf (hollow): 'GDD' high-ground turret (not explained) |  |  |
| `ai137000` | ネクロマンサー（イモータル） | necromancer (immortal) |  |  |
| `ai137010` | 上位：ネクロマンサー（イモータル）：汎用 | upper logic: necromancer (immortal): generic | Prowling Magus - Aldia's Keep |  |
| `ai138010` | 上位：拷問人：汎用 | upper logic: torturer: generic | Torturer (Whip) |  |
| `ai140000` | ベルカの鴉人 | Velka's crow-person |  |  |
| `ai140010` | 上位：ベルカの鴉人：汎用 | upper logic: Velka's crow-person: generic |  |  |
| `ai141000` | 強化不死人 | reinforced undead | Undead Aberration - Aldia's Keep |  |
| `ai141010` | 上位：強化不死人：汎用 | upper logic: reinforced undead: generic |  |  |
| `ai146000` | 亡者_富裕層の服 | hollow in wealthy-class clothes |  |  |
| `ai146010` | 上位：富裕層の服：汎用 | upper logic: wealthy-class clothes: generic | Lord Tseldora |  |
| `ai147000` | 亡者_病人の服 | hollow in sick-person's clothes |  |  |
| `ai147010` | 上位：亡者_病人の服：汎用 | upper logic: hollow in sick-person's clothes: generic |  |  |
| `ai148000` | 亡者：農民の服 | hollow: peasant's clothes | Undead Peasant (Pickaxe) |  |
| `ai148010` | 上位：亡者：農民の服：汎用 | upper logic: hollow: peasant's clothes: generic |  |  |
| `ai149000` | 亡者：製鉄所所員 | hollow: ironworks employee |  |  |
| `ai149010` | 上位：亡者：製鉄所所員：汎用 | upper logic: hollow: ironworks employee: generic | Undead Steelworker - Harvest Valley |  |
| `ai150000` | スタチュー：ナイト | statue: knight | Stone Knight |  |
| `ai150010` | 上位：スタチュー：ナイト：汎用 | upper logic: statue: knight: generic |  |  |
| `ai150011` | 上位：スタチュー：ナイト：領域待機 | upper logic: statue: knight: waits in zone |  |  |
| `ai150012` | 上位：スタチュー：ナイト：領域待機2 | upper logic: statue: knight: waits in zone 2 |  |  |
| `ai150013` | 上位：スタチュー：ナイト：領域待機3 | upper logic: statue: knight: waits in zone 3 |  |  |
| `ai150014` | 上位：スタチュー：ナイト：領域待機4 | upper logic: statue: knight: waits in zone 4 |  |  |
| `ai151000` | 王国重兵士（亡者） | kingdom heavy soldier (hollow) | Ironclad Soldier - Iron Keep |  |
| `ai151010` | 上位：王国重兵士（亡者）：汎用 | upper logic: kingdom heavy soldier (hollow): generic |  |  |
| `ai151040` | 上位：王国重兵士（亡者）：グループリーダー | upper logic: kingdom heavy soldier (hollow): group leader |  |  |
| `ai152000` | 王国兵戦士 | kingdom soldier-warrior | Royal Swordsman (Greatsword) - Drangleic Castle |  |
| `ai152010` | 上位：王国兵戦士：汎用 | upper logic: kingdom soldier-warrior: generic | Royal Swordsman (Crossbow) - Drangleic Castle |  |
| `ai153000` | 王国護衛兵 | kingdom guard | Syan Knight (Spear) - Drangleic Castle |  |
| `ai153010` | 上位：王国護衛兵：汎用 | upper logic: kingdom guard: generic | Syan Knight (Sword) - Forest of Fallen Giants / Shaded Woods / Drangleic Castle |  |
| `ai155000` | トカゲ人 | lizard person | Amana Aberration |  |
| `ai155010` | 上位：トカゲ人：汎用 | upper logic: lizard person: generic |  |  |
| `ai155030` | 上位：トカゲ人：音反応 | upper logic: lizard person: reacts to sound |  |  |

## 200000s: animals and creatures (47 scripts)

| Script | Japanese label | Translation | Sheet name (same id) | Note |
|---|---|---|---|---|
| `ai201000` | 奴隷獣（溶鉄壺有り） | slave beast (with molten-iron pot) |  |  |
| `ai201010` | 上位：奴隷獣（溶鉄壺有り）：汎用 | upper logic: slave beast (with molten-iron pot): generic |  |  |
| `ai201100` | 奴隷獣：子豚 | slave beast: piglet | Small Boar |  |
| `ai202000` | 奴隷獣（熔鉄壺無し） | slave beast (without molten-iron pot) |  |  |
| `ai202010` | 上位：奴隷獣（熔鉄壺無し）：汎用 | upper logic: slave beast (without molten-iron pot): generic |  |  |
| `ai203000` | パラサイトスパイダー | parasite spider | Parasite Spider |  |
| `ai203010` | 上位：パラサイトスパイダー：汎用 | upper logic: parasite spider: generic |  |  |
| `ai203012` | 上位：パラサイトスパイダー：領域奇襲 | upper logic: parasite spider: zone ambush |  |  |
| `ai203030` | 上位：パラサイトスパイダー：松明起動判定 | upper logic: parasite spider: torch-activation check |  |  |
| `ai203040` | 上位：パラサイトスパイダー：即起動 | upper logic: parasite spider: activates at once |  |  |
| `ai203050` | 上位：パラサイトスパイダー：接地せずに起動 | upper logic: parasite spider: activates without touching the ground |  |  |
| `ai204000` | グレートモス | great moth | Poison Moth |  |
| `ai204010` | 上位：グレートモス：汎用 | upper logic: great moth: generic |  |  |
| `ai205000` | 毒角虫 | poison horn bug | Poison Horn Beetle - Earthen Peak |  |
| `ai205010` | 上位：毒角虫：汎用 | upper logic: poison horn bug: generic |  |  |
| `ai206000` | 針ウジ蟲 | needle maggot | Razorback Nightcrawler - Black Gulch |  |
| `ai206010` | 上位：針ウジ蟲：汎用 | upper logic: needle maggot: generic |  |  |
| `ai209000` | 狩猟犬 | hunting dog | Hunting Dog - Aldia's Keep |  |
| `ai209010` | 上位：狩猟犬：汎用 | upper logic: hunting dog: generic |  |  |
| `ai210000` | バジリスク | basilisk | Basilisk - Things Betwixt / Old Akelarre / Shaded Woods |  |
| `ai210010` | 上位：バジリスク：汎用 | upper logic: basilisk: generic | Basilisk - Brightstone Cove Tseldora |  |
| `ai212000` | ワイバーン | wyvern |  |  |
| `ai212010` | 上位：ワイバーン：汎用 | upper logic: wyvern: generic |  |  |
| `ai213000` | 石守 | stone guardian? | Crystal Lizard - Forest of Fallen Giants / Huntsman's Copse |  |
| `ai213010` | 上位：石守：汎用 | upper logic: stone guardian?: generic | Crystal Lizard - Harvest Valley |  |
| `ai213100` | 石守：爆発 | stone guardian?: explosion | Red Crystal Lizard - Brightstone Cove Tseldora / Aldia's Keep |  |
| `ai214000` | 牙獣 | fang beast |  |  |
| `ai214010` | 上位：牙獣：汎用 | upper logic: fang beast: generic |  |  |
| `ai216000` | 扉憑き | door-haunter? | Wall Ghost - Undead Crypt |  |
| `ai216010` | 上位：扉憑き：汎用 | upper logic: door-haunter?: generic |  |  |
| `ai217000` | 闇に隠れし者 | one who hides in the dark | Dark Stalker - No-man's Wharf |  |
| `ai217010` | 上位：闇に隠れし者：汎用 | upper logic: one who hides in the dark: generic |  |  |
| `ai220000` | 巨大毒角虫・酸 | giant poison horn bug, acid |  |  |
| `ai220010` | 上位：巨大毒角虫：汎用 | upper logic: giant poison horn bug: generic |  |  |
| `ai222000` | 巨大バジリスク | giant basilisk | Giant Basilisk - Aldia's Keep |  |
| `ai222010` | 上位：巨大バジリスク：汎用 | upper logic: giant basilisk: generic |  |  |
| `ai223000` | 死体あさりネズミ | corpse-scavenging rat | Mongrel Rat |  |
| `ai223010` | 上位：死体あさりネズミ：汎用 | upper logic: corpse-scavenging rat: generic |  |  |
| `ai224000` | コールタール | coal tar | Darksucker |  |
| `ai224010` | 上位：コールタール：汎用 | upper logic: coal tar: generic |  |  |
| `ai224030` | 上位：コールタール：奇襲 | upper logic: coal tar: ambush |  |  |
| `ai226000` | 犬ネズミ | dog rat | Corpse Rat |  |
| `ai226020` | 上位：犬ネズミ：ボス用 | upper logic: dog rat: for a boss |  |  |
| `ai226030` | 上位：犬ネズミ：ボス取り巻き用 | upper logic: dog rat: for a boss's entourage |  |  |
| `ai226500` | ネズミ王 | rat king |  | ⚑ no enemy row in the Network Test |
| `ai227000` | 野犬 | wild dog | Stray Dog - Lost Bastille / Belfry Luna |  |
| `ai227010` | 上位：野犬：汎用 | upper logic: wild dog: generic |  |  |

## 300000s: larger enemies, knights, giants and some bosses (55 scripts)

| Script | Japanese label | Translation | Sheet name (same id) | Note |
|---|---|---|---|---|
| `ai300000` | オーガ | ogre | Ogre - Things Betwixt / Forest of Fallen Giants |  |
| `ai300010` | 上位：オーガ：汎用 | upper logic: ogre: generic |  |  |
| `ai300030` | 上位：オーガ：聴覚反応 | upper logic: ogre: reacts to hearing |  |  |
| `ai300040` | 上位：オーガ：石化ストップ | upper logic: ogre: stops when turned to stone? |  |  |
| `ai301000` | 彷徨える騎士 | wandering knight |  |  |
| `ai301010` | 上位：彷徨える騎士：汎用 | upper logic: wandering knight: generic | Heide Knight (Spear) - Sinner's Rise |  |
| `ai302000` | 獄吏（亡者） | jailer (hollow) | Undead Jailer - No-man’s Wharf / Lost Bastille |  |
| `ai302010` | 上位：獄吏：汎用 | upper logic: jailer: generic |  |  |
| `ai303300` | 下位：両面戦士：ヘビ頭 | lower logic: two-faced warrior: snake head | Flexile Sentry (Boss) |  |
| `ai303310` | 上位：両面戦士：ヘビ頭 | upper logic: two-faced warrior: snake head |  |  |
| `ai304000` | バンシー（亡者） | banshee (hollow) |  |  |
| `ai304010` | 上位：バンシー（亡者）：汎用 | upper logic: banshee (hollow): generic |  |  |
| `ai305000` | 下位：デーモンナイト | lower logic: demon knight | Smelter Demon (Boss) | Retail's Smelter Demon; the developers' name is Demon Knight. |
| `ai305010` | 上位：デーモンナイト | upper logic: demon knight |  |  |
| `ai306000` | デーモン鎧（亡者） | demon armour (hollow) | Alonne Captain (Katana) - Iron Keep |  |
| `ai306010` | 上位：デーモン鎧：汎用 | upper logic: demon armour: generic | Alonne Captain (Greatbow) - Iron Keep |  |
| `ai307000` | 下位：デュラハン | lower logic: dullahan |  |  |
| `ai307010` | 上位：デュラハン | upper logic: dullahan |  |  |
| `ai307100` | デュラハン（生者） | dullahan (living) |  |  |
| `ai308000` | 獅子戦士 | lion warrior | Lion Clan Warrior |  |
| `ai308010` | 上位：獅子戦士：汎用 | upper logic: lion warrior: generic |  |  |
| `ai309000` | 巨人上級兵士 | giant senior soldier | Forgotten Giant - Giant Memories |  |
| `ai309010` | 上位：巨人上級兵士：汎用 | upper logic: giant senior soldier: generic |  |  |
| `ai309600` | 下位：新・巨人上級兵士（亡者） | lower logic: new giant senior soldier (hollow) | Last Giant (Boss) |  |
| `ai309610` | 上位：新・巨人上級兵士（亡者） | upper logic: new giant senior soldier (hollow) |  |  |
| `ai311000` | 騎人兵：騎乗兵士 | rider soldier?: mounted soldier | Mounted Overseer |  |
| `ai312000` | イモータル上級兵士（イモータル） | immortal senior soldier (immortal) |  |  |
| `ai312010` | 上位：イモータル上級兵士（イモータル）：汎用 | upper logic: immortal senior soldier (immortal): generic |  |  |
| `ai313000` | 鷹匠戦士 | falconer warrior |  |  |
| `ai314000` | 象人（亡者） | elephant person (hollow) | Hollow Primal Knight - Doors of Pharros |  |
| `ai315000` | 象人（生者） | elephant person (living) | Primal Knight - Drangleic Castle |  |
| `ai316000` | 捕らわれた魔道士（生者） | captive mage (living) | Desert Sorceress - Drangleic Castle |  |
| `ai316010` | 上位：捕らわれた魔道士（生者）：汎用 | upper logic: captive mage (living): generic |  |  |
| `ai317000` | 龍の教授 | dragon professor? | Dragon Acolyte - Aldia’s Keep |  |
| `ai318000` | 下位：ベイグランド | lower logic: Beigrand (reading of the name uncertain) | The Pursuer - Drangleic Castle | Retail's The Pursuer; ベイグランド is the developers' name for him. |
| `ai318010` | 上位：ベイグランド | upper logic: Beigrand (reading of the name uncertain) | The Pursuer - Forest of Fallen Giants (Boss) |  |
| `ai319000` | デーモン鎧（量産型） | demon armour (mass-production model) | Alonne Knight (Katana) - Iron Keep |  |
| `ai319010` | 上位：デーモン鎧（量産型）：汎用 | upper logic: demon armour (mass-production model): generic | Alonne Knight (Greatbow) - Iron Keep |  |
| `ai321000` | νミミック | nu mimic | Mimic (Wooden Chest) - Aldia's Keep |  |
| `ai321010` | 上位：νミミック：汎用 | upper logic: nu mimic: generic | Mimic (Metal Chest) - Aldia's Keep |  |
| `ai324000` | ガーゴイル | gargoyle | Belfry Gargoyle (Boss) |  |
| `ai324010` | 上位：ガーゴイル：汎用 | upper logic: gargoyle: generic | Belfry Gargoyle - Drangleic Castle |  |
| `ai325000` | 鏡の中の戦士 | warrior in the mirror | Ruin Sentinel (Boss) |  |
| `ai325010` | 上位：鏡の中の戦士：汎用 | upper logic: warrior in the mirror: generic | Ruin Sentinel - Drangleic Castle |  |
| `ai326000` | ブッチャー | butcher | The Rotten (Boss) | Retail's The Rotten; the developers' name is Butcher. |
| `ai327000` | 骨ドラゴン | bone dragon |  |  |
| `ai327010` | 上位：骨ドラゴン：イベント専用 | upper logic: bone dragon: event only |  |  |
| `ai330000` | 古代の戦士 | ancient warrior | Old Knight (Sword & Shield) - Heide's Tower of Flame |  |
| `ai331000` | デーモン重兵士 | demon heavy soldier |  |  |
| `ai332000` | 王妃の騎士A | queen's knight A | Throne Defender (Boss) |  |
| `ai333000` | 王妃の騎士Ｂ | queen's knight B | Velstadt, the Royal Aegis (Boss) |  |
| `ai334000` | 王妃の騎士Ｃ | queen's knight C | Throne Watcher (Boss) |  |
| `ai335000` | 生の騎士 | knight of life |  | ⚑ no enemy row in the Network Test |
| `ai336000` | 死の騎士 | knight of death |  | ⚑ no enemy row in the Network Test |
| `ai337000` | すごい亡者 | formidable hollow | Captive Undead (Unarmed) - Huntsman's Copse |  |

## 500000s: man-scorpions, phantoms, the gravekeeper (13 scripts)

| Script | Japanese label | Translation | Sheet name (same id) | Note |
|---|---|---|---|---|
| `ai502000` | 下位：マンスコーピオン（男） | lower logic: man-scorpion (male) | Manscorpion Tark (NPC) |  |
| `ai502010` | 上位：マンスコーピオン（男） | upper logic: man-scorpion (male) |  |  |
| `ai503000` | マンスコーピオン♀ | man-scorpion (female) | Scorpioness Najka (Boss) |  |
| `ai503010` | 上位：マンスコーピオン♀ | upper logic: man-scorpion (female) |  |  |
| `ai504000` | ミラーナイト | mirror knight | Looking Glass Knight (Boss) |  |
| `ai504010` | 上位：ミラーナイト | upper logic: mirror knight |  |  |
| `ai506000` | 下位：墓守 | lower logic: gravekeeper |  | ⚑ no enemy row in the Network Test; agdayne-darklurker.md: retail kept this pair for the grave-warden NPC. |
| `ai506010` | 上位：墓守 | upper logic: gravekeeper |  | ⚑ no enemy row in the Network Test; agdayne-darklurker.md: retail kept this pair for the grave-warden NPC. |
| `ai508000` | レイス | wraith |  |  |
| `ai511000` | ファントムナイト | phantom knight | Imperious Knight |  |
| `ai512000` | ファントムメイジ | phantom mage | Leydia Pyromancer |  |
| `ai512010` | 上位：ファントムメイジ：汎用 | upper logic: phantom mage: generic |  |  |
| `ai512030` | 上位：ファントムメイジ：音反応 | upper logic: phantom mage: reacts to sound |  |  |

## 600000s: bosses and a few others (16 scripts)

| Script | Japanese label | Translation | Sheet name (same id) | Note |
|---|---|---|---|---|
| `ai600000` | 古の龍 | ancient dragon | Ancient Dragon (Boss) |  |
| `ai600010` | 上位：古の龍 | upper logic: ancient dragon |  |  |
| `ai601000` | サラマンダー | salamander | Flame Lizard - Forest of Fallen Giants |  |
| `ai601010` | 上位：サラマンダー | upper logic: salamander |  |  |
| `ai601030` | 上位：サラマンダー：固定砲台 | upper logic: salamander: fixed turret |  |  |
| `ai602000` | 美声カエル | sweet-voiced frog | Demon of Song (Boss) | Retail's Demon of Song; the developers' name is sweet-voiced frog. |
| `ai602010` | 上位：美声カエル | upper logic: sweet-voiced frog |  |  |
| `ai603000` | 蜘蛛の女王 | spider queen | The Duke's Dear Freja (Boss) | Retail's The Duke's Dear Freja; the developers' name is spider queen. |
| `ai603020` | 上位：蜘蛛の女王：初遭遇イベント | upper logic: spider queen: first-encounter event |  |  |
| `ai607000` | 熔鉄デーモン | molten-iron demon | Old Iron King (Boss) | Retail's Old Iron King; the developers' name is molten-iron demon. |
| `ai607010` | 上位：熔鉄デーモン | upper logic: molten-iron demon |  |  |
| `ai611000` | 下位：重騎馬兵：兵士のみ | lower logic: heavy cavalry: soldier only (presumably the rider without the mount)? |  |  |
| `ai611010` | 上位：重騎馬兵：兵士のみ | upper logic: heavy cavalry: soldier only (presumably the rider without the mount)? | Dragonrider (Bow) (Boss) - Drangleic Castle |  |
| `ai625000` | オーンスタイン | Ornstein | Old Dragonslayer (Boss) | Retail's Old Dragonslayer; the developers' name is Ornstein. |
| `ai625010` | 上位：オーンスタイン：汎用 | upper logic: Ornstein: generic |  |  |
| `ai690000` | 上位：ＮＰＣ汎用 | upper logic: generic NPC | Burnt Ivory King (Boss) | ⚑ no enemy row in the Network Test; The community sheet names the same id Burnt Ivory King; the label is a generic NPC script. |

## 700000s: NPCs and merchants (80 scripts)

| Script | Japanese label | Translation | Sheet name (same id) | Note |
|---|---|---|---|---|
| `ai700000` | 下位：巫女 | lower logic: shrine maiden |  |  |
| `ai700010` | 上位：巫女 | upper logic: shrine maiden |  |  |
| `ai701000` | 下位：巫女（子供） | lower logic: shrine maiden (child) |  |  |
| `ai701010` | 上位：巫女（子供） | upper logic: shrine maiden (child) |  |  |
| `ai702000` | 下位：巫女（死骸） | lower logic: shrine maiden (corpse) |  |  |
| `ai702010` | 上位：王妃（死骸） | upper logic: queen (corpse) |  | The lower script says shrine maiden (corpse); the upper one says queen (corpse). |
| `ai703000` | 下位：王妃 | lower logic: queen |  |  |
| `ai703010` | 上位：王妃 | upper logic: queen |  |  |
| `ai704000` | 下位：梯子屋 | lower logic: ladder seller |  |  |
| `ai704010` | 上位：梯子屋 | upper logic: ladder seller |  |  |
| `ai705000` | 下位：退役の火防女１ | lower logic: retired fire-keeper 1 |  |  |
| `ai705010` | 上位：退役の火防女１ | upper logic: retired fire-keeper 1 |  |  |
| `ai705100` | 下位：退役の火防女２ | lower logic: retired fire-keeper 2 |  |  |
| `ai705110` | 上位：退役の火防女２ | upper logic: retired fire-keeper 2 |  |  |
| `ai705300` | 下位：退役の火防女３ | lower logic: retired fire-keeper 3 |  |  |
| `ai705310` | 上位：退役の火防女３ | upper logic: retired fire-keeper 3 |  |  |
| `ai721100` | 下位：軍師の霊 | lower logic: strategist's spirit |  |  |
| `ai721110` | 上位：軍師の霊 | upper logic: strategist's spirit |  |  |
| `ai723000` | 下位：火防女の家政婦 | lower logic: fire-keeper's housekeeper | Milibeth |  |
| `ai723010` | 上位：火防女の家政婦 | upper logic: fire-keeper's housekeeper |  |  |
| `ai724000` | 下位：王国隊長 | lower logic: kingdom captain | Captain Drummond - Memory of Vammar |  |
| `ai724010` | 上位：王国隊長 | upper logic: kingdom captain |  |  |
| `ai725000` | 下位：深淵の巡礼者 | lower logic: pilgrim of the abyss | Darkdiver Grandahl - Shaded Woods |  |
| `ai725010` | 上位：深淵の巡礼者 | upper logic: pilgrim of the abyss |  |  |
| `ai726000` | 下位：ドワーフ | lower logic: dwarf |  |  |
| `ai726010` | 上位：ドワーフ | upper logic: dwarf |  |  |
| `ai730010` | 上位：ミラーナイトの下僕１ | upper logic: mirror knight's servant 1 |  |  |
| `ai741000` | 下位：心折れた青教戦士 | lower logic: broken-hearted Blue Sect warrior | Crestfallen Saulden |  |
| `ai741010` | 上位：心折れた青教戦士 | upper logic: broken-hearted Blue Sect warrior |  |  |
| `ai742000` | 下位：仇討ちの戦士 | lower logic: warrior seeking vengeance | Creighton the Wanderer - Brightstone Cove Tseldora |  |
| `ai742010` | 上位：仇討ちの戦士 | upper logic: warrior seeking vengeance |  |  |
| `ai743000` | 下位：月光爺 | lower logic: moonlight old man | Benhart of Jugo - Forest of Fallen Giants |  |
| `ai743010` | 上位：月光爺 | upper logic: moonlight old man | Benhart of Jugo (Summon) - Dragon's Rest |  |
| `ai744000` | 下位：パッチ | lower logic: Patches | Mild Mannered Pate - Forest of Fallen Giants |  |
| `ai744010` | 上位：パッチ | upper logic: Patches |  |  |
| `ai751000` | 下位：地図書き | lower logic: map-drawer | Cartographer Cale |  |
| `ai751010` | 上位：地図書き | upper logic: map-drawer |  |  |
| `ai752000` | 下位：女騎士 | lower logic: woman knight | Lucatiel of Mirrah - Aldia’s Keep |  |
| `ai752010` | 上位：女騎士 | upper logic: woman knight | Lucatiel of Mirrah (Summon) - Grand Cathedral |  |
| `ai753000` | 下位：鐘守 | lower logic: bell-keeper | Bell Keeper (NPC) - Belfry Luna |  |
| `ai753010` | 上位：鐘守 | upper logic: bell-keeper |  |  |
| `ai755000` | ストーカー戦士ウイックリー | stalker warrior Wickly (spelling uncertain)? |  | No sheet name; not identified. |
| `ai760000` | 下位：吟遊詩人の少女１ | lower logic: minstrel girl 1 | Milfanito - Shrine of Amana |  |
| `ai760010` | 上位：吟遊詩人の少女１ | upper logic: minstrel girl 1 |  |  |
| `ai760100` | 下位：吟遊詩人の少女２ | lower logic: minstrel girl 2 |  |  |
| `ai760110` | 上位：吟遊詩人の少女２ | upper logic: minstrel girl 2 |  |  |
| `ai760200` | 下位：虜囚の少女 | lower logic: captive girl | Milfanito - Drangleic Castle |  |
| `ai760210` | 上位：虜囚の少女 | upper logic: captive girl |  |  |
| `ai761000` | 下位：防具屋 | lower logic: armour seller | Maughlin the Armourer |  |
| `ai761010` | 上位：防具屋 | upper logic: armour seller |  |  |
| `ai762000` | 下位：素材屋 | lower logic: materials seller | Stone Trader Chloanne - Earthen Peak |  |
| `ai762010` | 上位：素材屋 | upper logic: materials seller |  |  |
| `ai763000` | 下位：呪術屋 | lower logic: pyromancy seller | Rosabeth of Melfia |  |
| `ai763010` | 上位：呪術屋 | upper logic: pyromancy seller |  |  |
| `ai764000` | 下位：鍛治屋 | lower logic: blacksmith [sic: 治 for 冶] | Blacksmith Lenigrast | Uses the wrong kanji (治 for 冶); the upper script spells it 鍛冶屋. |
| `ai764010` | 上位：鍛冶屋 | upper logic: blacksmith |  |  |
| `ai764300` | 下位：ＥＸ鍛冶屋 | lower logic: 'EX' blacksmith | Steady Hand McDuff |  |
| `ai764310` | 上位：ＥＸ鍛冶屋 | upper logic: 'EX' blacksmith |  |  |
| `ai766000` | 下位：魔術屋 | lower logic: sorcery seller | Carhillion of the Fold |  |
| `ai766010` | 上位：魔術屋 | upper logic: sorcery seller |  |  |
| `ai768000` | 下位：スペルマニア | lower logic: spell-mania | Straid of Olaphis |  |
| `ai768010` | 上位：スペルマニア | upper logic: spell-mania |  |  |
| `ai769000` | 下位：奇跡の人 | lower logic: miracle person | Licia of Lindeldt |  |
| `ai769010` | 上位：奇跡の人 | upper logic: miracle person |  |  |
| `ai770000` | 下位：闇術屋 | lower logic: dark-arts seller | Felkin the Outcast |  |
| `ai770010` | 上位：闇術屋 | upper logic: dark-arts seller |  |  |
| `ai771000` | 下位：スペルマスター | lower logic: spell master | Royal Sorcerer Navlaan |  |
| `ai771010` | 上位：スペルマスター | upper logic: spell master | Royal Sorcerer Navlaan (Invader) - Forest of Fallen Giants |  |
| `ai772000` | 下位：珍品屋 | lower logic: curio seller | Magerold of Lanafir |  |
| `ai772010` | 上位：珍品屋 | upper logic: curio seller |  |  |
| `ai776000` | 下位：上位武器屋 | lower logic: high-grade weapon seller | Weaponsmith Ornifex - Brightstone Cove Tseldora | ⚑ no enemy row in the Network Test |
| `ai776010` | 上位：上位武器屋 | upper logic: high-grade weapon seller |  | ⚑ no enemy row in the Network Test |
| `ai777000` | 下位：猫 | lower logic: cat |  |  |
| `ai777010` | 上位：猫 | upper logic: cat |  |  |
| `ai784000` | 下位：免罪の教戒師 | lower logic: chaplain of absolution | Cromwell the Pardoner |  |
| `ai784010` | 上位：免罪の教戒師 | upper logic: chaplain of absolution |  |  |
| `ai785000` | 下位：守護の騎士 | lower logic: guardian knight | Blue Sentinel Targray |  |
| `ai785010` | 上位：守護の騎士 | upper logic: guardian knight |  |  |
| `ai786000` | 下位：キラキラ娘 | lower logic: sparkling girl |  | Retail's Dyna & Tillo (ドネ＆ティロ), the pair in Things Betwixt who swap items: NPC name id 7860 reads キラキラ娘 in the Network Test's npcMenu and Dyna & Tillo in retail's, and retail's m10_02 talk script still names her states キラキラ娘 (sparkling-girl-dyna-tillo.md). |
| `ai786010` | 上位：キラキラ娘 | upper logic: sparkling girl |  | Upper script of the same NPC (see ai786000). |

## Method

Labels: the first `ai…_name` string in each `ezstate\ai*.esd` (UTF-16-BE), read from the Network Test (`DS2_GAME=nt`) and from retail v1.00 (`vanilla`) archives. Placed ids: `GeneratorParam_<map>` -> `GeneratorRegistParam_<map>` (`LogicParamID`, `DefaultLogicParamID`) over every map in the Network Test's GameData (`bosses_pc.generators`). Enemy-row check: any `EnemyParam` row in `regulation\EnemyParam.param` whose id starts with the script's first four digits. A generator is the only placement path this checks, so an enemy spawned some other way (event script) would show here as unplaced.
