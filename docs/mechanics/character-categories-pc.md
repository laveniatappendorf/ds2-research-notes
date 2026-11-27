# Character categories: what sets `chr + 0x54` (PC Scholar 1.0.3.0, 2026-11-27)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

**Question.** Several notes ([Brightbug](brightbug.md), [Broken status](../network-test/broken-status.md), and a note
on the Ivory Warrior Ring that isn't published yet) rest on a category byte at `chr + 0x54` and a 17-row table at
`0x1410bfff0` whose byte 3 is set only on categories 1 and 2. They guessed that 1 and 2 are "the player", and that
enemies, NPC invaders and bosses are other categories. What does the code say?

**Short answer (code, PC).** The category is chosen when the character is created, by the function that creates it,
and written once. Each creator also names the character, and those names are in the exe, so most rows now have a
developer label:

- **1 = your own character** ("Player_"), **2 = another real player's character in your world** ("NetworkPlayer_").
  Byte 3 is set on exactly these two, so it means "a real player's character", on its own game (1) or on someone
  else's (2).
- **3 = another player's ghost** ("GhostPlayer_"). Not a real player for byte 3.
- **NPC phantoms (NPC invaders and summons) = 5** on the player body (the Copse Forlorn, read live as 5) **or 12**
  on any other body ("NpcPhantom_").
- **Ordinary enemies = 14** ("c%d"; read live). **"Boss" = 15**, but only for rows whose EnemyParam +0x18 is 3, which
  isn't every boss (section 5). NPCs (EnemyParam +0x18 = 4) are 7 / 8 or 10 / 11, "Dummy" rows 9, other player-bodied
  characters 6, network previews 4 / 13, cutscene characters 16.
- An earlier reading, "1, switched to 2 for a while by event 240000", was wrong: those writes go to a different
  object (section 4). No code found changes a character's category after it's created.

Only 1, 5 and 14 have been read live. Category 2 for other players is from the code and hasn't been seen.

## 1. The byte, its setter and its getter

| address | what |
|---|---|
| `0x14014eca0` | getter: `out = chr ? (s8) byte [chr+0x54] : 0` |
| `0x140315a10` | `CharacterCtrlBase` constructor: writes `+0x54 = 0` (and `+0x50` = an id word, `+0x55 &= 0xE0`) |
| `0x1403161b0` | **setter**: `mov byte [rcx+0x54], dl; ret`. A two-instruction leaf with no `.pdata` entry, which is why earlier function-by-function scans didn't see it |

The setter has two callers in the exe (a raw scan of both `.text` sections for `call` / `jmp` to it):

| caller | value |
|---|---|
| `0x140312e07` in `CharacterCtrl` vtable +0x208 (`0x140312dd0`, the character's init from a creation record) | `byte [record + 0xAD]` |
| `0x140323b64` in `0x140323b00`, the `DemoCharacterCtrl` init (called right after its constructor by the two cutscene-character creators) | **16** |

`PlayerCtrl` derives from `CharacterCtrl` (its constructor `0x14037ebe0` calls `0x1403114f0`), so every player-type
character also gets its category from `record + 0xAD`.

**Search behind "written once".** Every `.pdata` function (95,445 entries) disassembled for byte stores to
`[reg+0x54]`; a raw byte scan of both `.text` sections for `88` / `C6` stores with displacement 0x54, plain and SIB
forms; each hit checked against the object it writes. Apart from the setter and the base constructor, none writes a
character. Not covered: 16/32/64-bit stores that overlap +0x54, stores through a computed address, and the anti-tamper
part of the second `.text`, which doesn't decode from the file.

## 2. Who fills `record + 0xAD`

All four functions that fill a creation record (the callers of the record initialiser `0x140354910`) set +0xAD, and
each passes a UTF-16 name format to `0x140003690`. The names below are those strings, quoted from the exe.

### 2.1 Players: `0x140357920` ("Player_%06u") and `0x1403572e0` ("NetworkPlayer_%06u" / "GhostPlayer_%06u")

Both build a `PlayerCtrl` and copy the category from their input record's +0x29 (`0x140357dde`, `0x140357773`). The
callers set +0x29 to a constant:

| caller | +0x29 | other fields |
|---|---|---|
| `0x1400ee420` → `0x140357920` | **1** (`0x1400ee50a`) | record +0x20 (the character id) = 100 |
| `0x140419610` → `0x1403578e0` → `0x140357920` | **1** (`0x140419845`) | |
| `0x14051ce20` → `0x1403572a0` → `0x1403572e0` | **2** (`0x14051cf2a`) | id = 100; +0x2A = byte [member+0x40], the member's kind; reads a session member's data (+0x630, position at +0x30, +0x21E, +0x278-0x284) and stores the new character back in the member slot |
| `0x1401a1650` → `0x1403572a0` → `0x1403572e0` | **3** (`0x1401a171b`) | +0x2A = 0x12 or 0x13 |

`0x1403572e0` picks the name from the category table: "GhostPlayer_%06u" when the category's byte 1 is 2, else
"NetworkPlayer_%06u" (`0x1403574f5`). So 2 is built as a NetworkPlayer and 3 as a GhostPlayer.

`0x14051ce20` sits with the session object at GameManager +0x22F0 (constructor `0x140512f30`; its member list is the
five 0x4C-byte entries that `0x140253480` / `0x1402539d0` walk). Its owning class isn't named yet; "another player's
character" rests on the NetworkPlayer name and the session-member data it reads.

### 2.2 Map characters: `0x1403560a0`

Every character spawned from map data goes through here (it's the only creator besides the preview one that builds a
`CharacterCtrl`). Inputs: the EnemyParam row for the spawn's id (`0x14003e8f0` on the manager's table), and a kind byte
at spawn record +0x40 that indexes a second table, 20 rows x 16 bytes at `0x1410c0050` (rows 1-17 have byte 1 set).

- **Player body** when the EnemyParam row's +0x04 is 100 (`0x140356161`); a `PlayerCtrl` is built. (+0x04 is the row's
  own id on most rows, 100 on 223 player-bodied rows, the Forlorn's 694003 among them.) Then, in order:
  - kind byte's table row has byte 1 set → "NpcPhantom_c%06d", **5** (`0x140356427`)
  - EnemyParam +0x18 = 4 → "Npc_c%06d", **7**, or **8** when the session check below passes (`0x140356453`)
  - otherwise → "c%06d", **6** (`0x140356464`)
- **Any other body**, a `CharacterCtrl`:
  - EnemyParam +0x18 = 0 → "Dummy_c%d", **9** (`0x1403565bc`)
  - +0x18 = 3 → "Boss_c%d", **15** (`0x1403566ce`)
  - kind byte → phantom (as above) → "NpcPhantom_c%d", **12** (`0x1403567a3`)
  - +0x18 = 4 → "NPC_c%d", **10**, or **11** with the session check (`0x1403567c8`)
  - otherwise → "c%d", **14** (`0x1403568df`)

The session check is `0x140513440` on the GameManager +0x22F0 object: its +0x38 flag, or at least one active entry in
its five-member list whose kind counts as a phantom in the `0x1410c0050` table, and GameManager +0x24B1 bit 6. So 8 and
11 are likely "an NPC created while another player is in the world". That reading is a guess.

### 2.3 Previews: `0x140355930` ("NetPreview_%06u")

Category **13**, or **4** when the id is 100 or the EnemyParam row's +0x04 is 100 (`0x140355a69`). What these preview
characters are for isn't traced (the exe also has `NetSvrPreviewChrManager`).

### 2.4 The whole table

| cat | who (code) | name (exe) | class built | bytes 0-4 | live |
|---|---|---|---|---|---|
| 0 | default from the base constructor; never set by a creator | | | `00 04 01 00 01` | |
| **1** | your own character | Player_%06u | PlayerCtrl | `02 00 01 01 01` | **you** |
| **2** | another real player's character | NetworkPlayer_%06u | PlayerCtrl | `02 01 01 01 01` | |
| 3 | another player's ghost | GhostPlayer_%06u | PlayerCtrl | `02 02 00 00 00` | |
| 4 | preview, player body | NetPreview_%06u | PlayerCtrl | `02 02 00 00 00` | |
| 5 | NPC phantom, player body | NpcPhantom_c%06d | PlayerCtrl | `02 01 01 00 01` | **Copse Forlorn** |
| 6 | other player-bodied map character | c%06d | PlayerCtrl | `02 04 01 00 01` | |
| 7 | NPC, player body | Npc_c%06d | PlayerCtrl | `02 03 01 00 01` | |
| 8 | as 7, session check passed | Npc_c%06d | PlayerCtrl | `02 03 00 00 01` | |
| 9 | EnemyParam +0x18 = 0 | Dummy_c%d | CharacterCtrl | `01 04 01 00 01` | |
| 10 | NPC, other body | NPC_c%d | CharacterCtrl | `01 03 01 00 01` | |
| 11 | as 10, session check passed | NPC_c%d | CharacterCtrl | `01 03 00 00 00` | |
| 12 | NPC phantom, other body | NpcPhantom_c%d | CharacterCtrl | `01 01 01 00 01` | |
| 13 | preview, other body | NetPreview_%06u | CharacterCtrl | `01 02 00 00 00` | |
| 14 | everything else from the map | c%d | CharacterCtrl | `01 04 01 00 01` | **shield enemy 102000** |
| 15 | EnemyParam +0x18 = 3 | Boss_c%d | CharacterCtrl | `01 05 01 00 01` | |
| 16 | cutscene character | DemoChr_c%06d / DemoNpc (`0x1403552c0`) | DemoCharacterCtrl | `03 06 01 00 01` | |

The live column is from reading the byte in PC Scholar's memory (25-26 Nov).

## 3. What the table's columns look like now

These readings come from lining the bytes up with the creators above. Only byte 3 (and the uses named) is read in code.

| byte | values | reading | basis |
|---|---|---|---|
| 0 | 2 on 1-8, 1 on 9-15, 3 on 16, 0 on 0 | the class built: PlayerCtrl / CharacterCtrl / DemoCharacterCtrl | matches the creators on all 16 rows; no use read |
| 1 | 0 = 1; 1 = 2, 5, 12; 2 = 3, 4, 13; 3 = 7, 8, 10, 11; 4 = 0, 6, 9, 14; 5 = 15; 6 = 16 | a role group: local player, other players and NPC phantoms, ghosts and previews, NPCs, ordinary, boss, cutscene | grouping is a reading. Used for the GhostPlayer / NetworkPlayer name choice, and in `CharacterCtrl` init (`0x14031335e`: byte 1 = 3 and byte 2 = 0) |
| 2 | clear on 3, 4, 8, 11, 13 | not named | `CharacterCtrl` init creates a `ChrFootEffectCtrl` at chr +0x238 only when set (`0x140313d60`); its other uses aren't read |
| 3 | set on 1 and 2 only | **a real player's character** | the creators (2.1) |
| 4 | clear on 3, 4, 11, 13 | not named | not read |

## 4. Correction: the "event 240000" writes aren't the category

An earlier note (not published) said the game "only writes those [1 and 2] for the player's own character (1,
switched to 2 for a while by event 240000)", from a scan that listed constant stores to `[reg+0x54]`. The object
behind those stores was never checked. It isn't the character:

