# Port: "draw the whole map" chain + App / game-object globals, Network Test -> retail v1.00

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Static only (2026-10-17, game not running). Method: two-level TOC resolution of each site, callers / vtable slots,
`xmatch.py` pairs, side-by-side disassembly of NT and retail functions, plus a linear symbolic register tracker over the
whole text of both EBOOTs (scratch: `agentA\symscan.py`, `fnacc.py`, `rng.py`, `annot.py`, `callers.py` in the session
scratchpad). Nothing here was read live: every retail value still wants one live read before a table entry trusts it.

## 1. Table

| Item | NT | Retail | Status |
|---|---|---|---|
| App pointer global | 0x1b09e50 | **0x1d98310** (bss) | confirmed |
| App vtable / size | 0x19ac3b8 / 0x2490 | **0x1c1f7b8** / 0xf10 | confirmed |
| Map manager MM = [App+0x1c] | +0x1c (size 0xf4) | **+0x1c** (size 0x110) | confirmed |
| W = [MM+4], W+4 = MM | +4 (W size 0x100) | **+4** (W size 0x110) | confirmed |
| W+0x10 current collision part | +0x10 | **+0x10** | confirmed |
| W+0x14 applied collision part | +0x14 | **+0x14** | confirmed (writer); the compare that triggers the recompute not re-derived |
| Map id | W+0x1c | **moved to MM+0xb4** (current) / MM+0xb0 (last announced); W+0x1c is now the map-instance array | confirmed |
| Map instances | [W+0x20 + 4i], i < s16 [W+0xb0] | **[W+0x1c + 4i], i < s16 [W+0xc8]** | confirmed |
| Map instance +4 | map id | map id | confirmed |
| Collision loader L | [inst+0xec] | **[inst+0xfc]** | confirmed |
| Collision parts / count | [L+8] / [L+0xc] | **[L+8] / [L+0xc]** | confirmed |
| MO = [MM+8] | 0x14-byte object, vtable 0x19b4f20 | **0x34-byte object, vtable 0x1c2bfa8** (different class) | confirmed |
| MO+0xc88 / +0xc8c list | works live on NT | **does not port** (see 3) | n/a |
| Collision part vtable | 0x19b46d0 | **0x1c2b770** | confirmed |
| part+0x18 -> loaded MSB entry | +0x18 | **+0x18** | confirmed |
| entry+0x40 display mask (4 x u32) | read by MM::Update | **read by MM::Update** | confirmed |
| part+0x0c component | +0x0c | **+0x0c** | probable |
| MapHitBackreadComponent vtable | 0x19b5fa0 | **0x1c2cee0** | probable (state byte +0x18 not re-derived) |
| part+0x14 -> map info (+4 id, +8 index) | +0x14 | +0x14 | confirmed |
| part type byte (2 = collision) | +0x7e | **+0x7a** | confirmed |
| MM::Update (recompute driver) | 0x6a2200 | **0x8af0a0** | confirmed |
| W::SetCurrent / W::Apply | 0x6d0ac0 / 0x6d0a58 | **0x8dff58 / 0x8e01f0** | confirmed |
| Player chr pointer used by MM::Update | **App+0x60** | **App+0x64** | confirmed (affects the whole table) |
| Player floor-collision getter | 0x5a5118: tag [[chr+0xa8]+8]+0x138 | **0x792428: tag [[chr+0xb8]+8]+0xd0** | confirmed |
| chr floor-collision field / cached | chr+0x35c / chr+0x358 (refresher 0x5a58fc) | **chr+0x368 / chr+0x364** (refresher **0x7926c0**), new byte chr+0x36c; +0x368 only filled on one branch | probable |
| Game-object pointer global (world speed) | 0x1b0b1d8 | **0x1d99650** | confirmed (the global); its value not read live |

## 2. Evidence (NT site = retail site)

