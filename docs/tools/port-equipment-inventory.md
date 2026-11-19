# Retail v1.00 equipment, inventory and enemy-swap layouts for the cheat table (2026-10-17)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Follow-up to [port: equipment, AI freeze and enemy swap](port-equip-ai-spawn.md) (sections 1, 2 and 4), which left the inventory node layout, the mirror, what an
equipment record must hold, the 14-entry group, the generator chain and the load-wait release open. All answered here
from the retail EBOOT (Ghidra `DefineAndDecomp` on DS2V and DS2NT_mac, capstone, `tools\xmatch.py`) and retail params,
then **read live (read only) on the author's Warrior the same day: section 5**, which corrected one thing (the lists).
**C** = matched code in retail (with a Network Test twin where noted), **P** = probable, **L** = seen live.

Used by `..\..\..\DS2 Network Test\tools\ct_equip.lua` / `ct_enemy.lua` through `ctprofile.py` ("vanilla"), the data
from `..\..\tools\ct_items_vanilla.py`, the entries in `..\..\cheat-tables\DS2V.CT` (14-16).

## 1. The open questions, answered

| question | answer | st | evidence |
|---|---|---|---|
| node field order and size | **0x20 bytes: +0 prev, +4 next, +8 byte (saved in the mirror; probably the "new item" mark), +0xc item id, +0x10 ItemParam word 0, u16 +0x14 index, u16 +0x16 flags (high byte = kind = ItemParam +0x4f; 0x80 = stack limit > 1; 0x40 equipped; 0x20 saved as mirror bit 23), +0x18..+0x1f tail** (the Network Test's 8-byte tail, 4 bytes later) | C | store reset 0x533280 (free-list loop writes u16 +0x14 = index at stride 0x20; NT 0x3f26a8 +0x10 at 0x1c); node identity 0x52c468 / 0x52beb0 (NT 0x3eb888 / 0x3eb6f0: id +8, word 0 +0xc, kind +0x12); list find 0x52c010 compares +0xc with the id; delete 0x536e88 (count u16 +0x18, kind byte +0x16); 0x52bc74 `lhz 0x14` index |
| mirror entry | **16 bytes per node index at X+0x18** (0xf00 entries; X+0x18+0xf000 = quick-item index byte, +0xf001 spell index byte, then the second store's entries, then the saved table at +0x10004): +0 id, +4 byte = node +8, bit 23 = flag 0x20, +8.. by list class: weapons / armour / rings f32 durability (+8), upgrade +0xc, infusion +0xd; ammo / goods u16 count (+8; the Estus Flask also +0xa..+0xc); spells casts byte (+8) | C | quick-item index writer 0x52f598 `[S+0x1e0c8]+0xf000` (NT 0x3ed7c8 `+0x5400` = 0x700 x 12); loaders 0x52caac / 0x52d178 / 0x52efd0, writers 0x52c390 / 0x52c468, 0x52bdb0, 0x52be0c, 0x52bd64, 0x52be58 (clear) |
| what record +0 / +4 must hold | record = **+0 u32 0, +4 the PARAM id (WeaponParam / ArmorParam / RingParam / ArrowParam / SpellParam id; goods: the item id), +8 category byte, +0xa leftover bytes, +0xc u16 count, +0x10 f32 durability (-1 = none)**; the same ids the demo's slots hold | C, L (the worn armour reads 17440100-03 in E, 27440100-03 = item ids in the nodes) | builder 0x7ec96c / 0x7ec8b0 -> 0x49a1ec (`+0 = 0, +4 = id, +8 = category`); ammo writer 0x532c48 passes ItemParam +0x1c (ArrowParam), quick items 0x530338 the item id; NT armour writer 0x3ee4ac passes ItemParam +0x1c (ArmorParam), NT spells 0x3f1790 +0x28 (SpellParam) |
| category byte | from a 52-entry slot descriptor table at **0x1d96880** (bss, filled by the static initialiser 0x49a484): +0 name, +4 group, +5 category, +8 index, +0xc pointer-table index. Category **0 weapon, 1 armour, 2 ring, 3 ammo, 4 goods, 5 spell, 6 gesture, 0xff none**; the CT writes it when a record still reads 0xff | C | `straight.py` interpretation of 0x49a484 (scratch) |
| the 14-entry group | **the attunement slots**: the EBOOT names its slots `L_hand1`..`R_hand3`, `Helmet`, `Armor`, `Guntlet`, `Leggings`, `Quiver0/1`, `Arrow1/2`, `Bolt1/2`, `Ring[0-3]`, `Item[0-9]`, **`Spell[0]`..`Spell[13]`**, `Gesture[0-7]` (string table 0x17f3a88, the descriptor's +0). Slot order 0-5 hands, 6-9 armour, 10-11 quivers, 12-15 ammo, 16-19 rings, 20-29 items, 30-43 spells, 44-51 gestures | C | descriptor table above; the demo has no such names |
| generator chain | **not visible in code**: the demo's `[[[[App+0x20]+0x34]+0x34]+0x38]` came from a live heap search, and no NT code site walks it (every App+0x20 reader goes through accessors). Retail finds the objects by a heap scan for the object vtable **0x01c2d940** and takes the manager at object +0x38 (array +0x10, count +0x14) | object vt, +0x38 C; manager +0x10 / +0x14 P | `appuse.py` / `chainscan.py` (scratch): 31 NT functions read App+0x20, none continues +0x34 |
| retail load-wait release | **[[App+0x0c]+0x418]+0xc** (pending > 1 = still waiting; writing 1 releases), during **App state 21**: the App state table (retail 0x1c1f6b0, NT 0x19ac2b0) has the same handlers at the same indices (3 = App init 0x54d7c0, 21 = 0x544d08 matched strictly to NT 0x3fcf9c, 28 = 0x54b1a0, 30 = in game) | C | check 0x7fe824 `lwz 0x418; lwz 0xc; cmplwi 1` (NT 0x5fee18 +0x3b0 / +8) |

## 2. Inventory, in full

- **X = [[[App+0x54]+8]+8]** (0x100e8 bytes, ctor 0x520198, no vtable): +0 manager, +4 item-param holder, +8 store S
  (0x1e108, ctor 0x5337b8), +0xc second store (0x2050; saved positions 42-49 go there as NT 36-43 do, so probably the
  eight gesture slots; the CT never writes them), +0x10 (0x16c4), +0x14 (0x10, new), +0x18 mirror, +0x1001c saved u16
  table (50 entries), +0x100d4.. counters.
- **Store S** (vtable **0x01c1f4d8**, 21 slots; the demo's has 33, retail calls several of them directly): +8 u16 free
  count, +0xc allocator, +0x10 holder, **pool +0x14** (0xf00 x 0x20), last node 0xeff at +0x1dff4, **+0x1e014 free-list
  head pointer** (the list is circular through every free node), **pointer table +0x1e018** (42 u32: weapons 0-5 in
  R1 L1 R2 L2 R3 L3 order, armour 6-9, rings 10-13, ammo 14-17, items 18-27, spells 28-41), +0x1e0c0 last node freed,
  **+0x1e0c8 = X+0x18, +0x1e0cc = X+0x1001c** (the store's copies, a good sanity check), list group pointers +0x1e0d0..
  +0x1e0e8, **item counters u16 +0x1e0ec / limit +0x1e0ee (0x6fc)**, +0x1e0f0 / +0x1e0f2 (4), +0x1e0f4 / +0x1e0f6
  (0x800), quick-item / spell selection bytes +0x1e104 / +0x1e105.
- **Lists** (base init 0x52bc38): +4 holder, +8 head, +0xc tail, +0x10 pointer-table section, +0x14 mirror, u16 code
  +0x18, u16 count +0x1a, byte slot count +0x1c, byte +0x1d. Groups (pointer in S, count x stride, class vtable,
  section): A +0x1e0d0 15 x 0x2c vt 0x01c1f3d8 (weapons, section +0x1e018), B +0x1e0d4 4 x 0x2c (armour, +0x1e030), C
  +0x1e0d8 1 x 0x2c (rings, +0x1e040), D +0x1e0dc 1 x 0x30 vt 0x01c1f418 (ammo, +0x1e050), E +0x1e0e0 2 x 0x30 (goods,
  +0x1e060), F +0x1e0e4 1 x 0x28 vt 0x01c1f498 (spells, +0x1e088), G +0x1e0e8 1 x 0x24 vt 0x01c1f458 (new; no section).
  Each list's code and index come from **MenuCategoryIconParam** (row id = code, byte 0 = index in its group; the
  store reset 0x533280 walks it): the codes are the demo's: 10-150 weapon categories, 400-430 armour, 500 rings, 600
  ammo, 700 / 720 goods, 800 spells, plus 2000 (group G). Group predicates 0x53ec98.. (code < 399, 400-498, ...).
  ([port: equipment, AI freeze and enemy swap](port-equip-ai-spawn.md)'s "0xd2 / 0xdc / 0xe6 / 0xf0 / 0xfa" are some other code, not list codes; ItemTypeParam rows
  carry such values at +0x10 / +0x14.)
  **A list holds every owned item of its menu category, equipped or not (L)**: the Warrior's worn broadsword is in list
  20, the Iron Parma in 150, each armour list holds the worn piece next to the carried starting piece. So 0x8c / 0x96
  are not "the hands' lists" (the demo's tools assumed that from its own character's two weapons). Which list an item
  joins (the per-category functions in table 0x1c1f530, by ItemTypeParam (+0x40) category +0x18): 3 / 4 weapons and
  shields -> **WeaponTypeParam +0x130** (0x531308; 0 for fists and placeholders), 5 armour -> by ArmorParam part,
  6 rings 0x1f4, 7 spells 0x320, 8 goods 0x2bc, 9 ammo 0x258, 2 (ItemTypeParam 1200, the 17 61xxxxxx goods) 0x2d0;
  category 1 (key items, kind 10) has no function here: the second store keeps them.
- **Allocator 0x53abe0**: node = [S+0x1e014]; refused when the counter (+0x1e0ec, or +0x1e0f0 for key byte 2) is at its
  limit; head = node.next (0 if alone); unlink; free count -1; counter +1; node zeroed except its index; then the
  list's +0xc method fills it (weapons 0x52cc50: tail +0x18 durability clamped to WeaponParam +0x28 / ArmorParam +0x38 /
  RingParam +0x04 by 0x52c954, +0x1c = 0xff at full durability (0x525720), +0x1d / +0x1e upgrade / infusion; goods and
  ammo 0x52e708: count clamped to ItemParam +0x4a, +0x1a = 3 for type category 1 else ItemUsageParam(+0x44) +4 bit 21
  inverted, +0x1c bits 31 / 30 / 29 for ItemParam +0x40 = 0x44c / 0x46a / 0x488; spells 0x52f198: casts from the
  character's attunement (0x525d04), slots = SpellParam +0xf0), and 0x52c468 links it at the list tail and writes the
  mirror's id. Deleting (0x536e88) reverses it and puts the node back before the head.

What the CT writes for "Make permanent" (ct_equip.lua, K from the profile, `LIST_BY_ITEM`): the allocator's steps above,
node id / word 0 (the item id, or `ds2ntItemW0`) / index / flags (kind << 8 | 0x80 if stackable | 0x40) / tail, the
16-byte mirror, the new node linked into **its item's own list** (weapons: the list code in `ds2ntWpnItem`'s 4th value;
goods 0x2bc or the 10th value), the pointer-table entry and the saved index. Swapping never moves nodes between
lists: the old occupant keeps its list and loses flag 0x40, a carried copy that gets equipped stays where it is. Casts
= SpellParam +0xf1 (the first level's value; the game uses the character's attunement and refreshes it). The Estus
Flask is left out (its charges come from the game's own node), key items too.

The demo's ct_equip path (unchanged, kept equivalent) unlinks the old occupant and appends every weapon to 0x8c / 0x96;
on the demo that only touched the session's menus (the loader rebuilds the lists from the mirror at the next load, and
the saves verified then). Worth fixing on the demo too, as a separate change.

## 3. Enemy swap

- Generator object vt 0x01c2d940, ctor 0x940298: +0x38 manager, +0x3c GeneratorParam row, +0x40 GeneratorRegistParam
  row (C); base class ctor 0x935030 is the demo's 0x7293f8 apart from one TOC offset, so +0x10 / +0x18 position = P.
- Per-map params (24 maps, in the archives as `param\Generator*Param_mXX_YY_00_00.param`): [GeneratorParam rows](../modding-world/generatorparam-fields.md) are 160
  bytes (demo 128) with the same first 0x80 bytes (regist id at +8); GeneratorRegistParam rows are the demo's 16 bytes
  (+0 enemy id). The table carries every map's row offsets and, per enemy, the regist row the map uses most; the map
  in memory is the one whose offsets put most objects on one base. Retail picks an **enemy id** (one dropdown row per
  map, MapName.fmg names), the demo a regist id.
- chr +0x70 position and +0x14 id unchanged (C, [port: player character](port-character.md)), +0x1c type 2 = enemy (P).

## 4. Still to check live (DS2V-README.md's first-boot list)

Only the writing paths now (section 5 read the rest): an applied slot edit (the rebuild bit and the category byte
rule), a made-permanent item through save and reload (allocation from the head pointer, the counters, the mirror per
class, the new node's list), the enemy swap through a rest (chr +0x1c = 2 for the spawned enemy), and the "Loading..."
release (the count reads 14-18 in game, so it is not "1 when idle"; the table writes it only in state 21 after 8 s).

## 5. Read live (2026-10-17, read only; Warrior, SL 12, Things Betwixt bonfire)

`vmem.py` reads (scratch `equiplive.py`), then the table's own Lua lookups run against the live process with every
guest write refused (scratch `livelua.py`: ctmock's Run over a ReadProcessMemory memory image; no entry ticked).

| item | predicted | live |
|---|---|---|
| E = [[chr+0x2d4]+0x10] | records of 0x14 from E+4 | 0x32727fe0; L1 Iron Parma 11020000, R1 broadsword 1200000, L2-R3 fists, armour 17440100-03 (ArmorParam ids), quivers id 0 / category ff, empty ammo / rings / items / spells -1, item 1 Lifegem 60010000 count 10, gestures 10 20 30 60 240 280 80 90 |
| record +0 / +8 / +0xc / +0x10 | 0 / category / count / ? | 0 everywhere / the descriptor's categories (0 0 .. 1 1 1 1, ff ff, 3 3 3 3, 2 2 2 2, 4.., 5.., 6..) / 1 for worn gear, 10 for the Lifegem / f32 durability (broadsword 40, parma 80, armour 65), -1 when empty |
| E+0x4ec | bit 31 = rebuild pending | 0 |
| X, store S | vt 0x01c1f4d8, [S+0x1e0c8] = X+0x18, [S+0x1e0cc] = X+0x1001c | all as predicted; holder [S+0x10] = [X+4] |
| free list / counters | head pointer, free count, items / 0x6fc | head = node 14, free count 3826, items 14 / 1788, others 0 / 4 and 0 / 2048 |
| lists | 25 in seven groups, demo codes | all 25 (codes 10-150, 400-430, 500, 600, 700, 720, 800, 2000), every [list+4] = holder, every [list+0x14] = X+0x18 |
| equipped nodes | id +0xc, word 0 +0x10, index +0x14, flags +0x16, tail +0x18 | all 7: word 0 = id, byte +8 = 0, flags 0x0040 / 0x0140 / 0x0240-0x0540 / 0x08c0, tails `42200000ff000000` (broadsword), `000a000000000000` (Lifegem) |
| mirror | id, 0, class bytes | e.g. `00124f80 00000000 42200000 00000000`, Lifegem `0393ae10 00000000 000a0000 00000000` |
| pointer table / saved table | agree | 0 mismatches; ct_equip's own `ds2ntInvCheck` says "tables agree" |
| lists' members | (hands' lists?) | 20: broadsword; 150: parma; 400-430: worn + carried piece each; 700: Darksign, two soapstones, Lifegem |
| generator scan | vt 0x01c2d940 | 46 objects in 0.2 s, all naming one manager whose array (+0x10, count +0x14 = 46) holds exactly them; positions +0x10 / +0x18 next to the bonfire; GeneratorParam +8 = regist ids 705xxxxx, regist row +0 = enemy ids |
| map vote | the map whose offsets fit | ct_enemy's own vote: m10_02_00_00 (Things Betwixt), GeneratorRegistParam copy at 0x35870d40 |
| load wait [[App+0x0c]+0x418]+0xc | 1 when idle? | 14, later 18, in App state 30 |

Scratch scanners (session scratchpad, not kept): `straight.py` (straight-line PPC interpreter for static initialisers),
`slotdec.py`, `tochop.py` (two-level TOC loads), `appuse.py` (code reaching App through the TOC idiom, then an offset).

## Related pages

- [Port: first live read](port-live-check.md): the same boot's first live read (root, character, Havok, map and AI values); it left the equipment and inventory layout unread, which section 5 here covers.
- [Cheat table port](cheat-table-port.md): the parent note; its section 4 records how these layouts went into the retail table.
