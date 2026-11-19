# Seven community claims checked (2026-11-02)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

The author asked whether seven commonly repeated DS2 claims hold up. Checked against PC Scholar (exe 1.0.3.0, regulation
2.02) params, SpEffect*.emevd scripts, the player TAE and the exe code (headless Ghidra `script.cmd` with DecompRange /
FuncAt, plus capstone scans). Nothing was tested live. Verdicts: 1 true but not specific to guard breaks, 2 true,
3 mostly false, 4 plausible / not traced, 5 true, 6 plausible (data fits the known explanation), 7 effect real,
footstep link unproven.

## 1. Guard break "disables poise" (Gank Squad's Varg stunlocked)

The code explains the stunlock, but it is how every poise break works, not something special to guard breaks.

- ChrPoiseCtrl (vtable 0x1410bfc58): current poise at chr+0x218, floor +0x21c, max +0x220. Max = EnemyParam
  `stat_poise` (+0x118) + a virtual equipment term + two state terms (FUN_140145a40).
- Regen, FUN_140145ba0, runs every frame:
  - While byte [chr+0xb8]+0x7e6 is set, poise does not regenerate at all.
  - When that byte clears and poise is <= 0, poise snaps straight back to max.
  - Otherwise poise rises by EnemyParam `stat_poise_regen` (+0x11c) x dt. The +0x124 rate is used instead inside
    TAE event 111200 windows (counter +0x610), and poise refills to max when such a window opens or closes.
- The +0x7e6 byte means "in a hit reaction":
  - FUN_14013b4d0 sets it when a damage reaction starts.
  - TAE event 250 (0xfa, in FUN_140369890) holds it during damage animations.
  - FUN_14031c530 clears it every frame.
- So once poise is broken, it stays broken for the whole stagger. Every hit that lands before the target gets out of
  the stagger finds poise at 0 and staggers again. Leave a gap and poise comes back full at once.
- Gank Squad rows: Varg 863000 poise 220, Cerah 862000 66, Graverobber 864000 99. All three regen 0.33. Most enemies
  have poise 0 and regen 1.47.
- Not traced: whether the guard-break reaction itself zeroes poise. FUN_140134eb0 deals a full-max poise hit when hit
  record byte +0xa6 is set (and +0x640 is non-zero), which is a candidate, but it isn't identified.

## 2. Left-hand attacks cost more stamina and deal less damage: TRUE for nearly every weapon

- Stamina: WeaponParam `weapon_stamina_cost_id` points to a WeaponStaminaCostParam row. The left_1h_* multipliers are
  x1.08 the right_1h_* ones on every row that weapons use, except row 30 ("special strong attack set": Smelter
  Sword, Drakewing UGS), which is x1.0.
  - That is 308 of 310 named base weapons.
  - The field names come from SmithBox's community defs, but the ratio is consistent across every attack kind.
- Damage: WeaponTypeParam multipliers are right 1 / left 0.9 on 263 weapons (97 types). The same applies to the
  stamina damage dealt to a blocker.
  - 28 weapons are 1 / 1 (catalysts, torch, Majestic GS).
  - Bows are 0.9 / 0.9 and Avelyn is 0.865 / 0.865, so they have no hand difference.
  - Orma's Greatshield is reversed: right 0.9 / left 1.

## 3. Dried Root (claimed: 10 min, ~3000 HP, only heal that keeps going at max HP, cheese by stacking): MOSTLY FALSE

- SpEffectActiveItem 60036000 `100050:0 (1, 850, 1000, 500.0, 0)` = 1000 HP over 500 s (8 min 20 s, 2 HP/s).
  It is the same in vanilla PC 1.12, and absent in the 2014-03-31 build. Fextralife has the same figure.
- Other healing items, as (HP, s, last byte):

  | Item | HP | s | last byte |
  |---|---|---|---|
  | Lifegem | 500 | 11.1 | 1 |
  | Radiant Lifegem | 850 | 18.8 | 1 |
  | Old Radiant Lifegem | 1300 | 28.8 | 1 |
  | Estus 60155000 | 550 | 1.8 | 1 |
  | Elizabeth Mushroom | 2000 | 44.4 | 0 |
  | Dried Root | 1000 | 500 | 0 |

