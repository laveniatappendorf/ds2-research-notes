# "Break" (破壊, equipment destruction): a cut damage element (2026-10-26)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Found while checking the Network Test's infusion stones (`scaling-letters-and-infusion.md`). The Network Test has a
ninth damage / status type, **Break** (破壊, "equipment destruction"), that wears down the target's equipment
durability. v1.00 removes it from the menus and the infusion set but keeps a few leftovers. Static only: text and
params of the US / JP Network Test and v1.00; nothing tested in game.

## Network Test

Menu help (`iconHelp.fmg`, English and Japanese):

| id | label | text (Japanese, summarised) |
|---|---|---|
| 909027 | Break ATK (破壊攻撃力) | strength of the damage dealt to the opponent's equipment; **rises with Strength** |
| 909309 | Break ATK (weapon screen) | "Ability to reduce opponent's equipment durability" |
| 909035 | Break resist (装備破壊耐性) | resistance to attacks with the equipment-break attribute; rises with 耐久力 (Resistance) |
| 909321 | Break reduction (カット率 装備破壊) | share of break damage a shield cuts |
| 909336 / 909409 | Break resist (armour) | the higher, the slower equipment loses durability; total of the armour worn |

The stat that governed it, 耐久力 / **Resistance** (909012: "sets defence, also raises poison and bleed
resistance"), is the Network Test's DS1-style stat (Adaptability replaces it in retail).

Items:

| id | Network Test (Japanese) | rows | v1.00 |
|---|---|---|---|
| 61120000 | 溶解石 "dissolving stone": "gives equipment the Break attribute" (infusion) | ItemParam row | gone; the id is a **gap** between Bleed Stone 61110000 and Raw Stone 61130000 |
| 60300000 | 龍の紅液 "dragon's crimson fluid": "temporarily gives the right-hand weapon a break effect" (a resin) | text only | gone, but its **icon `IC_0060300000` survives** in every retail build: a red potion in a stoppered flask (the community wiki's "Unknown Serum 60300000", which it guesses was a stronger bleed buff; 2026-11-09) |
| 60220000 | 硫黄海月の丸薬 "sulphur jellyfish pills": "temporarily raises break resistance" | ItemParam row | **text and icon only**: Yellow Sea Troches, "Temporarily boosts equipment break resist" (Saint Elizabeth's medicines); no ItemParam row, so not obtainable. Its icon `IC_0060220000` (a grey case of pills) is in every retail build (found 2026-11-09; the community wiki's picture) |
| 60610000 | 紅液瓶 "crimson fluid bottle": "destroys equipment with acid solution" | ItemParam row | Corrosive Urn (強力な酸の瓶), "uses acid to destroy equipment" |
| 60900000 | 破壊の大矢: "greatarrow that deals Break damage" | ItemParam + ArrowParam | Destructive Greatarrow, "designed to destroy equipment with impact ... grind them down from afar" |

Also: the JP Network Test's player-message vocabulary has 破壊 "destroy" (4240) and 装備破壊 "equipment breakage"
(8090), and its Things Betwixt messages include 装備破壊に注意 ("beware of equipment breakage", m10_02 1025).

## v1.00

- **Menus:** every Break entry above is deleted. Where the Network Test lists `... Poison, Bleed, Break, Curse`
  (stat screen 909033-909036, shield cut rates 909319-909322, armour 909334-909337 / 909407-909410), v1.00 has
  `Poison, Bleed, -, Petrify`, with Curse moved to new ids 909411 / 909412. So the Break slot's neighbour became
  Petrify, and the damage-type lists kept nine entries.
- **ArrowParam** (bytes +0x50.. are (switch, amount) pairs: poison 0x50 / 0x51, bleed 0x52 / 0x53, then a third pair
  0x54 / 0x55; the community defs call these damage_poison / apply_poison ... damage_durability / Unk55, one off):

  | arrow | Network Test 0x54 / 0x55 | v1.00 0x54 / 0x55 |
  |---|---|---|
  | Wooden Greatarrow | 0 / 50 | 0 / 0 |
  | Destructive Greatarrow | 0 / 30 | **0 / 200** |
  | (Poison Arrow, poison pair) | 0 / 50 | 1 / 44 |
  | (Lacerating Arrow, bleed pair) | 0 / 50 | 1 / 44 |

  v1.00 raised the Destructive Greatarrow's break amount to 200, but where the poison and bleed arrows got their
  switch byte set to 1, the greatarrow's stays 0: possibly inert in retail (not tested).
- **Survivors in name:** Acid Surge (pyromancy: "eats away at their equipment"), Corrosive Urn, Destructive
  Greatarrow, Yellow Sea Troches (text only), message words "destroy" / "equipment breakage".
- FeWeaponThreshold row 8's developer name 耐久力影響閾値 = "Resistance influence threshold": the Network Test's
  Resistance stat, not Break; unused by the retail weapon screen.

## PC developer names (2026-11-09)

The pre-release PC params' developer row names identify two Break leftovers that are in the Network Test and every
retail build, unnamed: a Break **weapon buff glow** (ChrPhantomParam 25 "weapon enchant: weapon break", shield 57)
and Break as a **third status effect** between bleed and curse (PlayerDamageParam 9030 "weapon break" -> DamageCtrlParam
99987030 "generic – status ailment – weapon break"). Details: `upgrade-paths-pc-dev-names.md`.

## Retail's "Broken" status (2026-11-12)

Break was never fully removed. It's still ailment 2 in code and damage type 8 in the damage params. In the NT, a
full Break build-up applied SpEffect 900300 (damage row 9030, "weapon break") to everyone. From v1.00 on, two
character categories still get 900300, which retail data deleted. Everyone else, enemies included, gets 900301:
x0.95 damage dealt, x1.05 damage taken, 60 s (the wiki's "Broken"). Acid Surge (new in v1.00), the urn and the
greatarrow all deal Break damage, and the Ring of Resistance still raises Break resist (defence element 7). Details:
`broken-status.md`.

## Not established

- Whether the Network Test's nine-type damage block (WeaponReinforceParam) has Break in the slot retail calls
  petrify: no Network Test weapon fills that slot. The menu order says so; the params can't confirm it.
- The Network Test's armour resistances (ArmorReinforceParam, 180-byte rows in an older layout: eight values after
  the physical ones) were not decoded.
- Whether retail's Destructive Greatarrow damages equipment at all (live test: shoot an NPC / invader, watch
  durability; or trace the arrow's damage code).

## Update 2026-11-16: the Break-resistance knuckle ring

The Network Test's four knuckle rings (40100000-40130000: 解毒 / 止血 / 防護 / 阻呪の指甲) follow the status order of
its Ring of Resistance, whose text lists poison, bleeding, weapon breakage and curse (types 5-8). Retail's rings under
three of those ids resist types 5, 6 and 8. The third, 防護の指甲 "protective knuckle ring", sits in the type-7 slot,
and retail renamed it 封壊の指甲 (Bracing Knuckle Ring), which slows equipment wear. So it was most likely the
Break-resistance ring. By position and name only: the demo has no event for any knuckle ring. See
`unused-ring-icons.md`, "Ring order". Retail v1.00 text: 封壊の指甲 / "Slows equipment degradation" (SimpleExplanation 40120000). Added to the wiki-team doc's Break evidence table 2026-11-16.
