# Retail v1.00 layouts for the equipment / inventory, AI freeze and enemy swap entries (static, 2026-10-17)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Static only (game not run). Method: NT code found by vtable install sites (data word -> TOC two-level loads, long
window), displacement / chain scans and `bl` callers; matched into retail with `tools\xmatch.py` (strict or voted
windows), then both disassemblies read side by side (and Ghidra `DefineAndDecomp` for 0x3f20b0 / 0x3e8400 / 0x5337b8 /
0x3026d8 / 0x4268f4 / 0x7cef78). Scratch scanners: `%TEMP%\...\scratchpad\agentC\` (vtref, farref, callers, chainscan,
bigoff, pairscan, vtcmp, globrefs). Status: **C** = confirmed by matched code in both builds, **P** = probable (one
piece of evidence or inferred from structure), **N** = not found.

## 0. App root

| item | NT | retail | st | evidence |
|---|---|---|---|---|
| static App pointer | 0x01b09e50 | **0x01d98310** | C | AI think NT 0x74cc08 / retail 0x95c228 load the global, `lwz +0x14` and pass it to the AI-off reader; 2053 code sites load 0x1d98310 (NT global: 1575). [Cheat table port](cheat-table-port.md)'s "probably 0x1edfbe0" has only 150 sites: **use 0x1d98310** |
| App+0x14 | AiManager | **AiManager (same offset)** | C | as above |
| App+0x54 | -> Q = [App+0x54]+8 holder | **same** | C | NT 0x2bd240 = retail 0x2e8290 (identical hops `lwz 0x54`, `lwz 8`, accessor `lwz r3,4(r3)` NT 0x36d370 = retail 0x2d9710) |
| App+0x0c, +0x20 | CharacterManager, generator system | same | C / P | +0x0c: think / CharMgr sites below; +0x20 kept per state-28 handler ([cheat table port](cheat-table-port.md)) |

## 1. Player data, equipment block

P = [chr+0x2d4] retail ([port: player character](port-character.md)). **P is a 0x28-byte handle**, not the big player-data block: NT ctor 0x5d9a68 =
retail **0x7cea50** (byte-identical layout: vtable, +4, +8, +0x0c..+0x20 pointers, +0x24 flags).

| item | NT | retail | st | evidence |
|---|---|---|---|---|
| P vtable | 0x019b2c20 | **0x01c29a70** | C | ctor 0x7cea50 `lwz -0x8000(r30)` via TOC-0x7174 (NT TOC-0x7558); slot order differs from NT (NT +0x18 0x5d9e20 = retail +0x14 0x7cef78) |
| E = [P+0x10] | equipment | **same** | C | P update NT 0x5d9e20 = retail 0x7cef78 (reads [P+0x10]+flags) |
| spell state = [P+0x14] | | **same**; byte flags +0x40/+0x41/+0x43 -> **+0x30/+0x31/+0x33** | C | same pair |
| [P+0x18] (player-data object) | u64 flag +0x730 | u64 flag **+0x8c0** | C | same pair (ld 0x730 / ld 0x8c0) |
| E rebuild flag | E+0xcc \|= 0x4000 | **E+0x4ec \|= 0x80000000** (sign bit) | C | 0x7cef78: `lwz 0x4ec; cmpwi; blt` -> FUN_007d9998([P+0x18]) then `clrlwi r0,r0,1`; setter 0x7eb4a4 `oris r0,r0,0x8000; stw 0x4ec` (NT 0x5eef84 `ori 0x4000; stw 0xcc`) |
| E layout | +0 u32, +0x04 u32[46] ids, +0xbc u16[..], +0xcc flags | **+0 u32, +0x04 rec[52] x 0x14 bytes, +0x414 u32[52] (init -1), +0x4e4 u16[4], +0x4ec flags** | C | E init NT 0x5ef288 (0x2e x u32 = -1) vs retail **0x7eb520** (0x34 records, then u32[0x34] at +0x414 = -1, u16s +0x4e4..+0x4ea = 0, clears bit 31 of +0x4ec) |
| slot record (0x14) | (u32 id) | **+0 u32 item type/category word, +4 u32 id, +8 u8 (0xff when empty), +0xc u16 (count?), +0xe u8, +0xf u8, +0x10 f32 (TOC constant; durability?)** | C (offsets) / P (meanings) | init 0x7eb520; record builder 0x7ec96c (fills it from FUN_0049a1ec(category, id) and passes it to SetSlot **0x7ec5d8**, which memcpys the 0x410-byte array and sets the flag); getter 0x7ebd60 `lwz 8(E+0x14*i)` for i <= 0x33; E compare 0x7ec158 gathers `lwz 8(E+0x14*i)` for 52 slots |
| **id of slot i** | E+0x04+4*i | **E+0x08+0x14*i** | C | getter / compare above |

Slot index groups (retail index-mapper functions `addi r3,r3,K; extsw; blr` at 0x7f4e38.., the weapon table in
0x4268f4, the armour / ring list in 0x3026d8):

| group | NT index (E offset) | retail index | retail id offset (E+8+0x14*i) | st | evidence |
|---|---|---|---|---|---|
| weapons L1 R1 L2 R2 L3 R3 | 0-5 (+0x04..+0x18) | **0-5**, same order | +0x08, +0x1c, +0x30, +0x44, +0x58, +0x6c | C | 0x4268f4: left hand = 0,2,4, right = 1,3,5 via identity mapper 0x718400; creation 0x8024d0 |
| head body arms legs | 6-9 (+0x1c..+0x28) | **6-9** (mapper 0x7f4e38, +6) | +0x80, +0x94, +0xa8, +0xbc | C | creation 0x8024d0 (`li r4,1` + 0x7f4e38), 0x3026d8 |
| (NT "1000, 1000") | 10-11 | **10-11** | +0xd0, +0xe4 | P | gap between the mappers |
| ammo arrow1 arrow2 bolt1 bolt2 | 12-15 (+0x34..+0x40) | **12-15** (mapper 0x7f4e44, +0xc) | +0xf8, +0x10c, +0x120, +0x134 | P | by elimination; order assumed as NT |
| **rings 1-4** | 16-17 (+0x44 / +0x48) | **16-19** (mapper 0x7f4e50, +0x10) | +0x148, +0x15c, +0x170, +0x184 | P (strong) | 0x4268f4 maps descriptor type 0 weapon / 1 armour (+6) / 2 -> +0x10 (the durability kinds); 0x3026d8 pairs armour with list codes 0xd2/0xdc/0xe6/0xf0 and the four +0x10 slots with 0xfa |
| quick items 1-10 | 18-27 (+0x4c..+0x70) | **20-29** (mapper 0x7f4e5c, +0x14; 10 calls in 0x365d88) | +0x198 .. +0x24c (step 0x14) | P | count 10 |
| spells | 28-37 (+0x74..+0x98, 10) | **30-43** (mapper 0x7f4e68, +0x1e; **14** calls 0..13 in 0x383f64) | +0x260 .. +0x364 | P | 6+4+4+4+10+14 = 42 = retail pointer-table size (sect. 2); NT 6+4+2+4+10+10 = 36 |
| other 8 | 38-45 | **44-51** (mapper 0x7f4e74, +0x2c) | +0x378 .. | P | used with inventory category 6 (0x34af88) and 0x47d66c; meaning unknown |

Not found (N): equip load, active left / right weapon, cast counter, last cast id. NT "P+0x174 / +0xb1c / +0xb20 /
+0xd7c / +0x13f4" are **not P fields**: P is 0x28 bytes, so on the demo heap they are offsets into the spell-state
object [P+0x14] (= P+0x130): +0x44, +0x9ec, +0x9f0, +0xc4c, +0x12c4. Its retail layout moved (flag bytes -0x10 at
least); re-find them live, or by matching the NT readers of [P+0x14]+those.

## 2. Inventory

**The NT chain [[[App+0x54]+8]+4] only works by heap adjacency.** Code (both builds): Q = [[App+0x54]+8];
[Q+4] = a 0x54-byte object (params holder); **X = [Q+8]** (NT 0xc080 bytes, ctor 0x3e8400; retail 0x100e8, ctor
**0x520198**, allocated by 0x5272f0 = NT 0x3eb2c4, same shape); **store S = [X+8]** (NT 0xc4e0, ctor 0x3f20b0; retail
0x1e108, ctor **0x5337b8**). On the demo heap X sat at [Q+4]+0x60 and S at +0xc0f0, which is why "+0x74 mirror,
+0xc104 pool" worked. Use explicit hops on retail:

| item | NT (via M = [[[App+0x54]+8]+4]) | NT real owner | retail | st | evidence |
|---|---|---|---|---|---|
| X | M+0x60 | [Q+8] | **[[[App+0x54]+8]+8]** | C | 0x5272f0 `stw r29,8(r31)` after `bl 0x520198` (NT 0x3eb34c) |
| mirror array | M+0x74, 0xc per entry | X+0x14 (memset 0xc05c) | **X+0x18** (memset 0x100bc) | C (region) / P (entry size) | 0x520198 vs 0x3e8400 |
| saved u16 index table | pool-0x8c = M+0xc078 | X+0xc018, u16[44] = 0xffff | **X+0x1001c, u16[50] = 0xffff** | C (position) | 0x520198 loop `li r9,0x32` at X+0x1001c; NT loop 0x2c at X+0xc018 |
| other X fields | | X+0xc070.. (999, flags) | X+0x100d4.. (999 at +0x100dc / u16 +0x100e0, flags +0x100e4) | C | same |
| S = store | M+0xc0f0 | [X+8] | **[X+8]** | C | 0x520198 `stw r29,8(r31)` after `bl 0x5337b8` |
| node pool | M+0xc104, 0x700 x 0x1c | S+0x14 | **S+0x14, 0xf00 nodes x 0x20 bytes** (memset 0x1e000) | C | 0x5337b8 loop `li 0xf00`, stride 0x20 |
| free-list sentinel | M+0x184e8 | S+0xc3f8 = last node (0x6ff) | probably the last node: **S+0x14+0x20*0xeff = S+0x1dff4** | P | by analogy only (NT: S+0x14+0x1c*0x6ff = S+0xc3f8 = M+0x184e8) |
| node fields | prev, next, id, id, u16 index, u16 flags, 8-byte tail | | 0x20 bytes; byte +8 cleared by the ctor; field order **N** | N | |
| pointer tables | M+0x18508 (36 u32) | S+0xc418 (memset 0x90) | **S+0x1e018 (42 u32, memset 0xa8)** | C | 0x5337b8 |
| equip-list objects | 0x2c bytes, vt 0x019abf58/98/d8 | pointers at S+0xc4b8.. (15 + 4 + 1 class A, 1 + 2 class B, 1 class C) | pointers at **S+0x1e0d0 (15 lists), +0x1e0d4 (4), +0x1e0d8 (1), +0x1e0dc (1, class B), +0x1e0e0 (2, class B), +0x1e0e4 (class C), +0x1e0e8 (new)**; list ctors 0x52cbfc / 0x52d034 / 0x52f148 / 0x52ee84 | C (pointer slots) / N (list codes, spell list) | 0x5337b8 vs 0x3f20b0 |
| saved-table order | 0-5 weapons R1 L1.., 6-9 armour, 10-11 rings, 12-15 ammo, 16-25 items, 26-35 spells | | probably 0-5 / 6-9 / **10-13 rings, 14-17 ammo, 18-27 items, 28-41 spells** | P | 42 pointer-table entries; not code-proven |
| list codes | 0x8c / 0x96 hands, 0x190-0x1ae armour, 0x1f4 rings, 0x258 ammo, 0x2bc items, 0x320 spells | | armour **0xd2 / 0xdc / 0xe6 / 0xf0**, rings **0xfa** seen as category codes in 0x3026d8 (FUN_002e7c98) | P | numbering changed; others N |

## 3. AI freeze

| item | NT | retail | st | evidence |
|---|---|---|---|---|
| AiManager vtable | 0x019b6dd0 | **0x01c2dc50** | C | slot +4 deleting dtor NT 0x746d20 = retail 0x956960 (strict match) |
| all-AI-off counter | AiManager+0x14d0 | **AiManager+0x0c** | C | reader NT 0x744d7c = retail **0x954b30** (`lwz r0,0xc(r3)`), called by think at 0x95c234 with [App+0x14]; the NT inc / dec helper 0x744d48 has no strict match |
| CharacterManager vtable | 0x019b3018 | **0x01c29f08** | C | ctor NT 0x605500 = retail 0x80aa94 (strict n=32); dtors 0x804060 / 0x804400 install it |
| chr vector | +0x0c begin / +0x10 end (+0x14 cap) | **same** | C | ctor: allocator `stwu 8`, then +0xc/+0x10/+0x14 = 0 in both |
| load-wait object | [CharMgr+0x3b0], pending count +8 (release: write 1) | **[CharMgr+0x418], pending count +0xc** | C | check NT 0x5fee18 = retail **0x7fe824** (`lwz 0x418; lwz 0xc; cmplwi 1`); +0x418 also in 0x7ff1f8, 0x80147c, 0x803058, 0x804458, 0x8040b4 |
| chr -> AI | [[[chr+0x9c]+0x0c]+0x10] | **[[[chr+0xac]+0x0c]+0x10]** | C | chr hop: 9 matched sites (e.g. 0x46a5e0, 0x47c310, 0x7b2a1c `lwz 0xac` / NT `lwz 0x9c`), +0xc unchanged, the +0xc object's methods (retail 0x821bf8 / NT 0x61e8dc) are the AI owner's class; owner's creator NT 0x61dbfc = retail **0x8215ec** stores the AI at owner+0x10 in both |
| AI object | 0x940 bytes, ctor 0x74a040 | **0x8e0 bytes, ctor 0x95da40** | C | 0x8215ec `li r3,0x8e0` |
| AI vtable | 0x019b6de0 | **0x01c2dc60** | C | ctor 0x95da40 `lwz -0x7fc0(r30)`; slots +0xc, +0x14 think (0x95c190), +0x18, +0x20 update (0x95cc20) match NT |
| AI+0x1c chr | +0x1c | **+0x1c** | C | update 0x95cd60 `lwz 0x1c(r28)` |
| flags u64 (bits 55 / 54 active) | +0x38 | **+0x38**, same bit tests | C | update / think prologues |
| frame-skip byte | +0x2a8 (ctor 2) | **+0x19c** (ctor 2) | C | update 0x95cc94, think 0x95c1d4, ctor 0x95dacc |
| enable byte (think) | +0x3bc | N | N | |
| enemy chr vtable | 0x019b20f0 | **0x01c28eac**? | P (vote, 8 slot functions agree) | vtcmp --find; player chr vtable 0x019b39e8 -> 0x01c2aa5c? (7 agree) |

**Output-block reset.** NT does it inline at 0x74a978 (start of every real update); retail calls a virtual instead:
update 0x95ccd8 `lwz r9,0xb0(vt)` -> AI vtable slot +0xb0 = **0x95dd74**, which resets the block at **B = AI+0x1d0**
(NT block at AI+0x2b0: same internal layout, shifted by -0xe0). The ctor 0x95da40 writes the same fields at
r27 = AI+0x1d0.

| field | NT | retail | st |
|---|---|---|---|
| vector move (zero) | +0x2b0 | **+0x1d0** | C (offset) |
| vector face (zero) | +0x2c0 | **+0x1e0** | C |
| vectors (zero) | +0x2d0, +0x2f0, +0x300 | **+0x1f0, +0x210, +0x220** | C |
| word(s) = 0 | +0x2e0, +0x2e4 | **+0x200** only (no +0x204 write) | C |
| input bits u64 (CT writes u32 = 1 at its first word) | +0x310 | **+0x230** (same top-word bit clears; the low-word field cleared is 5 bits wide in retail vs 3 in NT) | C (offset) / P (writing u32 1) |
| bytes = 0 | +0x315, +0x316, +0x317, +0x318 | **+0x235, +0x236, +0x237, +0x238** | C |
| attack id = -1 | +0x31c | **+0x23c** | C |
| = -1 | +0x328 | **+0x248** | C |
| = 0 | +0x32c | **+0x24c** | C |

**Live 2026-11-18 (Dragon Shrine, `tools\vgens.py`):** the enemy list works as written (player first, then only the
enemies created near the player; chr +0x14 = EnemyParam id). The Drakekeepers (331xxx) have chr vt 0x01c28ea8 and AI vt
0x01c2dc60, so "Enemy AI off" froze them. The three Dragon Knights (835000 / 835100 / 835200) have chr vt **0x01c2aa58 =
the player's own class** (type +0x1c still 2), and [[[chr+0xac]+0xc]+0x10] lands on an object whose first word is a heap
address, not an AI vtable: the table skips them and they attacked (the author). Other humanoid 8xxxxx enemies probably share
this class (unchecked); their AI object is not mapped.

**Player-class enemies' AI path (2026-11-18, mapped the same day: code in DS2V by headless Ghidra + linear capstone
listings, layouts read live (read only) on the three knights at Dragon Shrine).** The player class 0x01c2aa58 is a
subclass of the enemy class 0x01c28ea8 (ctor 0x847f84 calls the enemy ctor 0x791b74; 11 of 80 vtable slots overridden;
its init slot +0x108 = 0x847fdc adds objects at chr+0x370..+0x37c). chr+0xac is the same kind of object for both: a
0x18-byte **control holder** handed in by the creator (init 0x792b08 takes it from its argument +0x24; default ctor
0x79d940, base vt 0x01c29260), whose +0xc is the **current controller**. Its slot +0x20 = 0x79e1c0 picks the controller
from the holder's own getters (+0x34 for mode 0, +0x30 for modes 1 / 2). Live:

| chr | holder vt | controller [H+0xc] vt | AI |
|---|---|---|---|
| player (type 1) | 0x01c222b8 | 0x01c2a398 (pad controller) | none |
| Drakekeepers 331xxx | 0x01c2b3e8 | 0x01c2a1b0 (AI owner: AI at +0x10, created by its slot +0x98 = 0x8215e4) | vt 0x01c2dc60 |
| Dragon Knights 835xxx (type 2) | 0x01c22278 | **0x01c21e58 (player-type AI controller)** | **at controller+0x14, vt 0x01c1d550** |

- **Player-type AI controller** (vt 0x01c21e58, methods 0x5b1cb8..0x5b20b0, ctor ~0x5b2060): +0x8 chr, **+0x10 virtual
  pad** (0xd0-byte struct, +0 = chr), **+0x14 AI**. Update (slot +0x18 = 0x5b1e44): pad pre-step 0x5b0bc4 (clears the pad's
  +0x74 block), AI think (AI vt +0x14) with the time step, post-step 0x5b1a90, then copies pad bytes +4..+5, +6..+8,
  +9..+0xc into the controller output [ctl+0xc] (0x821cf4 / 0x821d5c / 0x821dcc, shared with the AI owner). Teardown
  (slot +0xc = 0x5b1f30) unregisters +0x14 from the AiManager, so the AI is registered there like any other.
- **Player-type AI** (vt 0x01c1d550, the only subclass of the enemy AI class 0x01c2dc60 in the EBOOT, ctor ~0x458714):
  slots +0x08..+0x90 are the base class's, including **think +0x14 = 0x95c190** and **update +0x20 = 0x95cc20**, so the
  **frame-skip byte +0x19c** works the same (think runs only while it is 0; update decrements it and returns while it is
  not), +0x1c = chr, and the **output block at +0x1d0** has the base layout (its reset slot +0xb0 = 0x457c50 calls the
  base reset 0x95dd74, then zeroes its own fields +0x8d8..+0x910). New: **+0x8d4 = the controller's virtual pad**; slot
  +0x9c (0x457d14, called first by think) resets the pad (0x5b11a8: bytes +4..+0xc = 0, +0x10 = zero vector, +0x20 /
  +0x30 = (0,0,1,0), +0x74 block); slot +0xa8 (0x457f5c, called by think after the AI has decided) **copies the output
  block into the pad**: vectors +0x1d0 / +0x1e0 / +0x1f0 -> pad +0x10 / +0x20 / +0x30, input bits +0x230 and attack id
  +0x23c -> pad buttons through 0x5b1238 / 0x5b04e8, the subclass fields +0x8d8.. -> pad requests.
- So holding +0x19c at 0xff stops think and update for the knights too, but the pad keeps whatever the last think put
  there (the controller copies it every frame). An idle AI frame leaves the pad with zero buttons and, after 0x457f5c,
  zero vectors (the base reset vector is (0,0,0,0)).

Retail freeze for player-class enemies (built into DS2V.CT 18 Nov, profile keys CHR_TYPE / PCTRL_* / PAI_* / VPAD_*):
chr vt 0x01c2aa58 **and** u32 chr+0x1c = 2 (the player is type 1); ctl = [[chr+0xac]+0xc] with vt 0x01c21e58; AI =
[ctl+0x14] with vt 0x01c1d550, AI+0x1c = chr and AI+0x8d4 = pad = [ctl+0x10]. Write the usual block below on the AI
(frame-skip first), then zero pad bytes +4..+0xc and pad +0x10..+0x3f; untick: AI+0x19c = 0. Read-only dry run of the
lookup at Dragon Shrine (18 Nov): 4 Drakekeeper AIs + 3 knight AIs with their pads; the player skipped.

**Network Test:** `make_ct.py` filters on the enemy class 0x019b20f0 the same way, but its EBOOT has **one** AI class
(0x019b6de0; no other vtable shares its think / update slots) and no counterpart of the player-type AI controller or the
virtual pad (retail 0x457f5c / 0x5b11a8 / 0x5b1e44 have no NT twin by xmatch.py reversed), so this retail path cannot
exist there. The NT's player class 0x019b39e8 has 3 creation sites (retail 4: 0x80f490 / 0x80fd54 / 0x810d94 /
0x812a18; only 0x810d94 has an NT twin, 0x6086ec). Whether any NT character of the player class is AI-driven (it would
then hang off the ordinary chain and be skipped only by the class test) is not checked; m10_23 registers 754000 /
754001, 783000 / 783001 and 810000 (one spawn each, no names in the NT EnemyParam), the likely candidates.

Retail freeze recipe (live 18 Nov: froze the Drakekeepers): for each enemy chr in [CharMgr+0xc..+0x10), AI = [[[chr+0xac]+0xc]+0x10] (check vt
0x01c2dc60), zero 16 bytes at +0x1d0/+0x1e0/+0x1f0/+0x210/+0x220, u32 +0x200 = 0, u32 +0x230 = 1, bytes +0x235..+0x238
= 0, +0x23c = -1, +0x248 = -1, +0x24c = 0, byte **+0x19c = 0xff**; untick: +0x19c = 0.

## 4. Enemy generators

| item | NT | retail | st | evidence |
|---|---|---|---|---|
| generator object vtable | 0x019b6ae0 | **0x01c2d940** | C | ctor NT 0x7337c0 = retail **0x940298** (`lwz -0x7f10(r30)`, TOC-0x6d68) |
| +0x38 manager, +0x3c GeneratorParam row, +0x40 GeneratorRegistParam row | | **same offsets** | C | ctor: +0x38 = [arg+4], +0x3c = [arg+0x10], +0x40 = [arg+0x14] in both |
| +0x10 position | +0x10 / +0x18 | N (base ctor 0x935030 = NT 0x7293f8 not compared) | N | |
| generator list vtable | 0x019b6ab0 | **0x01c2d8f0** | C | list ctor NT 0x72b11c = retail 0x93671c (identical body) |
| manager +0x10 array / +0x14 count | | **same** | P (strong) | loop NT ~0x72c6c0 = retail 0x937d88 (`lwz 0x10` / `lwz 0x14`, call per element, then +0xc = 0) |
| manager vtable | 0x019b6ac0 | N | N | NT manager ctor 0x72d928 has no retail match |
| chain [[[[App+0x20]+0x34]+0x34]+0x38] | | App+0x20 kept; the two +0x34 hops **unverified** | P | +0x38 = manager is ctor-proven. Safer on retail: validate with the object vtable 0x01c2d940 and fall back to a vtable scan |

## 5. Not done / open

**Closed later the same day in [port: equipment and inventory](port-equipment-inventory.md):** the first two points (node layout, mirror, free list, list
codes = the demo's, record +0 = 0 / +4 = param id / +8 category byte, the 14 slots are attunement slots), the generator
chain (no code twin: retail scans for the object vtable) and the load-wait release. Corrections to the tables above:
the record's +0 is a zero word and its +8 byte the slot's category, not "0xff when empty"; the "0xd2.. / 0xfa" codes are
not list codes; the free list has a head pointer S+0x1e014 (the last node is only its first tail); and (read live) the
lists are per menu category and hold every owned item, so 0x8c / 0x96 are not the hands' lists.

- Node field order, mirror entry size and the free-list sentinel in retail (sect. 2); equip-list codes other than
  armour / rings; spell count 14 vs the game's 10 attunement slots (E 30-43 may hold 14 entries with only 10 used).
- Which id the E record +4 holds for weapons (WeaponParam or item id; retail mostly shares them) and what record +0
  (category word) must be for a swap to apply.
- The derived player values (equip load, active weapons, cast counter, last cast) and the AI enable byte.
- Everything here is static: [confirm live](port-live-check.md) once a retail save boots.
