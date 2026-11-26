# Player character, Havok chain and out-of-bounds check: Network Test -> retail v1.00 (2026-10-17)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Static only (game not run). NT = NPUB90979 (TOC 0x1a97300), retail = BLUS41045 v1.00 (TOC 0x1d1f068). Method: matched
function pairs (`tools\xmatch.py --auto/--multi`), read side by side (`ppcdis.py`), plus a short-function vtable diff
and one NT recording (`..\..\DS2 Network Test\Ghidra Project\export\win\s15\roll_roll3.npz`) to check NT pointers.
Status: **C** = confirmed by a retail code site that is the twin of the NT site; **P** = probable (layout / ctor order
/ PC tables, no single use site traced); **?** = not found.

## 0. Read this first: chr+0x588 and chr+0x5f24 are not chr fields

The NT player chr object is only **0x370 bytes** (allocation `li r3,0x370` at NT 0x6086bc before the ctor call
0x6086ec); retail's is **0x380** (`li r3,0x380` at 0x80f458, ctor 0x847f84). NT ["chr+0x588" and "chr+0x5f24"](cheat-table-port.md#2-per-entry) only work
because the objects created after the chr sit at fixed heap distances on the demo. The NT recording proves the real
chains: `[chr+0x98] = chr+0x500` (component C), `[C+0x88] = [chr+0x588]` (OOB object),
`[[[chr+0xa8]+8]+8] = chr+0x5d90` (physics controller PC), `[PC+0x194] = [chr+0x5f24]` (wrapper W).
**For retail, use the pointer chains below, never chr + large offsets.**

## 1. Table

