# The tearstone rings: the threshold, the multiplier and what it lands on (PC Scholar 1.0.3.0, 2026-11-20)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

Question: the Red and Blue Tearstone Rings are known to work differently from their Dark Souls 1 namesakes. What is
the actual calculation?

**Both rings hold a plain multiplier that is live while the ring is worn, and a threshold byte of 30 that gates when
the game reads it.** The Red sets rate slot 27 to x1.2, the Blue sets rate slot 28 to x0.65, and each writes 30 into
its own byte on the character's status object. On the Blue the gate is in the clear: before using the x0.65 the game
reads that byte, compares it against a number taken from the wearer's own object, and applies the multiplier only
when 30 is greater than or equal to it - a current-HP percentage check, done per hit, not a timer.

The Blue's x0.65 multiplies the **finished damage, after defence has been subtracted, across all five damage types**.
It is not a physical-only cut and it is not a defence boost.

## 1. The two items

| | Red Tearstone Ring | Blue Tearstone Ring |
|---|---|---|
| `ItemParam` / `RingParam` row | 40260000 | 40280000 |
| price | 5,700 (sells for 570) | 4,000 (sells for 400) |
| weight / durability / repair | 1.5 / 110 / 6,600 | 1.0 / 110 / 6,600 |
| `sort_id` | 260 | 280 |

Both are ordinary rings: `speffect_context` 25, `item_category_type` 7, `item_usage_id` 1200, like every other ring
in the game. Nothing in the params marks them as conditional.

Their `DetailedExplanation` says only "Reacts when the wearer is in danger, temporarily increasing its wearer's
attack power" / "...defense". No number, and no threshold.

## 2. The four instructions

`SpEffectRing.emevd` event 40260000 (Red):

| instruction | meaning |
|---|---|
| `100130:2 (60, 1, 30.0)` | status icon 60, timed, 30 s |
| `100040:1 (931, ..., 30.0)` | the red aura, 30 s |
| `1000:0 (4, 0.0, 30)` | byte slot **4** += **30** |
| `1000:1 (27, 0.0, 1.2)` | rate slot **27** x1.2, duration 0 = while worn |

Event 40280000 (Blue) is the same shape: icon 60 timed 30 s, SFX 932, `1000:0 (5, 0.0, 30)`, `1000:1 (28, 0.0, 0.65)`.

Three things stand out from a scan of all eleven `SpEffect*.emevd` files in the regulation:

- `1000:0` byte slots **4 and 5 are used once each in the whole game**, by these two rings and nothing else.
- `1000:1` rate slots **27 and 28 are used once each**, likewise. Compare slot 59, which is shared by the Ring of
  Steel Protection, the Brightbug, Sacred Oath and Numbness - a shared slot carries no condition.
- `100130:2` is used 146 times and **every other use passes `(icon, 0, 0)`**. These two rings are the only timed
  use of it in the game, and icon 60 is theirs alone.

So each ring gets its own pair of slots, which is how the HP condition can be attached to them at all.

## 3. The threshold byte

`1000:0 (index, seconds, amount)` is `SpEffectActionImpl_ChrStatus_Value`. The apply path, at 0x14021a979, reads the
index from byte 0 of the argument block and the amount from byte 8 and does

    status[0x305 + index] += amount

    0x14021a979   mov    rdx, [rbx + 0x40]          ; the effect's arguments
    0x14021a97d   movzx  ecx, byte ptr [rdx]        ; index
    0x14021a980   movzx  eax, byte ptr [rdx + 8]    ; amount
    0x14021a984   add    byte ptr [rcx + r8 + 0x305], al

(The same byte array has a second, interpolated writer at 0x14021a8d1 - the `1000:19` form, which lerps the amount
between two bounds. Neither ring uses it.)

So the array base is `status + 0x305`, and:

| ring | `1000:0` index | byte | value |
|---|---|---|---|
| Red Tearstone | 4 | `status + 0x309` | 30 |
| Blue Tearstone | 5 | `status + 0x30a` | 30 |

For context, index 0 is the Ring of Giants' +10 / +20 / +30, at `status + 0x305`, read at 0x140145a79; index 12 and 13
are the Ring of Knowledge's and Ring of Prayer's +5 Intelligence and Faith. The index space is shared with the gear
stat bonuses but the tearstones' two entries are theirs alone.

## 4. The gate (Blue, in the clear)