- `0x14016c4a0` is the destructor of **`ChrMorphemeTimeActTrackDeadCtrl`** (it puts that class's vtable back), and
  `0x14016c510` is its handler for **TimeAct event 240000** (0x3A980). So 240000 is a TimeAct event, not an EMEVD event
  or a script command.
- The handler writes 1 or 2 (by the event's payload byte) to `[[track+0x18]+0xE0]`, through vtable +0x70.
  `chr+0xE0` is the character's action ctrl (set in `CharacterCtrl` init from record +0x30: `PlayerActionCtrl`,
  `EnemyActionCtrl`, `BossActionCtrl`, or `CharacterActionCtrl` by default), and vtable +0x70 returns the action ctrl's
  +0xE8 in all four classes (`0x14030dac0`).
- Action ctrl +0xE8 is a **`ChrDeadActionCtrl`**, built at `0x14030dec2`. Its constructor `0x14013c210` sets its own
  +0x54 to 1, and its setter `0x14013c500` accepts only 1 or 2.
- The other two "1 or 2" writes found by the same scan reach the same field: `0x14030ec70` (from status +0x4B8 bit 28)
  and `0x14031b722` (2 bits of a record it's handed, not followed).

What that 1 / 2 means for death handling isn't traced. It assumes the track's +0x18 is the character, which is how the
same `+0xE0` → vtable +0x70 chain is used at `0x14031b703`.

Errata E-055.

## 5. Data check: EnemyParam +0x18 and +0x04

The creator's tests on EnemyParam (PC regulation, 976 rows):

| +0x18 | rows | created as (other body / player body) |
|---|---|---|
| 0 | 2 (8100, 8110) | 9 / 6 |
| 1 | 650 | 14 / 6 |
| 2 | 19 | 14 / 6 |
| 3 | 65 | 15 / 6 |
| 4 | 238 | 10-11 / 7-8 |
| 5 | 2 | 14 / 6 |

(Unless the spawn's kind byte makes it an NPC phantom, which comes first on both sides: 12 / 5.)

The rows that `bosses_pc.tsv` (the full output behind [Bosses: vanilla 1.10 vs PC Scholar](bosses-pc.md)) lists
under a fight split. Ancient Dragon 600000, Flexile Sentry 303300, Giant Lord
309700, Last Giant 309600, Nashandra 627000, Royal Rat Authority 628000, Smelter Demon 305000, The Rotten 326000 and
Throne Watcher 334000 have 3, so they're created as 15. The rows listed for the Forest Pursuer (318011), Vendrick
(514600) and Freja (203000) have 1, so they'd be created as 14. So "Boss_" follows the row's +0x18, not whether the
fight is a boss fight. That table only lists rows that changed between builds, so this isn't a full boss list, and
which character each listed row is wasn't checked here.

## 6. What this changes elsewhere

- **Ivory Warrior Ring.** Its note (not published yet) rests on the same byte 3; what this means for the ring is
  written up there.
- **PS3 send override `0x5d6ee0`**: sending the hit to "the target character's peer" when the target's byte 3 is set
  fits: a category-2 character belongs to another player.
- **Brightbug** (slots 58 / 59, "exactly one side has byte 3"): the multipliers apply between a real player and
  anything else (enemies, bosses, NPC invaders), and never between two real players. That's the PvE-only reading,
  now from code.
- **Break** (`0x140145d24`, 900300 + 1): the two flagged categories are real players' characters (yours and other
  players' on your game). Everyone else, NPC phantoms included, gets 900301.

## 7. Open

- **Category 2, live.** Reading both characters' bytes on both machines in a two-player session would show it
  directly. Expected: each machine reads its own character as 1 and the other player's as 2.
- **PS3.** The PS3 Scholar EBOOT has all the same name strings (big-endian UTF-16 around file offset `0x1866378`-
  `0x1866500`), so the same creators are very likely there. Not traced; the PS3 table at `0x1865e20` is the same as
  PC's for rows 0-16.
- The spawn record's kind byte (+0x40), and how it's filled from the generator (GeneratorParam `Unk84` is a candidate
  from the NPC invaders note), aren't traced; nor is which table row each NPC phantom kind uses.
- The meaning of the 7 / 8 and 10 / 11 split (the session check) is a guess.
- What the preview characters (4, 13) and `ChrDeadActionCtrl` +0x54 are for.

## Files

Scratch scripts for this session (capstone + pefile, in the session scratchpad, not kept): the `.pdata` scan, the
raw +0x54 scans, function dumps. The method is in section 1. Ledger rows: `pcsotfs` `1403161b0`, `140315a10`,
`1403114f0`, `14037ebe0`, `140312dd0`, `140354910`, `1403560a0`, `140355930`, `1403572e0`, `140357920`, `1400ee420`,
`140419610`, `14051ce20`, `1401a1650`, `140323b00`, `140513440`, `14013c210`, `14013c500`, `14030dce0`, and the
corrected `14016c4a0` / `14016c510` / `14014eca0`.

## Related pages

- [Brightbug](brightbug.md): slots 58 / 59 and the same table.
- [Broken status](../network-test/broken-status.md): the 900300 / 900301 rule.
- [NPCs who also invade](npc-invaders-pc.md): the NPC spirit generator.