| What | NT | Retail | St | Retail proof (NT twin) |
|---|---|---|---|---|
| Player chr vtable | 0x019b39e8 | **0x01c2aa58** | C | player ctor 0x847f84 (NT 0x640288) installs it at 0x847fb4 |
| Enemy chr vtable (base class) | 0x019b20f0 | **0x01c28ea8** | C | base ctor 0x791b74 (NT 0x5a5280) installs it at 0x791bf4; 4 sites each build |
| chr size (player) | 0x370 | 0x380 | C | see 0 |
| +0x10 / +0x14 / +0x18 / +0x2c / +0x30 (header: ChrParam id, chr id = -1 at init) | same | **unchanged** | C | base-base ctor 0x79544c is NT 0x5a7f34 instruction for instruction (stores +0x10, +0x18, +0x14, +0x2c, +0x30) |
| +0x1c type (1 player 2 enemy 4 NPC) | 0x1c | 0x1c | P | inside the unchanged header; no use site traced |
| +0x28 -> ChrMoveParam row, +0x150 kill time | 0x28 | **0x28** (+0x150) | C | OOB check 0x835aa0 / 0x835aac (NT 0x62f4f4 / 0x62f500) |
| +0x60 forward row | 0x60 | **0x60** | C | vtable getter slot +0x124 0x78de58 (NT slot +0x10c 0x5a17a0) |
| +0x70 position | 0x70 | **0x70** | C | position getter slot +0xa4 0x78de40 (NT +0xa0 0x5a1788); OOB check calls it via vtable +0xa4 (NT +0xa0) |
| +0xc0 name (UTF-16 SSO; len +0x10, cap +0x14) | 0xc0 | **0xcc** (len 0xdc, cap 0xe0) | C | name getter slot +0x60 0x78e3d8 `addi 0xcc` (NT +0x5c `addi 0xc0`); ctor 0x791ca0 |
| HP | 0xf0 | **0xfc** | C | GetHP slot +0xdc `lwz 0xfc` (NT +0xcc `lwz 0xf0`); IsAlive slot +0xd8 (NT +0xc8); PC vanilla CT also +0xFC |
| min HP (1 = cannot die) | 0xf4 | **0x100** | P | ctor zeroes the 4-word block 0xfc-0x108 exactly where NT zeroes 0xf0-0xfc (0x791d08.. vs 0x5a5448..) |
| max HP | 0xf8 | **0x104** | P | same block |
| HP cap (current max) | 0xfc | **? (not 0x108)** | ? | same block, and retail's new getter slot +0xe0 does `lwz 0x108`, but the first live read had +0x108 = 827 with max HP 826, so it isn't the cap (corrected 2026-11-26) |
| stamina cur / min / max (float) | 0x17c / 0x180 / 0x184 | **0x140 / 0x144 / 0x148** | P (+0x140 C-) | ctor float triples start 0x140 (NT 0x17c); 0x7a344c `lfs f1,0x140(chr)` = current stamina vs cost (retail-only roll penalty); PC vanilla CT writes +0x140 |
| posture triple | 0x188 | **0x14c** | P | 2nd float triple in ctor (PC CT also writes +0x14C) |
| build-ups (7 triples, 0xc apart) | 0x194.. | **0x158..0x1a0** | P | triples 3-9 in ctor |
| poise triple | 0x1e8 / 0x1ec / 0x1f0 | **0x1ac / 0x1b0 / 0x1b4** | P | 10th float triple in ctor |
| S = state object | [chr+0x84] | **[chr+0x94]** | C | OOB check 0x8359e8 / 0x835ac8 (NT 0x62f438 / 0x62f538); [roll i-frames and Agility](../mechanics/agility-iframes.md) |
| S i-frame counter | +0x3dc | **+0x5a0** | C | TAE-state enum fn 0x789b84: 0x78a1e4 `addi 0x5a0` (NT 0x59e0b8: 0x59e744 `addi 0x3dc`) |
| S super armour | +0x3d4 | **+0x598** | C | same fn: 0x78a220 (NT 0x59e780) |
| S ultra armour | +0x3d8 | **+0x59c** | C | same fn: 0x78a190 (NT 0x59e6f0) |
| S OOB skip counter | +0x3b4 | **+0x56c** | C | same fn 0x78a1cc (NT 0x59e72c) and OOB check 0x835ad4 (NT 0x62f544) |
| S fall-death sink flag | u16 S+0x2b2 bit 0x0001 | **u16 S+0x478 bit 0x1000** (byte S+0x478 bit 0x10) | C | OOB check 0x835c18 `ori 0x1000` / clear 0x835d78 (NT 0x62f670 `ori 1`) |
| S roll ratio | +0x90 | +0xa4 | C | [Roll i-frames and Agility](../mechanics/agility-iframes.md) |
| pparam | [chr+0x368] | **[chr+0x378]** | C | souls code 0x6610f4, [roll i-frames and Agility](../mechanics/agility-iframes.md) |
| pparam base attributes (11 x u16) | +0x04 | **+0x04** (effective copy +0x1a) | C | [Roll i-frames and Agility](../mechanics/agility-iframes.md) (FUN_008626b8 reads p+0x00..0x14, p = pparam+4, writes p+0x16) |
| attribute order | VGR END VIT ATN STR DEX INT FTH RES AGL ? | **VGR END VIT ATN STR DEX INT FTH ADP ? ?** | ATN, ADP C; rest P | ATN = index 3 (+0x0a), ADP = index 8 (+0x14): Agility row code ([roll i-frames and Agility](../mechanics/agility-iframes.md)); the rest = SOTFS PlayerParam order (VGR..ADP at SOTFS +8..+0x18) |
| retail stat offsets | | VGR +0x04, END +0x06, VIT +0x08, ATN +0x0a, STR +0x0c, DEX +0x0e, INT +0x10, FTH +0x12, ADP +0x14, unused +0x16 / +0x18 | P | as above |
| soul level | +0xb8 | **+0xcc** | P | twin script-value switch: NT 0x5029d0 (SL, p+0xb4) -> retail 0x6b1b68 (`lwz 0xcc`), retail adds a +0xe8 case before the twin of NT 0x5185b4 (= 0x6ce0f8); = SOTFS +0xD0 - 4 |
| souls | +0xe4 | **+0xe8** (lost-on-death flag byte +0xec) | C | add souls 0x6610f4 (cap 999,999,999, same r31+0x24 amount / stb 0x20 / li r4,8 as NT 0x4c69ac); death clear 0x661268 |
| P (player data) | [chr+0x2e4] | **[chr+0x2d4]** | C | vtable getter slots +0x8c / +0x90 `lwz 0x2d4` (NT +0x88 / +0x8c `lwz 0x2e4`), neighbours (+0x84/+0x88/+0x94/+0x98) shift identically |
| evasion / fall-height object | [chr+0x2f4] | **[chr+0x2e4]** | C | OOB check 0x8359cc (NT 0x62f41c) |
| fall-death height | [chr+0x2f4] vt +0x3c | **[chr+0x2e4] vt +0x40** | C | 0x8361a4 (NT 0x62fc10); companion call vt +0x4c -> **+0x50** (0x835f9c / NT 0x62fa54) |
| component C | [chr+0x98] (= chr+0x500 on the demo heap) | **[chr+0xa8]** | C | ctor 0x794a0c/0x794a14 (NT 0x5a776c/0x5a7774); C vtable retail 0x01c28d88 (NT 0x019b1fd8) |
| **OOB check object O** | [chr+0x588] = [[chr+0x98]+0x88] | **[[chr+0xa8]+0x8c]** | C | C::Update 0x78cc90 (vt 0x01c28d88 +8): 0x78ce8c `lwz r3,0x8c(r31)`, call 0x78cea0 (NT 0x5a07d0: 0x5a09cc `lwz r3,0x88(r31)`, call 0x5a09e0) |
| OOB check function | FUN_0062f398 (0x62f398-0x62fcf4) | **FUN_00835950** (0x835950-0x83637c) | C | `--multi` vote + read side by side |
| O internals: +0 chr, +4 latched, +5/+6 fall-death, +7/+8, +9 pending, +0xc kill timer, +0x10/+0x14, +0x18 second timer, +0x20 reference position | | **unchanged**; new +0x30 (copy of +0x20, 0x836190) | C | e.g. timer 0x835a90/0x835a98, flag 0x8359e0, position 0x835b3c |
| kill decision | inline (timer >= row+0x150 or S+0x2b0 bit 4) | timer >= [[chr+0x28]+0x150] **or u16 S+0x476 bit 3** -> **FUN_008357c8(O)** (0x835d68) | C | 0x835aac-0x835abc |
| [chr+0x88]+0x34 bit 0x2 | | **[chr+0x98]+0x30 bit 0x2** | C | 0x835cf4 (NT 0x62f6f8) |
| [chr+0x8c]+0xb6 bit 0 (position override) | | **[chr+0x9c]+0xba bit 0** | C | 0x835b6c (NT 0x62f5dc) |
| S counter raised when the flag latches | S+0x400 | **S+0x5c8** | C | 0x835a30 (NT 0x62f480) |
| other S fields in the check | +0x52c byte, +0x7c, +0x78 | +0x77c, +0x8c, (+0x78 test replaced by **byte S+0x235**) | C | 0x835a58 / 0x835a68 / 0x835a84 |
| movement M | [[chr+0xa8]+8] | **[[chr+0xb8]+8]** | C | getter 0x78dfcc (NT 0x5a1900), called at OOB entry 0x8359d0 (NT 0x62f420) |
| M velocity | M+0x10 | **M+0x20** | C | OOB 0x8359fc `lvx v31,M,0x20` (NT 0x62f450 `lvx v31,M,0x10`) |
| on-ground test | FUN_0062a874 | **FUN_0082fbb4** | C | same call position in the check (0x835a70 / NT 0x62f4bc) |
| on-ground byte read by the check | [[M+8]+0x191] | **M+0xb4**, unless M+0xf0 != 0 or u16 [[chr+0x98]+0xee] bit 0x4 (then 1) | C | 0x82fbe0-0x82fc08; M+0xb4 = 1 or M+0xb5 (0x8304b0-0x8304d0, 0x8307ec) |
| raw contact byte (the NT byte's twin) | PC+0x191 | **[[M+8]+0x1a1]** = [[[[chr+0xb8]+8]+8]+0x1a1] | C | writer 0xca6edc stb 0x1a1 (NT 0xa980ec stb 0x191); read into M+0xb5 by 0x82e0bc |
| physics controller PC | [M+8] (chr+0x5d90 on the demo heap), vt 0x019bf820 | **[M+8], vt 0x01c362b8** | C | 0x82e0b8 `lwz r10,8(M)`; vtable slots 0/1/2 = 0xca8230 / 0xca87f0 / 0xca7e54 (NT 0xa993c0 / 0xa999a0 / 0xa99000) |
| **Havok wrapper W** (character rigid body) | [chr+0x5f24] = [PC+0x194], vt 0x019e60a0 | **[PC+0x1a4]** = [[[[chr+0xb8]+8]+8]+0x1a4], **vt 0x01c5c3e0** | C | PC vt+8 GetEntity 0xca7e60 `lwz r3,0x1a4(r3)` (NT 0xa9900c `0x194`); W init 0xca7e04 (NT 0xa98fb0); W methods match NT apart from TOC (0x1413064 = 0x11fbda4 ...) |
| **entity E** = [W+0x10] | vt 0x019e19a8 | **[W+0x10], vt 0x01c57ce8** | C | GetEntity helper 0xd91dd8 `lwz r3,0x10(r3)` (NT 0xb6e280); vtable via slot-0 twin 0x130c7a4 (NT 0x10f5414) |
| Havok world = [E+0x08], gravity +0x10 (y +0x14) | vt 0x019e1eb0 | **vt 0x01c581f0**, offsets unchanged | C | slot-0 twin 0x1324668 (NT 0x110d2d8) |
| E+0x18 transform (+0x30 / +0x40 / +0x50), E+0x2c filter, E+0x1b0 / +0x1c0 velocities | | **unchanged** | C (library) | Havok code differs only in TOC offsets: 0x130c7a4, 0x1324668, 0x13368d4, 0x136e05c, 0x1413064... vs NT (`--show`) |
| capsule layer values (1 normal, 39 no-clip) | | not checked | ? | game data / filter table; read live |
| [floor collision](port-map-draw.md#1-table) chr+0x35c | | skipped (other agent) | | retail ctor zeroes chr+0x368, the likely twin |

## 2. Ready-made retail chains (chr = [App+0x60])

```
pparam        [chr+0x378]          souls +0xe8, SL +0xcc (P), attributes u16 from +0x04
S             [chr+0x94]           i-frames +0x5a0, super armour +0x598, ultra armour +0x59c, OOB skip +0x56c
P             [chr+0x2d4]
movparam      [chr+0x28]           OOB kill time +0x150 (7.0 s retail)
oobchk        [[chr+0xa8]+0x8c]    flag +4, timer +0xc, reference position +0x20
M             [[chr+0xb8]+8]       velocity +0x20, on-ground (filtered) byte +0xb4
PC            [[[chr+0xb8]+8]+8]   contact byte +0x1a1, W at +0x1a4
pentity       [[[[[chr+0xb8]+8]+8]+0x1a4]+0x10]     filter info +0x2c
pbody         [pentity+0x18]       position +0x30 (teleport +0x30/+0x40/+0x50)
world         [pentity+0x08]       gravity y +0x14
```

## 3. Retail OOB check: behaviour changes worth knowing for Ghost / teleport

- The kill is no longer inline: when the timer reaches `[[chr+0x28]+0x150]` (or S+0x476 bit 3 is set) the check calls
  `FUN_008357c8(O)`, which (if byte `[[chr+0xac]+0x14]` is set, byte chr+0x13c = 0, `[[chr+0x9c]+0xc4]` = 0, u16
  S+0x478 bit 14 clear and S+0x5ac = 0) sets u16 S+0x470 |= 0x200 once and fills a death request at S+0x709..+0x71d
  (0x5a = 90 at +0x718). Raising ChrMoveParam +0x150 still disables it (the compare is at 0x835ab4).
- New gate: byte S+0x235 != 0 takes the "on ground" branch instead of counting (0x835a84) and blocks latching
  (0x835cc8); it sits where the NT tested S+0x78.
- The timer is copied to S+0x73c every frame (0x835c28).
- On-ground is no longer the raw contact byte: it is M+0xb4 (filtered: 1 also when 0x82fcd0(M) holds), or forced 1 by
  M+0xf0 / [[chr+0x98]+0xee] bit 0x4. For the Ghost landing use either M+0xb4 or the raw [[M+8]+0x1a1] and test live.

## 4. Open

- min HP / max / cap, stamina min/max, posture, build-ups, poise: layout-only (ctor order + PC vanilla); confirm with a
  [live read](port-live-check.md) (HP 0xfc next to 0x100 = -1, 0x104 = max).
- Attribute order other than ATN and ADP, SL +0xcc: from SOTFS and a matched switch, not from a use that names them.
- The retail chr type at +0x1c, layer numbers, and whether retail heap neighbours put W at a fixed chr distance: live.
- Scratch tools used (session scratchpad\agentB): `vtshort.py` (vtable getters side by side), `pairat.py` (difflib
  alignment of an NT instruction with its retail partner), `chain.py`, `pairld.py`, `ldcmp.py`.
