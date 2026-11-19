# Editing params: the encrypted regulation vs loose params (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- Most of the params a mod wants to change (shop stock and prices, level and upgrade costs, item lots, bonfire rows, menu
  lists) are **regulation params**: 205 of them, packed inside `enc_regulation.bnd.dcx`. That file is AES-encrypted and
  carries a 16-byte tag that the exe checks. An edited file with the old tag **crashes the boot** [live B3]; a plain or
  bare copy is ignored or crashes [live B3]. The per-map params (370 of them: `EventParam_<map>`, `GeneratorParam_<map>`
  and so on) sit in the game's archives and a loose copy replaces them directly, so they need none of this [live B9].
- **What works with no change to the exe: the "loose params" layout.** Replace the regulation with a plain, unencrypted
  DCX that holds only its 12 text files and 11 `SpEffect*.emevd` files (21,675 bytes), and put every param in the game's
  `Param` folder as its own file. The edits load [live B3b], and also through the legacy Mod Engine on a clean Steam copy
  [live B49]. This is the layout the community's "Load Loose Param Files" mod uses [web, 5 Nov 2026].
- The other route is to patch the tag check in the exe: two conditional jumps at +0x261231 and +0x261238 [code]. It
  works [live B3]. We used it once, in a test copy, to prove the edits themselves were sound. The mod does not ship it.
- **A field edit keeps the file layout** and works on every param [live B3c, B20e, B28]. **Adding rows rebuilds the
  file**: 24-byte row entries, sorted by id, data packed in id order. The project's writer does that for 168 of the 205
  regulation params and refuses the other 37, among them `ItemParam`, `ItemLotParam2_Other` and `PlayerLevelUpSoulsParam`
  [data].
- Two traps. **Build with Python 3.12**: 3.14 compresses the regulation differently. And **the project's param reader
  rounds floats to 4 places**, so read raw bytes to check a small value such as a price rate of 0.000214.

### Two kinds of params

| Kind | Where it lives | How many | Edited by |
|---|---|---|---|
| Per-map and per-scene params (`EventParam_<map>`, `EventLocation_<map>`, `GeneratorParam_<map>`, `MapObjectInstanceParam_<map>`, `TreasureBoxParam_<map>`, ...) | the game's archives (`param\` in the `GameData` archive), so loose copies are allowed | 370 [data] | a loose file, nothing else needed [live B9] |
| Regulation params (`ShopLineupParam`, `NpcEventParam`, `ItemLotParam2_Other`, `MapObjectBonfireParam`, `PluralSelectMenuParam`, `ReinforceCostParam`, ...) | inside `enc_regulation.bnd.dcx`, a loose file in the Steam install | 205 [data] | the regulation, or the loose-params layout below |

Steam's folder also holds one loose param, `KeyConfigParam.param`, which is a regulation param (it is also inside the
regulation, with the same bytes) [data, B3b].

Two loaders sit under this. A tool that unpacks the archives (UXM) or a loader that serves an override folder (the legacy
Mod Engine, see [Mod Engine for DS2](mod-engine-ds2.md)) makes the game read a loose copy of an archive file first. That is
what every loose edit in this project relies on: text [live B1], talk scripts [live B4], per-map params [live B9]. It does
**not** reach into the regulation: a loose copy of one regulation param beside the original regulation was ignored [live
B3, form 4].

### The regulation file

Layers, as found in Steam's 1.0.3.0 file [data, re-read 17 Nov 2026]:

