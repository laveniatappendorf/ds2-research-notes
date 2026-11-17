# Staff of Amana: the hollowing penalty (PC Scholar 1.0.3.0, 2026-11-15)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

**Formula** (code-confirmed): rate = 1.0 − 0.09 × hollow level, for hollow level 0 to 10; 0.1 at 10 and above.

| hollow level | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10+ |
|---|---|---|---|---|---|---|---|---|---|---|---|
| rate | 1.00 | 0.91 | 0.82 | 0.73 | 0.64 | 0.55 | 0.46 | 0.37 | 0.28 | 0.19 | 0.10 |

## Data
- ItemParam 3810000 `speffect_id` 3810000 → `SpEffectWeapon.emevd` event 3810000, one instruction **1000:15**,
  16 bytes: `01 00 0a 00 | 1.0f | 0.1f | 0.0f` = (counter 1, low 0, high 10, rate at low, rate at high, duration).
  It is the only staff with a SpEffect.
- The same command, mirrored (0.1 → 1.0 over hollow level 0-10), is on the Greatsword of the Forlorn 1997000, the
  Scythe of the Forlorn 3080000 and the Dark Pyromancy Flame 5410000. The Bewitched Alonne Sword's 1790020 uses it
  with low = high = 0 (so a flat x1.6, with duration 30 s); 5295010 uses counter 2, range 0-30, 1.0 → 0.0.
- Item text (DetailedExplanation 3810000): "Effect: spell damage modifier while human".

## Code (DS2SPC, headless on a copy; the MCP was locked by another session)
- RTTI: 1000:15 builds `SpEffectActionImpl_ChrStatus_AtkRate<SpEffectChrStatus_AtkRateByHollowCount>`
  (vtable 0x1410ce6f8, constructor FUN_14021ec20: duration = args +0xC). A sibling `<...AtkRateBySoul>` (vtable
  0x1410ce628, FUN_14021ed00) uses a 24-byte layout with int bounds; no SpEffect event uses it.
- Getter = vtable slot +0xB8, FUN_14021f1e0: only for query kind 8, and only when the effect's source slot (byte
  +0x2E: 0x1A / -4, 0x1B / -3, other) matches the asking hand flag (1 / 2 / 4) — likely right / left weapon / other,
  not checked; otherwise 1.0.
- Rate = FUN_14021f470: counter n from SpEffectOwner (vtable 0x1410af2b8): byte 1 → +0x18 = FUN_14023c7c0 =
  (signed char) [[[GameManagerImp 0x1416148F0]+0xD0]+0x490]+0x1AC = **hollow level** (the community CE table's
  "Hollow Lvl"); byte 2 → +0x20 FUN_14023c6a0 (calls FUN_14038b950, not identified); other → +0x10 = +0xEC (soul
  memory per the community table). Then: lo = hi or rate_lo = rate_hi → rate_lo; n < lo → rate_lo; n ≤ hi →
  rate_lo + (n − lo) / (hi − lo) × (rate_hi − rate_lo); n > hi → rate_hi.

## Open
- **What the rate multiplies is not traced**: the code that calls slot +0xB8 with kind 8 was not found (scans of
  CALL [reg+0xB8] sites found only unrelated callers). The class name says attack rate, the item text says spell
  damage; whether it scales the whole spell damage or only the catalyst's stat bonus needs the consumer or a live
  test (cast the same spell at hollow 0 and 10, read the damage).
- Whether hollow level can go above 10 in SOTFS (it would not matter: the rate stays 0.1).