`status + 0x30a` has exactly **one** read in the whole image, and `status + 0x42c` - rate slot 28, at
`0x3bc + 4 x 28` - has exactly **one** multiply. They are nine instructions apart, inside damage-ctrl vtable slot 13
(`FUN_1401373b0`), the function that folds every per-hit multiplier into one number:

    0x1401378c1   mov    rcx, [rsi + 8]             ; the target: the ring's wearer
    0x1401378e1   mov    rbx, [rsp + 0x50]          ; the target's status object
    0x1401378e6   mov    rcx, [rcx + 0x228]
    0x1401378ed   mulss  xmm6, [rbx + 0x424]        ; slot 26 on the target (unrelated)
    0x1401378f5   test   rcx, rcx
    0x1401378f8   je     0x14013791a                ; nothing there -> skip
    0x1401378fa   call   0x14016a440                ; -> xmm0
    0x1401378ff   movsx  eax, byte ptr [rbx + 0x30a]   ; the threshold byte, 30
    0x140137906   movd   xmm1, eax
    0x14013790a   cvtdq2ps xmm1, xmm1
    0x14013790d   comiss xmm1, xmm0
    0x140137910   jb     0x14013791a               ; threshold < value -> skip
    0x140137912   mulss  xmm6, [rbx + 0x42c]       ; slot 28: the Blue Tearstone's x0.65

The condition is **threshold >= whatever `FUN_14016a440` returns**, so x0.65 applies at 30 and below, inclusive.

`FUN_14016a440` is a thunk into the protected region (`jmp 0x141b9cc6c`), so what it computes is not readable. But
it is called on the wearer's own object, its result is compared against a byte holding **30**, and the test is "30 or
less". On a 0.0-1.0 fraction that test would pass always and the ring would be unconditional, so the value is on a
**0-100 scale**: the wearer's current HP as a percentage. That is an inference from the scale, not from the function.

## 5. What the x0.65 lands on

Slot 28's multiply is one factor of `xmm6`, the product that `FUN_1401373b0` stores at `[rdi + 0x78]` on the hit
record. Per [the Brightbug](brightbug.md), slot 32 (`FUN_140138d50`) then builds the damage: for each of the five
damage types it subtracts the target's defence, floors, divides by the cut and applies absorption into a running sum,
and only then, at 0x1401390b1, multiplies that sum by the slot 13 product.

Two consequences:

- **It covers every damage type**, physical and elemental alike, because the multiply lands on the sum of the five.
  The community pages describe the Blue Tearstone as a cut to physical damage taken; the code applies it to all of
  it. (The same correction applies to slot 59 and the Ring of Steel Protection, which the Brightbug note already
  found.)
- **Because it lands after the subtraction, x0.65 is 35 % off whatever was already getting through** - no more, but
  also no less, and independent of the size of the hit. That is the opposite of a defence-stat boost, whose value
  swings with the incoming attack.

## 6. The Red, as far as the protection allows

Rate slot 27 is `status + 0x428`, and it too has exactly one multiply:

    0x140027533   mulss  xmm6, [rbx + 0x428]
    0x14002753b   jmp    0x14039255b

- That is a **relocated basic block** - the obfuscator has moved it out of its function and left a jump at the end.
  Which function it belongs to, and so whether `rbx` is the attacker or the target there, is **not traced**.
- The threshold byte `status + 0x309` has one signed read in the image, at **0x141b88814**, deep in the protected
  region - and it is the same four instructions as the Blue's gate:

      0x141b88814   movsx  eax, byte ptr [rbx + 0x309]
      0x141b8881b   movd   xmm1, eax
      0x141b8881f   cvtdq2ps xmm1, xmm1
      0x141b88822   comiss xmm1, xmm0

  The conditional branch after it is dispatched through the obfuscator's jump chain rather than sitting next to the
  comparison, so the direction of the test is not readable there. The shape is identical, which makes the same
  reading - HP percentage at or below 30 - very likely, but it is not proven the way the Blue's is.

So: the Red's trigger is 30 % on the same evidence as the Blue's (its own byte, written with 30, read with the same
instruction sequence). **Where its x1.2 sits in the damage chain is still open.** It is a different function from
the Blue's, so it does not follow that it multiplies the finished damage; it could be earlier.

## 7. It is not a 30-second buff

Both `1000:1` lines carry **duration 0**, which on a worn item means "while worn". So the multiplier sits in its slot
the whole time the ring is on, and the gate above decides, per hit, whether it is read. Nothing in the readable code
starts a 30-second window.