| Layer | Size and layout |
|---|---|
| Whole file | 836,803 bytes: a 32-byte header, then the encrypted body |
| Header bytes 0-15 | the counter seed. SoulsFormats-based tools build the AES-CTR IV from it |
| Header bytes 16-31 | a 16-byte **tag** (see the check below). It is not a plain hash of the payload: no MD5, SHA-1 or SHA-256 of the DCX or BND4 matched it [data, B3] |
| Body | AES-128 in counter mode over a DCX (836,771 bytes). Decrypting and encrypting are the same operation |
| DCX | a 76-byte header (`DCX`, `DCS`, `DCP` + `DFLT`, `DCA`), then one zlib stream at level 9 (it starts `78 DA`). The big-endian uncompressed and compressed sizes sit at +0x1C and +0x20 of the DCX header |
| BND4 | 9,102,020 bytes uncompressed; an ASCII version string `02000200` at +0x18; 228 entries: **205 `.param`**, 12 `regulation<Language>.fmg`, 11 `SpEffect*.emevd`. Each entry header is 24 bytes: flags `0x40`, `-1`, size (u64), data offset (u32), name offset (u32). Names are Shift-JIS, there is no hash table, and each file's data starts on a 4-byte boundary |

The key is the one the community's tools already carry; we do not repeat it. Encryption is the easy half. Our writer
re-encrypts an edited regulation and the game decrypts it correctly (the four edited rows were in memory after the exe
patch, B3). The tag is the hard half.

### The tag check [code]

All `+0x...` addresses on this page are RVAs from the image base 0x140000000 (read from the PE header, 17 Nov 2026), so
Ghidra's address is `0x140000000` plus the RVA: +0x261231 is `0x140261231`.

| Address | What is there | Source |
|---|---|---|
| +0x48CB30 | getter for the file name `title:/enc_regulation.bnd.dcx`, called from +0x2DE2EF; the generic file load is +0x2DCE30 | [code, 28 Oct] |
| +0x2E0470 | the regulation loader: builds the AES key on the stack, calls +0x261150 with the key, the file's bytes, their size and an output buffer | [code, 28 Oct] |
| +0x261150 | decrypt and verify. Bytes 0-15 seed the counter; bytes 16-31 are the tag | [code, 28 Oct] |
| +0x260320 / +0x260D10 | the counter-mode decrypt | [code, 28 Oct] |
| +0x2602F0 then +0x260880 | computes a tag with a **custom MAC over 12-byte blocks**. It is not GCM | [code, 28 Oct] |
| +0x26122C to +0x261238 | compares the computed tag with the header's, as two 8-byte halves. Each compare is followed by a `jne` to the failure branch (at +0x261231 and +0x261238) | [code, re-read 17 Nov] |
| +0x26123A / +0x26123E | the success and the failure result (a flag in `bl`) | [code, re-read 17 Nov] |
| +0x265B3E | where a failed check ends: an access violation on a null pointer, about 1 s after launch. +0x275FC0 returns `[GameManagerImp+0x22F0]`, which is null. Nobody traced who sets it | [live B3] |

The compare site has the same bytes in Steam's exe and in the unpacked test copy's exe [code]. A failed check is not a
clean error: the boot goes on and dies on a null pointer a moment later [inference from the crash address].

So the file must decrypt correctly **and** carry a tag the MAC agrees with. We never reproduced the MAC, so we could not
write a valid tag [untested]. The project did not need to, because of the loose layout below.

### What the game did with each form of the regulation

All of these ran on PC 1.0.3.0 in the project's UXM-unpacked test copy, except the last row (a clean Steam copy). The test
copy also carried a debug plug-in, which we never ruled out as a cause of form 2's result [data, 5 Nov note].

| Test | Regulation file | Param folder | Exe | Result |
|---|---|---|---|---|
| B3 form 1 | edited, re-encrypted, **original 32-byte header** (stale tag) | none | stock | boot crash at +0x265B3E, three times out of three [live B3] |
| B3 form 1, retried | the same file | none | both jumps patched | boots; all four edited `ShopLineupParam` rows in memory at 0.0; Melentia's Lifegems and Lenigrast's shards cost 0 [live B3] |
| B3 form 2 | edited, **plain DCX** with all params inside | none | stock | boots; **edits ignored**: the Lifegem row is in memory with price_rate 1.0 and no 0.0 copy exists [live B3] |
| B3 form 3 | edited, **bare BND4** (9,102,020 bytes, no DCX, no encryption) | none | stock | boot crash at the same address, twice [live B3] |
| B3 form 4 | the original | one edited loose `ShopLineupParam.param` | stock | boots; **normal prices**: loose regulation params are not read [live B3] |
| B3b | **plain DCX with no params** (12 + 11 entries, 21,675 bytes) | **all 205 params loose**, edits in them | stock (no patch) | boots, loads the save; both Lifegem rows at price_rate 0.0 in memory [live B3b, 6 Nov] |
| B49 | the same plain DCX, from the shipped package | the same, in Mod Engine's override folder | Steam's own exe, legacy Mod Engine | passed on a clean install: every vendor stocked and free [live B49]. The console showed every package file served, regulation and loose params included [live B45] |