- **App global.** `new App` then store: NT 0x672e44 `li r3,0x2490` .. 0x672e70 `bl 0x3ffce8` (ctor) .. 0x672e78/80 `lwz r9,
  -0x7ff8(r30)` = 0x1b09e50, `stw r31,0(r9)` = retail 0x878a4c `li r3,0xf10` .. 0x878a78 `bl 0x547160` .. 0x878a80/88
  global 0x1d98310. The only store to each global. Retail has 2102 code sites reading 0x1d98310 (NT 1591 for 0x1b09e50).
  Matched readers: NT 0x409c04 (App -> +0x1c -> +0xcc) = retail 0x55734c (App -> +0x1c -> +0xd4); NT 0x3c1a60 = retail
  0x4da248; the part ctor NT 0x69e3d0 = retail 0x8a9010.
  **0x1edfbe0 is not the App global**: it is the counterpart of NT 0x1c51680 (a 64-byte vector block the collision part
  ctor copies into part+0x20..+0x5f: NT 0x69e3fc = retail 0x8a903c).
- **App vtable.** App dtor NT 0x4057bc `lwz r0,-0x7f60(r30)` = 0x19ac3b8 / retail 0x54ffd4 = 0x1c1f7b8. Slot +0x10 is the
  state machine in both (NT 0x3fdf38 = retail 0x545e48, the known pair); slots +0x00..+0x0c are ctor/dtor pairs.
- **MM.** NT 0x58a030 `li r3,0xf4` / ctor 0x6a1f98 / init 0x6a1ac8 / 0x58a070 `stw r29,0x1c(App)` = retail 0x774b9c `li
  r3,0x110` / ctor 0x8ae780 / init 0x8ae0f8 / 0x774bdc `stw r29,0x1c(App)` (xmatch unique hit). Both ctors zero +4, +8,
  +0xc, +0x10; members from MM+0xb0 on are +8 in retail (init: NT stw +0xb0/+0xb4/.. = retail +0xb8/+0xbc/..).
