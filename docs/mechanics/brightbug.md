# Brightbug: what it multiplies, and where in the chain (PC Scholar 1.0.3.0, 2026-11-20)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

Question: the Brightbug raises attack and defence. Is that the finished attack rating, or something earlier in the
damage calculation?

**Neither. It never touches attack rating.** It sets two multipliers - x1.2 on the damage the user deals and x0.8 on
the damage the user takes, both for 120 seconds - and the game applies both to the finished damage number, after the
target's defence has already been subtracted. The stats screen does not move, and both multipliers are gated on who
is hitting whom.

## 1. The item

| field | value |
|---|---|
| `ItemParam` row | 60239000, `speffect_id` 60239000, `icon_id` 60239000, `sort_id` 250 |
| price | 6,000 (sells for 600), stack of 99 |
| `item_usage_id` | 1615 (see [4](#4-who-it-works-against)) |

Its `DetailedExplanation` ends with the line "Has no effect in other worlds or for phantoms."

## 2. The two multipliers

`SpEffectActiveItem.emevd` event 60239000 is four instructions: two `100040:1` visual lines (ids 969 and 970) and

| instruction | meaning |
|---|---|
| `1000:1 (58, 120.0, 1.2)` | rate slot **58** x1.2 for 120 s |
| `1000:1 (59, 120.0, 0.8)` | rate slot **59** x0.8 for 120 s |

So this is two separate quantities, not one shared attack-and-defence stat.

`1000:1 (slot, seconds, rate)` builds `SpEffectActionImpl_ChrStatus_Rate` (vtable 0x1410cd328; the slot is a byte at
the effect's +0x40, the rate a float at +0x44). Applying it (`FUN_14021bd30`) does

    [status + 0x3bc + 4 * slot] *= rate

and letting it run out (`FUN_14021bcb0`) multiplies the same float by 1 / rate. So every slot is one float on the
character's status object (`chr + 0xb8`), each source multiplies into it, and nothing overwrites anything: slot 58
sits at +0x4a4 and slot 59 at +0x4a8.

Who else uses these two slots (every `SpEffect*.emevd` in the regulation):

| slot | users |
|---|---|
| 58 | the Brightbug alone |
| 59 | Brightbug x0.8, Ring of Steel Protection x0.9 / x0.9 / x0.875, Sacred Oath x0.9 (all five durations), Numbness x0.8 |

Numbness carries slot 26 x0.85 **and** slot 59 x0.8, which is the clearest sign that the two are not the same thing:
26 is read with no gate, 59 only in the case in [4](#4-who-it-works-against) below.

## 3. Where the game reads them

A disassembly pass over both `.text` sections finds one read of each slot in the damage path (the other instructions
at those displacements belong to unrelated structures). Both are in **damage-ctrl vtable slot 13,
`FUN_1401373b0`** - the function that folds every per-hit multiplier into one number:

| address | instruction | what |
|---|---|---|
| 0x1401378d9 | `mulss xmm6, [rax+0x26c]` | bind / grab state, `EnemyCommonParam` +0x26c (see [Najka's damage cuts](najka-damage-cuts.md)) |
| 0x1401378ed | `mulss xmm6, [rbx+0x424]` | slot 26 on the **target** (Iron Flesh, enemy guard states) |
| 0x140137912 | `mulss xmm6, [rbx+0x42c]` | slot 28 on the target, behind an HP check: the Blue Tearstone Ring |
| 0x140137977 | `mulss xmm6, [rdx+0x4a4]` | **slot 58 on the attacker** - the Brightbug's x1.2 |
| 0x140137994 | `mulss xmm6, [rbx+0x4a8]` | **slot 59 on the target** - the Brightbug's x0.8 |
| 0x1401379c6 | `movss [rdi+0x78], xmm0` | the product, stored on the hit record |

**Slot 32, `FUN_140138d50`** then turns that into HP damage. Per damage type 0-4 (physical, magic, lightning, fire,
dark) it takes the attack value, subtracts the target's defence for that type (0x140138e71), floors the result,
divides by the cut, applies the per-type absorption, and adds it to a running sum. Only then:

    0x1401390b1   mulss xmm11, [rsi+0x78]      <- the slot 13 product, Brightbug included

followed by the counter and damage-part multipliers, the [distance cut](distance-damage-cut-pc.md), the one unscaled
term, and truncation to an integer.

So in the [damage formula](damage-formula-radai.md)'s terms the Brightbug is not `pMod`, not `hMod`, and not the
`bFlat` / `bMod` of a weapon buff. It sits at the end with `Dm` and `Cm`, on `(PhysDmg + ElemDmg)`.

Two things follow:

- **It covers every damage type at once**, physical and elemental, because the multiply lands on the sum of the five.
  The same is true of slot 59, so the x0.8 - and Ring of Steel Protection's x0.9 - cuts all five types, not just
  physical. (Earlier notes guessed "physical damage taken" for slot 59 from the ring's item text; the code says
  otherwise.)
- **x1.2 after the subtraction is worth less than x1.2 on attack rating would be**, since
  `1.2 x (AR - DEF)` is below `1.2 x AR - DEF`. Against a 400-defence target it is exactly 20 % of the damage that
  was already getting through, no more.

## 4. Who it works against

Both reads sit behind the same gate. The code asks for a category byte for the attacker and for the target
(`FUN_14014eca0` = `byte [chr+0x54]`), looks each up in a 14-entry, 5-byte table at 0x1410bfff0 and tests byte 3 of
the entry:

- slot 58 applies only when the **attacker's** byte 3 is set and the target's is not;
- slot 59 applies only when the **target's** byte 3 is set and the attacker's is not.

Byte 3 is set on categories 1 and 2 only. The "exactly one side" shape means the two multipliers do nothing when both
sides are of the same kind, so if categories 1 and 2 are the player characters the effect is PvE only: no help when
hitting another player, no protection from one. That reading also fits Numbness carrying an ungated slot 26 x0.85
beside the gated slot 59 x0.8 - weaker against players than against enemies. What `+0x54` holds is **not traced**,
so treat the PvE reading as inference, not fact. The same table is the one read by the
[Broken status](../network-test/broken-status.md) work.

The item's own restriction looks separate and is enforced at use time: `ItemUsageParam` row 1615 is
`usage 7, Unk02 93, item_no_trade 6`, where almost every other consumable is 252-254. The two bits the Brightbug
clears (0x20 and 0x80) are clear on exactly the items that are restricted away from other worlds - the Aged Feather,
the Darksign (both 88) and the Homeward Bone (92). Which bit is which is not traced.

## 5. It is not in the launch game

| build | row 60239000 |
|---|---|
| Network Test (US and JP) | absent |
| PS3 v1.00, 1.01, 1.02, 1.03; Japanese disc | absent |
| PC 2014-03-31, 2014-04-24, 1.02, 1.03, 2014-06-02 | absent |
| PS3 1.08, 1.09, 1.10 | present |
| PS3 Scholar, PC 1.12, PC Scholar | present |

So the Brightbug arrives with the DLC-era patches, first seen in **PS3 1.08** - the same patch as the
[seven look-only weapon copies](../builds/patch-1.08-weapon-copies.md). PS3 1.04-1.07 are unobtainable, so the exact patch could
be a little earlier. The slot numbers and both values are identical in every build that has it.

## 6. Open

- **Not seen live.** A test: note the damage of one attack on a fixed enemy, drink, repeat. The prediction is a flat
  x1.2 on the number, and no change at all in the stats screen.
- **What `byte [chr+0x54]` is**, which decides whether section 4 is really "PvE only".
- **The Red Tearstone Ring is a different slot (27) and a different consumer**: its read at 0x140027533 sits in the
  anti-tamper protected code, so where in the chain that one lands is still open. The same goes for slot 25, the
  general attack multiplier used by enemy rage buffs.

## Related pages

- [Radai's damage formula guide](damage-formula-radai.md): the formula these multipliers sit at the end of.
- [Distance damage cut](distance-damage-cut-pc.md): the rest of the same hit pipeline, slot by slot.
- [Najka's damage cuts](najka-damage-cuts.md): the bind-state multiplier that shares this spot, and the proof that
  the slot 13 product applies after defence.
- [Staff of Amana: the hollowing penalty](staff-of-amana-hollowing.md): the sibling command `1000:15`, a rate that
  varies with a counter instead of a fixed number.
- [Broken status](../network-test/broken-status.md): slots 25 and 26, and the same character-category table.