The 30 s appears twice, in `100130:2` (the status icon, with its timed flag set) and in `100040:1` (the aura SFX).
The straightforward reading is that those are the **display**: the icon and the glow run for 30 s once the ring fires,
while the multiplier itself stays gated on live HP.

**In play this is settled**: the rings keep working for as long as you stay under the threshold, with no 30-second
cut-off, and a player can see that without a stopwatch. So the per-hit gate above is the behaviour, and nothing more
needs testing on that point.

What that does **not** settle is whether something runs on a 30-second cycle underneath - the effect being re-applied,
refreshed or re-armed every 30 s, which a player would never notice because the result is the same either way. These
two rings are the only timed use of `100130:2` in the game, so the 30 s is doing *something*. Reading the rate slot
over a few minutes while the wearer sits below the threshold would answer it: a slot that is re-applied without
expiring would climb (1.2, 1.44, 1.728...), one that expires and re-fires would toggle, and a flat 1.2 throughout
would put the 30 s entirely in the icon and the aura. That read is in `tearstone-red-chain-test-plan.md`, step 0.

## 8. Every retail build has the same numbers

The four instructions and all four values are **byte-identical** in:

| build | |
|---|---|
| PS3 v1.00, 1.01, 1.02, 1.03 | same |
| PS3 1.08, 1.09, 1.10 | same |
| PS3 Scholar | same |
| PC Scholar 1.0.3.0 | same |

(PS3 1.04-1.07 are unobtainable.) Nothing about these rings was ever rebalanced.

The **Network Test** is different. Its `SpEffectRing.emevd` holds only 8 ring events, and the tearstones are not
finished:

- **40260000** exists with three instructions in the older command banks: `130:23 (60)`, `40:3 (0.7, 1.0)`, `90:9 ()`.
  Icon 60 is already the icon retail uses. What `40:3` meant in that bank is not traced.
- **40280000 does not exist at all** - the Blue Tearstone has no effect in that build.

This matches the earlier reading that the ring is a no-op in the demo
([Armour, rings and spells: NT vs retail](../network-test/armour-rings-spells-nt-vs-retail.md)).

## 9. Against Dark Souls 1

| | Dark Souls 1 | Dark Souls 2 |
|---|---|---|
| threshold | 20 % HP | **30 % HP** |
| Red | attack x1.5 | **x1.2** |
| Blue | defence x1.5 | **damage taken x0.65** |
| what Blue changes | the defence stat, before the subtraction | **the finished damage, after it** |

The second row of that table matters more than the numbers. Dark Souls 1's Blue Tearstone raised a defence *stat*,
so its real worth swung with the size of the hit - close to useless against a big one. Dark Souls 2's is a flat
multiplier on the damage that was already getting through, so 35 % off is 35 % off whatever hits you. Against heavy
hits the smaller-looking number is the better ring.

The DS1 values here are from the community's own tables, not from our own read of DS1 data.

## 10. Open

- **Where the Red's x1.2 lands in the chain**, and which side `rbx` is at 0x140027533 - on the attacker, or on the
  victim. The protected region hides the function the block belongs to. This is the main one, and it has a run
  sheet: `tearstone-red-chain-test-plan.md`.
- **Whether the Red covers all damage types** or only physical, which follows from the above.
- **Whether anything runs on a 30-second cycle under the hood** (section 7). Not visible in play either way.
- **What `FUN_14016a440` returns.** The 0-100 reading is an inference from the threshold byte's scale.
- Whether the comparison the Red's gate performs is the same direction as the Blue's.
- The byte array at `status + 0x305` is only partly mapped. Of its 20 used indexes, 6-14 are the attribute bonuses
  (Vigor to Adaptability) and 20 is attunement slots (Southern Ritual Band, Black Witch Hat); 1, 3, 17, 18, 19, 21
  and 22 have users but no identified meaning.

## Related pages

- [Brightbug: what it multiplies, and where in the chain](brightbug.md): the same damage-ctrl function, slot by slot,
  and the proof that the slot 13 product applies after defence. It is where the Blue Tearstone's gate was first
  spotted.
- [Radai's damage formula guide](damage-formula-radai.md): the formula these multipliers sit at the end of.
- [Najka's damage cuts](najka-damage-cuts.md): another multiplier in the same product.
- [Staff of Amana: the hollowing penalty](staff-of-amana-hollowing.md): the `1000:15` sibling command, a rate that
  varies with a counter.
- [Armour, rings and spells: NT vs retail](../network-test/armour-rings-spells-nt-vs-retail.md): what the Network
  Test's ring list had and what it did with it.