- **W.** MM::CreateW NT 0x6a193c: `li r3,0x100`, `bl 0x6d0e40` (W ctor), 0x6a19b8 `stw r29,4(MM)`, `bl 0x6d0cf0` (init) =
  retail 0x8addcc: 0x8adf8c `li r3,0x110`, `bl 0x8e04c8`, 0x8adfc8 `stw r31,4(MM)`, `bl 0x8e0368`. W ctor (NT 0x6d0e40 /
  retail 0x8e04c8): +0 heap, **+4 = MM**, +8/+0xc/**+0x10/+0x14 = 0**, +0x18 = -1; NT +0x1c = -1 and memset +0x20..+0xaf,
  u16 +0xb0/+0xb2; retail memset +0x1c..+0xc3, u16 +0xc4/+0xc6/+0xc8/+0xca, no separate +0x1c field.
- **W+0x10 (current).** NT W::SetCurrent 0x6d0ac0: if [W+0x10] != part, map id from [[part+0x18]+0x6c]+0x28 (or area x 10000
  + block x 100 + ..) into W+0x1c (calls the [App+0x21d0] notifier on change), then 0x6d0b68 `stw part,0x10(W)`. Retail
  0x8dff58 is just `if (part) W+0x10 = part`; the map-id part moved into MM::Update (retail 0x8af3a4..0x8af46c: MM+0xb4 =
  [[part+0x18]+0x6c]+0x28, compare MM+0xb0, notifier [App+0xd20], MM+0xb0 = MM+0xb4). NT App+0x21d0 -> retail +0xd20 is the
  known App tail shift.
- **W+0x14 (applied).** W::Apply NT 0x6d0a58 = retail 0x8e01f0 (same code; retail also sets byte W+0x105): calls the
  sub-object [W+0xc] update (NT 0x6cf58c / retail 0x8debe4), then `stw part,0x14(W)` (NT 0x6d0a90 / retail 0x8e022c) and
  W+0x18 = [part+0x14]+8 (NT 0x69d9b8 / retail 0x8a86a4). Only caller: MM::Update (NT 0x6a23e0 / 0x6a2538 = retail 0x8af2e8
  / 0x8af580). NT W's big per-frame method 0x6d12cc reads W+0x14 / +0x18; where exactly "+0x10 != +0x14" forces the
  recompute was not traced (the s21 live test is the proof on NT; the retail path is structurally the same).
- **entry+0x40.** NT MM::Update 0x6a2398 `bl 0x69e250` = 4 x lwz [[part+0x18]+0x40..+0x4c] (0x69e250..0x69e278), copied to
  MM+0x98..+0xa4; retail inlines it at 0x8af288..0x8af2ac (`lwz r9,0x18(part); lwz 0x40/0x44/0x48/0x4c(r9)` -> MM+0x98..).
  (NT 0x69e280 is the same getter for +0x50..+0x5c.)
- **MM::Update** NT 0x6a2200 / retail 0x8af0a0: player = **[App+0x60] (NT 0x6a22ac) / [App+0x64] (retail 0x8af1a4)**, floor
  part = getter(player) (NT 0x5a5118 / retail 0x792428), accepted only if part type byte == 2 (NT part+0x7e at 0x6a22d8 /
  retail part+0x7a at 0x8af1d0), old = [W+0x10] (NT 0x6a22b0 / retail 0x8af1a8), SetCurrent, mask copy, Apply.
- **Map instances and the collision list.** W::Update NT 0x6d2060 loops 0x6d25a4..0x6d2628: n = s16 [W+0xb0], inst =
  [W+0x20+4i], calls 0x6bbcf8(inst) -> 0x6bb928 -> 0x6b997c(inst) (also matches the current one by [inst+4] == map id of
  [W+0x10], 0x6d25dc). Retail W::Update 0x8e18e8: n = s16 [W+0xc8] (0x8e1c0c), inst = [W+0x1c+4i] (0x8e1c60..0x8e1c98),
  calls 0x8cb070 -> 0x8cab80 -> 0x8c8838(inst); id compare at 0x8e1c44. In the per-map loader (NT 0x6b997c / retail
  0x8c8838) the collision loader L (NT 0x20 bytes, retail 0x24) is built and stored at **inst+0xec (NT 0x6b9c44) / inst+0xfc
  (retail 0x8c90e4)**, then filled by NT 0x6bc8d8 / retail 0x8cb5f0: each part from the factory (NT 0x6df2b0 / retail
  0x8ef8f8, the one that adds the backread component) goes to **[L+8][i]** (NT 0x6bcd48 / retail 0x8cba50), count to
  **L+0xc** (NT 0x6bcd84 / retail 0x8cba8c). Neighbouring loaders: NT inst+0xe4/+0xe8/+0xf0, retail +0xf4/.. (other part
  kinds).
- **[MM+8].** NT MM 0x6a2628 (called from App init 0x405c54): `li r3,0x14`, ctor 0x6d26e0 (vtable 0x19b4f20, base
  0x19a9f98), `stw 8(MM)`, load 0x6d2848 with [[MM+0xc]+0x90]+8/+0xc. Retail 0x8aecd8 (called from App init 0x54da28):
  `li r3,0x34`, ctor 0x8e1ed8 (**vtable 0x1c2bfa8**, same base 0x1c13de0), `stw 8(MM)` (0x8aed34), load 0x8e2360 with the
  same args. The class with NT's vtable layout (retail 0x1c36aa0: ctor 0xcbfe10, dtors 0xcbfe44 / 0xcc00fc, xmatch-paired
  with NT 0x6d2714 / 0x6d27a0) is created elsewhere in retail (0xcae4dc stores it at owner+0x8c).
- **Collision part.** Ctor NT 0x69e3b0 = retail 0x8a8ff0 (same stores: +0x10..+0x1c, +0x20..+0x5f vector block, +0x60..+0x6c
  mask from a global, +0x70..; type byte NT +0x7e / retail +0x7a). Vtables NT 0x19b46d0 / retail 0x1c2b770 have the same
  slot layout. Slot +0x28 (NT 0x69e990 / retail 0x8aa21c) = init: +0x14 = a1 (map info), **+0x18 = a2 (MSB entry)**, +0x1c =
  a3, and copies entry+0x30..+0x3c into +0x60..+0x6c (so retail 0x599708 / 0x599e60 reading +0x60.. read the draw mask
  copy, as on NT). Slot +0x20 (NT 0x69dbf0 / retail 0x8a8988) reads [part+0xc] and calls its virtuals (component).
- **Backread component.** Ctor NT 0x7063c0 (vtable 0x19b5fa0) = retail 0x914b6c (vtable 0x1c2cee0), 0x30 bytes in both,
  only caller the collision factory (NT 0x6df41c / retail 0x8efb08), which appends it to the host's component list. That it
  is what [part+0xc] points to is inferred from the NT live note, not from a store.
- **Floor collision on the chr.** NT getter 0x5a5118 (tag word at [[chr+0xa8]+8]+0x138, top nibble 1 -> 0x3c1828, 7 ->
  0x3c144c) = retail 0x792428 (tag at [[chr+0xb8]+8]+0xd0; xmatch's 3 hits are 0x7920f8 / 0x792428 / 0x7924dc, the same
  getter family). NT refresher 0x5a58fc (virtual; OPD 0x1a4c218 in vtable slots 0x19b21a4 / 0x19b3a9c): tag
  [[chr+0xa8]+8]+0x134, chr+0x35c = part, chr+0x358 = 0x427610(part). Retail 0x7926c0 (virtual; OPD 0x1cd3b68 in slots
  0x1c28f6c / 0x1c2ab1c): tag [[chr+0xb8]+8]+0xcc, fallback 0x4daf90([chr+0x94]+0x23c); returns chr+0x364; if the byte
  [chr+0x94]+0x240 >= 0: chr+0x364 = 0x578be0(map index, byte), chr+0x36c = byte, **chr+0x368 = 0**; else chr+0x364 =
  0x578ce0(part, 1), **chr+0x368 = part**, chr+0x36c = -1. Not a byte match (restructured), so "probable". The draw logic
  does not use this field: MM::Update calls the getter itself.
- **Game-object pointer.** The class ctor NT 0x927e68 (vtable 0x19bb7a0, clears the global at 0x927ec4) = retail 0xb27ab8
  (vtable 0x1c31f70, clears **0x1d99650** at 0xb27b14), unique xmatch hit, identical code. Member use through the global
  matches: +0x0c/+0x10/+0x14/+0x18, +0x1c (byte), +0x20 (NT 47 / retail 65 reads), +0x24, +0x2c, +0x30, +0x34, +0x38,
  +0x40, +0x44, +0x48, +0xb8, +0xc3..+0xc5, +0x17c, +0x188 (NT 54 / retail 59), +0x18c; NT +0x190 -> retail +0x194. The
  world-speed fields (+0xe0 mode, +0xe4 switch) are used by the frame function on `this` (NT 0x927010 = retail 0xb26d38,
  identical), not through the global, so for the table: **[0x1d99650]+0xe4 / +0xe0**. The non-zero store of the global was
  not found by the TOC-idiom scanner in either build (NT reads it 345 times), so "holds main's frame" is the NT live fact.

## 3. What this means for the port

1. The NT table's collision list [[[App+0x1c]+8]+0xc88] lies 0xc88 bytes into a 0x14-byte allocation: it only worked because
   the collision loader L happened to be allocated at MO+0xc80 in the same heap (that is: NT [MO+0xc88] = [L+8]). In retail
   [MM+8] is a different 0x34-byte object, so **do not port +0xc88**. Use the real chain instead (works on both builds):
   - retail: W = [[[0x1d98310]+0x1c]+4]; for i < s16 [W+0xc8]: inst = [W+0x1c+4i]; L = [inst+0xfc]; parts = [L+8], n = [L+0xc]
   - NT (to check the claim live): inst = [W+0x20+4i], i < s16 [W+0xb0]; L = [inst+0xec]
   - checks: [W+4] == MM; part vtable 0x1c2b770; [[part+0x18]] == [part+0x18] + 0x70 (loaded entry); type byte part+0x7a == 2;
     the current map's instance is the one with [inst+4] == [[[W+0x10]+0x14]+4].
2. Draw all = 0xff x 16 at [part+0x18]+0x40 for every collision part of every instance, then W+0x14 = any other part (as on
   NT). Retail loads several neighbouring maps at once (that is the instance array), so loop over all instances.
3. Map id for logging: MM+0xb4 (retail), not W+0x1c.
4. **Player pointer: App+0x64 in retail**, not +0x60 (MM::Update; 225 reads of App+0x64 vs 4 of App+0x60). Check every
   other table entry that roots at [App+0x60].