- The "3000 HP" figure matches only abnormal-state event 901000 (3000 HP over 1 s), which is not the root.
- The last byte splits the items into lifegems / Estus = 1 and Mushroom / Root = 0. Its meaning (stop at full HP?
  cancel on hit?) was not found: the heal code sits in SpEffectEach_Heal / SpEffectGroup_Common<Heal,4>
  (vtables 0x1410ce900 / 0x1410cec68), and the tick itself wasn't reached. Even if it does mean "keeps ticking at
  full HP", the Mushroom shares it, so "the only one" is wrong.
- Stacking: Fextralife says the root doesn't stack with itself. The heal group keeps per-type counts (ItemParam
  `item_sp_effect_type`: lifegems 1, Mushroom / Root 2). Even stacked, 2 HP/s can't carry a hard fight.

## 4. Fire-infused bow + plain arrows light oil: PLAUSIBLE, NOT TRACED

- The "oil" in DS2 is tar:
  - MapObjectParam 9113 floor tar, 9123 scorched tar, 9250 scattered tar, 3001 tar pot, 3007 oil pot.
  - System bullet / damage 200006010 "Tar Flame" trap.
  - GroundMaterial 100-109 oil, including the "Butcher room" oil-flame trap 200014050.
- Arrow hits carry the bow's elemental AR (Radai's ranged formula, eAR term), so a fire bow's plain arrows deal fire
  damage.
- The tar ignition test is in code and was not found. Community reports (Steam) say fire bows set off flame barrels
  without fire arrows.
- A live test would settle it.

## 5. Higher frame rates wore weapons out faster, fixed May 2015: TRUE

Kotaku (7 May 2015) reported the PC Scholar patch note: "Fixed issue whereby weapon durability was decreased
drastically when used on enemy corpses, friendly characters, etc." The wear rate had been tied to frame rate, so 60 fps
wore weapons out faster. Not checked in code.

## 6. Ripostes deal more damage at 60 fps than at 30 fps: PLAUSIBLE (data fits the explanation)

- [Radai's guide](damage-formula-radai.md) (findings\damage-formula-radai.txt): every critical TAE has two damage events. Above 30 fps they can
  overlap for a frame, giving 3 ticks instead of 2 (his 60 fps table: ripostes 3 ticks for daggers / straight swords /
  UGS / fists, 2 for axes / spears).
- Player TAE c000100_pl.tae has 38 critical animations (xx0103xx, 6 weapon families). Each has two type-2201 events
  whose windows touch exactly, for example 20010300 at 0.8333-0.9333 and 0.9333-1.0333 s. That is the setup the
  overlap needs.
  - Type 2201 = damage tick is my reading, not proven.
  - The tick counts weren't measured live.

## 7. Wet footsteps show the lightning-down / fire-up effect: EFFECT REAL, FOOTSTEP LINK NOT PROVEN

- ChrWetCtrl (vtable 0x1410bfcd8, update thunk_FUN_1401476b0):
  - It reads ChrWetParam rows 0-6 (0 water, 1 poison, 2 oil).
  - Every frame it passes the wetness level (0-1) to FUN_140145fe0, which applies abnormal-state effects: water
    900800, oil 900700, poison 900900, rows 3-6 902000-902300.
- 900800 (water) has 100090:4 fire 30 and 100090:5 lightning 15, both 300 s.
  - 100090:4 is the command the burrs and quartz rings use for resistance up; Small Orange Burr is fire 15.
  - 100090:5 is the clutch rings' defence-down command.
  - So the effect is fire resistance up, lightning down.
  - 902300 is fire 10 / lightning 10 over 5 s.
- Oil-wet 900700 has no defence change in its script, so "oil makes you weak to fire" is not in the data.
- Footstep visuals are a separate system:
  - Ground-material splash sfx: GroundMaterialParam Unk09 -> FootEffectWaterSfxParam 10 / 30 / 40 / 50.
  - Footprint decals: DecalParam 900003 "Footprint (Water) _Test".
- Whether the footprints end exactly when 900800 ends is not shown. The script's 300 s timer hints the effect may
  outlast the footprints, unless the controller removes it as wetness reaches 0 (the apply call
  thunk_FUN_141afbc6d is in protected code).
- Live test: walk out of water, then check 900800 against the footprints.
