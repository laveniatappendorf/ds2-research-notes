# Najka's damage cuts: the tail grab (x0.2) and the spell casts (x0.5) (2026-11-02)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

The author's question: on vanilla PC, a +4 Claymore R2 that normally did about 300 to Scorpioness Najka sometimes did only 65.
That happened after she stabbed her tails into the ground. Is that damage cut in the data?

**Answer: yes.** The tail stab is her **grab**, action 13. GrabParam 150300510 has the developer name
マンスコーピオン♀ 行動13_テイル投げ, "female man-scorpion, action 13: tail throw" (1.02 regulation names).
While a grab animation runs its "bind" window, the grabber takes **x0.2** damage.
- 300 x 0.2 = 60, and about 325 x 0.2 = 65.
- It is a fifth of whatever the hit would have done, not a cap.
- The window starts just before the tails hit the ground and lasts while they are stuck there. It applies whether or
  not the grab catches anyone.

She has a second, milder cut: **x0.5** on her whole body during her two magic-spear casts.

## The x0.2 grab state

| link | where | what |
|---|---|---|
| event | `c5030.nmb` (morpheme network), track `AttackMasterBind` | value **170** on `a65_13_1010` "Attack_13_1" from 0.267 for 0.467 of the clip, and on `a65_13_1020` "Attack_13_2" for the whole clip |
| timing | TAE `c503000_pl.tae` 65131010 (5.0 s) | 0.267-0.733 of the clip = about **1.33-3.67 s** (times read as clip fractions, as morpheme stores them). Ultra armour 110600 1.37-1.93 s, the tail hit 2200 at 1.60-1.73 s, turning locked 1.57-3.80 s |
| follow-up | 65131020 (4.67 s), the throw | the whole clip; second hit 2201 at 3.5 s; behaviour EnemyBehaviorSecondParam 503000 group 4 (damage 150300510 / 150300520, 20 s cooldown) |
| state | `FUN_140368e70` (DS2SPC) | event-track handler: case 0xAA (170) sets byte `[chr+0xB8]+0x28D` on entry and clears it on exit (the death / critical state entry `FUN_14032C530` sets it too) |
| damage | `FUN_1401373B0` (damage ctrl vtable slot 13) | if the hit character's +0x28D is set, damage x **EnemyCommonParam +0x26C** (`[chr+0x40]` is the EnemyCommonParam row itself) |
| value | EnemyCommonParam +0x26C | **0.2 on all 225 rows** (vanilla PC 1.12). SmithBox / DSMapStudio call it `crit_damage_mult`; "bind-state damage rate" fits better |

The multiplier goes into attack block +0x78. `FUN_140138D50` (slot 32) multiplies that **after** defence is subtracted,
so it is a straight x0.2 on the final number.

The same event is on **65 character networks** (PC Scholar scan; the c0001 dummy left out). They are always the
`Attack_NN_1` / `Attack_NN_2` grab pairs: the Pursuer, The Rotten, Demon of Song, Old Dragonslayer, the Ancient
Dragon, the man-scorpions, many regular enemies and several NPC phantoms (`tools\nmb_events.py scan 170`).

## The x0.5 spell casts (TAE event 111000 = per-part damage rate)

- Event 111000 (f32 rate, u8 part 0-4) writes `[chr+0xB8]+0x770 + 4 x part`. The game resets those five slots to -1
  each frame (`FUN_14031C460`).
- `FUN_140136F80` (slot 11) builds the hit character's rate table:
  - slot [0] is 0.0;
  - slots [1..5] are EnemyCommonParam +0x258..+0x268 (`Unk117`, `Unk118`, `Unk119`, `headshot_damage_mult`, `Unk120`);
  - a TAE override replaces any of them while it is 0 or more;
  - the hit's damage part then picks one entry.
- The table is only used when EnemyCommonParam +0x254 (`Unk116`) is 1, or while the counter at +0x60C (TAE 111100) is held.
- The result goes into the same attack block +0x78 multiplier as the bind state, so it also applies after defence.
- Najka: flag 1, ladder 0.5 / 0.75 / 1.0 / 1.5 / 2.0, but **all 65 of her hit capsules are damage part 3 (x1.0)**. She
  has no weak spot.
  - The part comes from `model\chr\c5030.bnd` -> `c5030_h.hkxpwv` (`tools\hitparts.py`).
  - Check: Demon of Song's shell is part 1 (x0.0) and its `Inside_*` / `InsideFace` capsules are part 4 (x0.0). Its attack
    animations raise TAE part 3 (= hit part 4) to x1.0, so the face can only be hurt while it is open.
- Najka's 111000 events are x0.5 on parts 0-4, so her whole body:
  - **65011050** = action 1, 行動1_魔術の槍 "magic spear" (bullet 150300010), the whole 3.33 s;
  - **65021050** = action 2, 拡散する魔術の槍 "spreading magic spears" (bullet 150300110), the whole 4.33 s;
  - 75017100 / 75017101: buried, and the first 0.17 s of emerging; she is invincible from 0.17 to 3.67 s.
- The same pattern (x0.4-0.6 over a whole special animation) is on Smelter Demon's sword-in-the-ground buff (c3050
  65011050 / 65021050, x0.4) and c3330 / c3240 / c6750 / c1320 animations (`tools\tae_pc32.py <chr> find 111000`).

## Najka's other numbers (vanilla PC 1.12)

EnemyParam 503000:
- HP 5,910;
- physical defence 477;
- magic 50, lightning 30, fire 15, dark 30.

EnemyCommonParam 503000:
- absorption: slash 1.0, **thrust 1.35**, strike 1.0;
- bullet absorption 0.85.

Neither changes during the moves above.

## Limits

- The code was read in the **PC Scholar** exe (DS2SPC). The vanilla PC 32-bit exe was not disassembled.
- The vanilla PC data does match:
  - its `c5030.nmb` has the same two event-170 records (byte search: same start, duration and value);
  - its `c503000_pl.tae` 111000 events are the same;
  - its EnemyCommonParam rows are the same.
- Not tested live. A test: hit her during the tail stab and during a magic-spear cast with the same attack. Expect a
  fifth and a half of the usual damage.
- Morpheme event times are read as clip fractions. The fit with the TAE (window opening 0.03 s before ultra armour)
  supports it.

Tools: `tools\tae_pc32.py` (vanilla PC 32-bit TAE), `tools\hitparts.py` (hit-capsule damage parts),
`tools\nmb_events.py` (morpheme event tracks, PC Scholar networks). Ghidra: `DS2 Scholar PC\Ghidra Project\script.cmd
FuncAt.java` / `FindOffsetUses.java` (0x1B198 = 111000, 0x770, 0x28D, 0x26C).