What the table says:

1. A regulation that decrypts but whose tag does not match **crashes the boot** (forms 1 and 3). Params arrive from
   inside the regulation only when that file passes.
2. A valid regulation wins over loose params (form 4).
3. A **plain DCX without params**, plus **loose params**, is the combination that works with no exe change (B3b, B49).

One reading fits every row [inference]. If the regulation does not verify as an encrypted file, the game does not take
params from it. It falls back to something we could not identify, and for any param that exists as a loose file in
`Param\` it uses the loose file. In form 2 there were no loose regulation params, so the vanilla values came from the
fallback. In B3b there were, so they won. Two things this does not explain: the archives hold no regulation params
(their index lists 370 `.param` entries, all per-map [data]), so the fallback is not the archive; and a bare BND4 crashed
(form 3) where a plain DCX did not (form 2). Both are open below.

Plain regulations are not rare in the community. Yapped, a param editor, writes the repacked regulation unencrypted [web,
code read 5 Nov 2026], and mods built on it have been shipped for years. Our form 2 was ignored in the UXM-unpacked test
copy. We never retried it on a clean Steam copy, which may behave differently [untested].

### The loose-params layout

This is the layout of the community's "Load Loose Param Files" mod (Nexus Mods, Dark Souls II, mod 1092; v1.0.1 from
October 2022; 35 mods list it as a requirement; a commenter confirmed it on a fresh Steam install in July 2026) [web, 5
Nov 2026]: a replacement `enc_regulation.bnd.dcx` with the `.param` files taken out, plus a `Param` folder holding every
param as a loose file. Its page says it does not change the game itself [web]. A mod built on it ships only the params it
changes, so mods that change different params do not collide [web]. A Smithbox export of one param as a loose file is the
same idea [web]. The Smithbox author's `DS2-Regulation-Extract-Repack` tool writes the same layout [web, code read 5 Nov].

Our package follows it: the plain regulation, then **all 204 other regulation params loose** (Steam's own loose
`KeyConfigParam.param` is left alone), then the per-map params the mod changes [data, SHIPPING 8 Nov]. That makes our
package override every other param mod [design choice]. See Open questions.

### Adding rows: the param file layout

Worked out on the regulation's params and the loose per-map ones, little-endian, as the PC build writes them [data].

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0x00 | u32 | names offset | equal to the end of the row data: data start + row size x row count |
| 0x04 | u16 | 0 | |
| 0x06 | u16 | 0, 1 or 2 | meaning unknown. On the 168 rebuildable regulation params: 0 on 120, 1 on 45, 2 on 3. Copied unchanged |
| 0x08 | u16 | paramdef version | `ShopLineupParam` 4, `ItemParam` 21 |
| 0x0A | u16 | **row count** | so a param holds at most 65,535 rows |
| 0x0C | 32 bytes | type name, NUL padded | for example `SHOP_LINEUP_PARAM`; it picks the paramdef |
| 0x2C | 4 bytes | `00 04 07 00` on every regulation param | bit `0x04` of the second byte = 24-byte row entries (the PC format) |
| 0x30 | u64 | **data start** | always 0x40 + 24 x row count |
| 0x38 | u64 | 0 | |
| 0x40 | 24 x count | row entries `{u32 id, u32 0, s64 data offset, s64 name offset}` | **sorted by id** |
| data start | row size x count | the rows, **packed in id order**, one fixed size per param | `ShopLineupParam` 36 bytes, `NpcEventParam` 48, `ItemParam` 84, `ItemLotParam2_*` 124, `PlayerLevelUpSoulsParam` 12 |
| names offset | 4 bytes on regulation params | the shared empty name | per-map params carry 4 to 16 bytes here: the empty name, then (on most) junk padding up to a 16-byte multiple |

To add a row: copy an existing row's bytes, set the fields, put the row in id order, and rewrite the names offset, the
row count, the data start and every row entry. Every row's **name offset points at the shared empty name** at the names
offset. The tail is copied through unchanged. `tools/paramrows.py` does exactly this and only this [code].

**Whether the file is the layout the writer understands is checked first.** `paramrows.py check` rebuilds a param with no
change and compares bytes. On the 205 regulation params:

| Result | Count | What they have in common |
|---|---|---|
| Rebuild byte-identical | 168 | rows sorted and packed; every row's name offset = the names offset; pad field 0 |
| Refused | 37 | rows sorted and packed, pad 0, but **every row's name offset is 0** and the file ends where the data ends. The writer's message says "own row names"; there are no names at all |

The 37 include `ItemParam`, `ItemLotParam2_Chr`, `ItemLotParam2_Other`, `ArmorParam`, `WeaponReinforceParam`,
`PlayerLevelUpSoulsParam`, `PlayerDamageParam`, `EnemyDamageParam` and `CriminalAndDeceasedParam`. Their layout looks like a
small change (keep name offset 0), but we never wrote one [untested]. A **field edit** on them works, because it changes no
layout [live B20e, B28]. The shop, `NpcEventParam`, `ReinforceCostParam`, `CustomAttrCostParam`, `PluralSelectMenuParam` and
`MapObjectBonfireParam` params rebuild.

Rows added so far, and what the game did [live]:

| Param | Rows, game vs installed | Test |
|---|---|---|
| `ShopLineupParam` | 692 to 1,824 | B28 |
| `PluralSelectMenuParam` | 3 to 56 in the v1.01 package (52 in the first live build) | B36c, B57 |
| `MapObjectBonfireParam` | 83 to 85 | B2, B54, B55 |
| `EventParam_<map>` (per-map) | one new row per arena group | B9 |

The grown `ShopLineupParam` is 109,508 bytes, which is not a multiple of 16, and the game loads it [live B28]. So no
16-byte alignment is needed on regulation params.

### Field edits: no layout change

A field edit overwrites bytes in a row and touches nothing else, so the BND4, the DCX sizes (for a regulation) and the
param layout stay as they are. `tools/regpatch.py` finds a row through the BND4's own row table. Field names, types and
offsets come from Smithbox's DS2S paramdefs, which fit all 205 regulation params and all 370 per-map params [data, 24 Oct
2026]. The editors refuse bitfields [code]. Wildcard edits (`<Param>:*:<field glob>=value`) are used for the free level,
upgrade and infusion costs; they are on [Vendor stock](../modding-menus/vendor-shops.md).

### Reading values back

`paramdef.decode`, the reader every tool here uses, **rounds `f32` fields to 4 places** [code]. A price rate of
1.5 / 7000 is stored as 0.00021428571 and reads back as 0.0002. A check that printed the decoded value would pass or fail
for the wrong reason. The edit tools (`regpatch.py set`, `looseparams.py`) compare the raw bytes with `struct` instead.
`paramrows.py add` prints its read-back through the decoder, so a small value shows rounded there [code].

### Python version

The regulation round-trip depends on the interpreter's zlib [data, checked 17 Nov 2026 on the Steam file]:

| Python | zlib | Re-compressed stream | Whole file after decrypt, edit-free re-encode |
|---|---|---|---|
| 3.12.13 | 1.3.1 | 836,695 bytes, **identical** to FromSoftware's | 836,803 bytes, **identical** |
| 3.14.3 | 1.3.1.zlib-ng | 831,775 bytes | 831,883 bytes, different. Same contents |

Stock zlib at level 9 reproduces FromSoftware's stream byte for byte, so an unchanged regulation can be rebuilt exactly.
The zlib-ng stream decodes to the same bytes but is not the same file. The release build compares every file against what
was tested, and that comparison failed on 3.14 [data, 16 Nov build notes]. All builds use 3.12. Whether the game itself accepts
a zlib-ng stream was not tested on the final file. The 28 Oct round trip was logged as "not byte-identical" and the
re-encrypted file was still accepted once the tag was patched [live B3]; that run was probably on 3.14 [inference].

## How to do it

This is the loose-params route, as `tools/looseparams.py` and `tools/release.py` build it.

1. **Get a clean reference.** Decrypt `enc_regulation.bnd.dcx` (SoulsFormats' DS2 regulation decrypt, as Smithbox does it;
   our copy is `pccrypto.regulation`) and unpack the BND4: 205 params, 12 FMGs, 11 EMEVDs. Check the source file against
   a known hash, so you know you are editing Steam's 1.0.3.0.
2. **Make the edits on the param bytes.**
   - A field edit: find the row (the BND4 row table, or the param file's own), pack the new value at the field's offset.
   - New rows: first run the no-change rebuild on that param and compare bytes (`tools/paramrows.py check`). Only add rows
     to a param that passes. Keep the rows sorted by id.
3. **Read every edit back from the staged file, as raw bytes.** Also check that every param you did not mean to touch
   equals the reference, byte for byte. `looseparams.py build` refuses a staged set where the changed params differ from
   the list it expected.
4. **Write the trimmed regulation.**
   - A BND4 that keeps only the non-param entries (12 FMG, 11 EMEVD), with the original's header, 24-byte entry headers,
     Shift-JIS names and 4-byte data alignment. Update the entry count, the data start, and each entry's data and name
     offsets.
   - Wrap it in a DCX: reuse the original 76-byte header, write the two sizes (big-endian, at +0x1C and +0x20), and use one
     zlib stream at level 9.
   - **Do not encrypt it.** Decode it again and compare with what you wrote before saving.
5. **Write every param as `Param\<Name>.param`** (Windows file names are case-insensitive; UXM writes them lower case).
   All 205, or all but Steam's identical `KeyConfigParam.param`. A partial set was never tried [untested].
6. **Install with the game closed.** Every regulation-param change in this project was installed with the game closed and
   picked up after a restart [live B20d, B20e, B28]; the installer refuses to run while the game is open [code]. A change
   with the game running was never tried [untested]. Keep the originals and a manifest of what was added, so you can undo
   exactly: `looseparams.py uninstall` removes only the files its manifest lists.
   - UXM-unpacked copy: the plain regulation replaces `Game\enc_regulation.bnd.dcx`; the params go in `Game\Param\`.
   - Legacy Mod Engine: the same two things in the override folder, nothing in the game's own files [live B49]. See
     [Mod Engine for DS2](mod-engine-ds2.md) for the install and its settings.
7. **Check the load, not just the boot.** Start the game and confirm the value the way the test needs. A shop price is the
   obvious check, but the vendor may not be reachable on your save (Melentia was in the Forest, B3b), so also read the edited
   fields in memory: `looseparams.py check` finds every in-memory copy of an edited row by its unedited bytes and prints
   the field (read-only). A match on the edited value means the loose param was read.
8. **Build on Python 3.12** (the table above).

### The exe-patch route, for the record

Used once, in the test copy, to prove that the edited contents were fine. It patches the two jumps at +0x261231 and
+0x261238 so that a correctly encrypted file is accepted whatever its tag; `tools/exepatch.py` applies and restores it, and
keeps an `.orig`. It needs a modified exe on every player's machine, and the exe changes with every game update
[inference]. The mod does not use it. We do not repeat the patch details here.

## Pitfalls and what failed

| Test | What happened | Why / lesson |
|---|---|---|
| B3 form 1 | A re-encrypted, edited regulation with the original header crashed the boot, three times | The header's tag no longer matched the content. The decrypt itself was right |
| B3 form 2 | A plain DCX with all params booted, but the game used vanilla values | Params are not taken from a plain regulation. Fallback source unknown (see above) |
| B3 form 3 | A bare BND4 crashed the boot at the same address, twice | Not encrypted, not a DCX: same failure as a stale tag |
| B3 form 4 | One edited loose param beside the original regulation: normal prices | The valid regulation wins. Loose regulation params only count when the regulation has none |
| B3 | Four rows were at 0.0 in memory, but Lenigrast's shards still cost money | The first edit was on another vendor's rows (shop 762); his shop is 764, row 76400604. Check which row the NPC reads |
| B3b | Melentia could not be reached to check the price | She only moves to Majula after the Last Giant is beaten and her talk is done. The in-memory read of `price_rate` proved the load instead |
| B3c | The Effigy row and the lot edit read back right; the shop itself was not visited | Memory passed at every load from 12:14; the lot gave 99 Effigies in one pop-up [live B20d]. Treat "in memory" as the load proof and the game screen as a second check |
| After B28 | The Effigy row 75400607 reads back right but no longer shows in the shop | The vendors' ranges moved to a new block, so the row is outside every range. A field edit can be correct and dead. See [Vendor stock](../modding-menus/vendor-shops.md) |
| B20e | An unused lot repurposed as Torch x99 worked: the amount above the item's max held count still counted in full | `ItemLotParam2_Other` is one of the 37 params the writer refuses, so it cannot add lot rows there. The project reused existing lots (99995000-99996008) that no MSB, param, script or the exe references [data, 6 Nov]. Whether the refusal was the reason is not recorded [inference] |
| 16 Nov build | The release comparison failed when the build ran on Python 3.14 | zlib-ng. Use 3.12 |
| Any | A read-back through `paramdef.decode` showed 0.0002 for a small `price_rate` | The decoder rounds to 4 places. Read the raw field |
| Any | Two param mods collide when both ship the whole `Param` set | Ours ships all 204, so it overrides every other param mod. Ship only what you changed if you can rely on the community base [untested] |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| The regulation is a 32-byte header + AES-CTR(DCX DFLT(BND4)), 228 entries: 205 params, 12 FMG, 11 EMEVD | decrypted and unpacked; counts re-run | [data] B3, 17 Nov check |
| The unchanged regulation round-trips byte for byte on Python 3.12, not on 3.14 | re-run on Steam's file | [data] 17 Nov check; release build 16 Nov |
| A changed, re-encrypted regulation with the stale tag crashes the boot at +0x265B3E | three boots | [live B3] |
| A plain DCX with params is ignored; a bare BND4 crashes | booted / crashed, memory read | [live B3] |
| One loose param beside the original regulation is ignored | booted, normal prices | [live B3] |
| The tag check is a custom MAC over 12-byte blocks, compared at +0x26122C to +0x261238 | static reading, then the patch worked | [code] B3, 28 Oct; compare site re-read 17 Nov |
| Patching both jumps lets the re-encrypted edited file load | live; four rows 0.0 in memory | [live B3] |
| Plain DCX without params + all params loose loads, with no exe patch | live; two rows 0.0 in memory | [live B3b] |
| The same layout works through the legacy Mod Engine on a clean Steam copy | live; console log of served files | [live B45, B49] |
| Field edits on regulation params take effect (shop rows, upgrade costs, level costs, item lots) | live | [live B3, B3c, B20e, B28] |
| Added rows load: `ShopLineupParam` 692 to 1,824; `MapObjectBonfireParam` 83 to 85; `PluralSelectMenuParam` 3 to 56 | live; counts read from the v1.01 package | [live B28, B2, B36c, B57] [data] |
| The param file layout (header, row entries, data, names) | read from the files; rebuilds byte-identical on 168 of 205 | [data] `tools/paramrows.py check` |
| 37 regulation params use name offset 0 and are refused by the writer | classified by script | [data] 17 Nov check |
| `paramdef.decode` rounds floats to 4 places | read in the code; shown on 1.5 / 7000 | [code] `tools/paramdef.py` |
| The community "Load Loose Param Files" mod uses this layout; Yapped writes unencrypted regulations | public pages and source, not tested by us | [web] 5 Nov 2026 |
| Regulation params are read at boot | every regulation change was installed with the game closed and seen after a restart; no in-session change was tried | [live B20d, B28] [untested] |

`[web]` means read on a public page or in public source code, and not tested by us.

## Tools

All in `tools/`. The shared readers `pccrypto.py` (decrypt, AES-CTR), `paramdef.py` (row decode) and `bndx.py` (BND4) come
from the project's earlier toolbox.

| Tool | For this topic |
|---|---|
| `regpatch.py` | `check` (round trip and compare with the unpacked reference), `get` / `find` (decoded rows), `set` (field edits; `--plain` and `--raw` write the other forms), `loose` (one param as a loose file), `restore`. Keeps `.orig`. Built for B3 |
| `exepatch.py` | the two-jump patch for the test copy only: `status`, `apply`, `restore`. Checks the context bytes first and refuses an unknown exe |
| `looseparams.py` | the loose-params build: field edits (`ARENA_EDITS`, wildcards), the row-adding generators in `REBUILD_FROM`, the trimmed plain regulation, the read-back checks; `install`, `uninstall` (by manifest), `status`, `check` (in-memory read) |
| `paramrows.py` | `check` (rebuild unchanged: byte-identical?), `add`, `delete`. Importable `read()` / `Table.write()`; the generators build on it |
| `release.py` | builds the shipped package from Steam's own files and compares it with what was tested; needs Python 3.12 |

## Open questions

- **Why does a plain DCX without params work, when a plain DCX with params was ignored?** And where did the vanilla values
  in form 2 come from? The archives hold no regulation params. Why does a bare BND4 crash when a plain DCX does not? A
  test with Process Monitor on the game's file reads would show what is opened.
- **Are the 12 regulation FMGs and 11 `SpEffect*.emevd` in the plain DCX read at all?** The package keeps them unchanged,
  so nothing shows whether the game reads them or falls back. A changed text or effect script would settle it.
- **A partial loose set.** Does a plain regulation with only the edited params loose work, relying on the community base
  mod for the rest? That would stop our package overriding other param mods.
- **A form 2 retry** on a clean Steam copy, and with the legacy Mod Engine: Yapped-style plain regulations may load there.
- **A valid tag.** The MAC is not reproduced. Doing so would make an encrypted, edited regulation load on a stock exe.
- **Does the game accept a zlib-ng DCX stream?** Probably (the stream is valid), but the final file was only ever built on
  3.12.
- **Are regulation params re-read at a map load, or only at boot?** Only restarts were tested. One live observation
  fits "only at boot": an AreaParam row forced in memory stayed forced after its map unloaded
  ([Chameleon objects](../mechanics/chameleon-objects.md#live-test-15-nov-2026-steam-install-no-mods)).
- **Adding rows to the 37 refused params.** A writer that keeps name offset 0 should do it. Untested.
- **Is the sorted row order a rule of the game or of the format?** Our files are always sorted. Untested.
- **What `0x06` in the param header means** (0, 1 or 2), and what the 4-to-16-byte junk tail on per-map params is.

## Related pages

- [Vendor stock](../modding-menus/vendor-shops.md): the largest user of this layout: 1,132 added shop rows, and the wildcard cost edits.
- [Adding a new bonfire](../modding-world/adding-a-bonfire.md): `MapObjectBonfireParam` rows through the loose layout.
- [Adding new dialogue options](../modding-menus/adding-dialogue-options.md): `PluralSelectMenuParam` rows.
- [Giving the player items](../modding-menus/giving-items.md): item-lot edits (`ItemLotParam2_Other`), B20e.
- [Event script groups](event-script-groups.md): `EventParam_<map>` rows, the per-map side.
- [Bringing a boss back](../modding-world/respawning-bosses.md): per-map `GeneratorParam_<map>` field edits.
- [Mod Engine for DS2](mod-engine-ds2.md): the override folder that serves the plain regulation and the loose params.
- [One save file for every install](saves-and-installs.md): the rest of the install.
- [Diagnosing crashes from minidumps](crash-diagnosis.md): how the boot crash was located.
