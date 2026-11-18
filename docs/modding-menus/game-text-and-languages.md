# Adding and translating game text (PC Scholar 1.0.3.0, 2026-11-18)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 18 November 2026 from the Boss Arena project's tests.
> Test ids such as B12 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags are explained in the Evidence section.

## Summary

- **The game's text is in `.fmg` files, one set per language folder.** Menus and windows are in `menu\Text\<language>\`,
  NPC lines in `...\talk\`, developer messages in `...\bloodmes\`. A loose copy of a file replaces the game's own
  [live B1 in an unpacked copy; B45 and B49 through the legacy Mod Engine, English only].
- **The PC file is easy to rebuild:** a 0x1C-byte header, id ranges, an offset table and UTF-16 strings. A short
  reader and writer (below) gave back all 773 `.fmg` files of our test copy byte for byte [data, 18 Nov].
- **A string needs an id inside one of the file's ranges.** Fill an empty slot in a range, or add a new range for new
  ids. An id outside every range cannot just be set. Both routes worked in the game [live B2, B36c, B20d, B32].
- **Each FMG feeds one kind of window** (table below). A string can exist and never show: the list window asks for
  `npcmenu` 80000 for its title and nothing else, so title strings in other ids are never read on PC [code; live B37].
- **Texts are read when the game starts.** A build run with the game open left the old bonfire names on screen [live
  B12]. Scripts are different: they are re-read at the next map load.
- **12 language folders, fonts for 11.** The global Steam depot has no Japanese text or font. Names in other languages
  come from the game's own files, sentences from AI translation keyed by the English text, and the Japanese
  originals came from the PS3 Scholar disc. The main trap is a window text over 127 characters, which crashes the
  game: [Text length limits](text-length-limits.md).

### The file layout (PC, version 1)

All numbers are little-endian. Offsets are from the start of the file. [data; layout read from 773 files, 18 Nov 2026]

| Offset | Size | Field |
|---|---|---|
| 0x00 | 4 bytes | `00 00 01 00`: byte 1 is 0 for little-endian, byte 2 is the version, 1 on PC. (PS3 files have a non-zero byte 1 and version 0: they are big-endian) |
| 0x04 | u32 | file size, padding included |
| 0x08 | u32 | 1 in every PC file (meaning unknown) |
| 0x0C | u32 | number of ranges |
| 0x10 | u32 | number of strings |
| 0x14 | u32 | offset of the offset table |
| 0x18 | u32 | 0 |
| 0x1C | 12 bytes per range | `{first string index, first id, last id}` |
| table | u32 per string | file offset of the string; **0 means no string** (an empty slot) |
| after the table | UTF-16LE | the strings in index order, each ended by `00 00`; zero-padded so the file size is a multiple of 32 |

An id's string index is `first index + (id - first id)` in the range that holds it. In all 773 files the ranges are
sorted by id, none overlap, the string indexes run on without a gap, and the offset table sits directly after the range
records (`0x1C + 12 x ranges`). [data] Ranges are how the game keeps gaps in its id space: English `mapevent.fmg` has 32
ranges for 391 slots, and 263 of the slots hold a string.

```python
import struct

def read_fmg(d):
    """-> (ranges, strings); ranges = [(first_index, first_id, last_id)]; strings[i] is None or str"""
    assert d[1] == 0 and d[2] == 1, "not a little-endian, version-1 (PC) FMG"
    n_ranges, n_strings, table = struct.unpack_from("<III", d, 0x0C)
    ranges = [struct.unpack_from("<III", d, 0x1C + 12 * i) for i in range(n_ranges)]
    strings = []
    for i in range(n_strings):
        off = struct.unpack_from("<I", d, table + 4 * i)[0]
        if off == 0:
            strings.append(None)
            continue
        end = off
        while d[end:end + 2] != b"\0\0":
            end += 2
        strings.append(d[off:end].decode("utf-16-le"))
    return ranges, strings

def write_fmg(d, strings):
    """same header, ranges and table position; the string area laid out again"""
    table = struct.unpack_from("<I", d, 0x14)[0]
    out = bytearray(d[:table + 4 * len(strings)])
    blob = bytearray()
    for i, s in enumerate(strings):
        if s is None:
            struct.pack_into("<I", out, table + 4 * i, 0)
        else:
            struct.pack_into("<I", out, table + 4 * i, len(out) + len(blob))
            blob += s.encode("utf-16-le") + b"\0\0"
    out += blob
    out += bytes(-len(out) % 32)                 # pad to a multiple of 32 bytes
    struct.pack_into("<I", out, 4, len(out))     # the size field counts the padding
    return bytes(out)

def index_of(ranges, text_id):
    for first_index, lo, hi in ranges:
        if lo <= text_id <= hi:
            return first_index + text_id - lo
    return None      # outside every range: add a range, do not just set it
```

`write_fmg(d, strings)` with nothing changed returns `d` exactly, for every file we tried. That is your first test of
any writer. [data, 18 Nov: all 773 files in the 12 language folders of the test copy; the 4 Japanese ones were converted
from PS3 files, see below]

### Changing and adding strings

| Case | Route | Notes |
|---|---|---|
| The id is in a range and has a string | `set`: replace the string | a rename. Shared strings rename everywhere they show (see Pitfalls) |
| The id is in a range, the slot is empty | `set`: fill it | what the mod did for all 52 of its `mapevent.fmg` texts [data, v1.01 package] |
| The id is outside every range | `add_ids`: add a new range for each run of consecutive new ids | the writer then re-sorts the ranges by id, renumbers the string indexes and keeps the offset table right after the ranges, the layout all the game's own files have |

`tools/fmgwrite.py` does both. `set` refuses an id outside every range. `add_ids` refuses an id that is already in a
range, even an empty slot. The numbers from the shipped English files show the two routes (original to modded):

| File | Ranges | Strings | What changed |
|---|---|---|---|
| `mapevent.fmg` | 32 to 32 | 391 to 391 | 52 empty slots filled |
| `pluralselect.fmg` | 1 to 4 | 21 to 74 | 53 new ids |
| `npcmenu.fmg` | 87 to 88 | 821 to 823 | 2 new ids, 3 strings renamed |
| `bonfirename.fmg` | 27 to 29 | 360 to 362 | 2 new ids, 36 strings renamed |
| `titlemenu.fmg` | 16 to 16 | 87 to 87 | 1 string renamed |

[data, v1.01 package compared with the originals, 18 Nov] Empty slots exist where you might not expect them: the game's
`pluralselect.fmg` has one range, 10 to 30, 21 slots, and only ids 10, 20 and 30 hold a string. [data]

**What was seen in the game.** Filling empty `mapevent.fmg` slots: the setup questions and the shop page windows
[live B20d, B32]. New ranges: the bonfire names added for a new bonfire show in the travel menu [live B2] and the 49
fight-list labels added as new `pluralselect.fmg` ids show in the list window [live B36c]. Adding a range to
`mapevent.fmg` was never needed, so that one is untested. Ranges sorted by id is how every game file is laid out; that
the game *needs* them sorted (a binary search, say) is an inference, which is why the writer keeps them sorted.
[inference]

### Which FMG feeds which window

Every file is in `menu\Text\<language>\` (`charaname.fmg` is in its `talk\` subfolder). The "Page" column says where
the window itself is explained.

| File and id | What shows it | Evidence | Page |
|---|---|---|---|
| `mapevent.fmg`, any id in a range | The text of a YES/NO window (`1:130441`) or an OK window (`1:130443`). The script passes the id as the command's first argument | [live B20d, B32, B41] | [YES/NO windows](yes-no-windows.md); page windows: [Vendor shops](vendor-shops.md) |
| `pluralselect.fmg`, the row's `ladder_unlock_index` | One row label in the N-choice list window (`1:130481`). The game's own three strings are Gilligan's prices | [live B36c] | [Adding dialogue options](adding-dialogue-options.md) |
| `npcmenu.fmg` **80000** | The list window's title. The title id a script passes is ignored on PC | [code, read after B36b; live B37] | same |
| `npcmenu.fmg` **89101** | The confirm window after a list pick. The game's text holds `%d`, the row's soul cost | [code; live B51] | same |
| `npcmenu.fmg` 89901 | The list window's key-guide strip | [code, from the list page] | same |
| `npcmenu.fmg` **10101-10123** | The labels of the 23 NPC-menu entries, one string each, **shared by every NPC**. 10113 is "Move path", 10116 is "Talk" | [data; live B42, B43] | [NPC menus](npc-menus.md) |
| `npcmenu.fmg`, the id in a row's `npcMenu_NameID` (7540 is Melentia, 7610 Maughlin) | The NPC's name as the menu's title | [data: the names are there; the title role is from the NPC-menu page] | same |
| `bonfirename.fmg`, the **bonfire id** | The bonfire's name in the travel menu | [live B1, B2, B12] | [Adding a bonfire](../modding-world/adding-a-bonfire.md), [Moving a bonfire](../modding-world/moving-a-bonfire.md) |
| `titlemenu.fmg` **2000060010** | The title screen's "Ver." label. The exe prints the game's version after it as `%d.%02d` (1.03) | [code; live B48, B57] | below |
| `titlemenu.fmg` 2000060011 | The "Calibrations" label, with the regulation's version (2.02) printed after it by the same code | [code; live B48] | below |
| `talk\charaname.fmg`, `itemname.fmg`, `mapname.fmg`, `weapontype.fmg`, `common.fmg` | The game's own names. The mod only reads them, as the source of official names | [data] | below |

Some strings carry printf-style tokens the game fills in: `%d` in `npcmenu.fmg` 89101, `%s` in 119303 (the respec
confirm), and the key-guide string uses circled-number characters that probably draw as button icons. Keep the tokens
in every language. [data; the icon reading is an inference]

**The title label.** The exe loads id 2000060010 with `mov edx, 2000060010` at 0x1400f39c0 (Ghidra address, image base
0x140000000; RVA +0xf39c0). It then prints the game's version number after the label with the wide format string
`%d.%02d` (`lea rdx` at 0x1400f3a7d; the string is at 0x1410bce08), which gives "1.03". "Calibrations" (2000060011) is
loaded at 0x1400f3b0e and uses the same format string at 0x1400f3be8, and prints the regulation's version (2.02).
[code, bytes checked in the unmodified Steam exe, 18 Nov; the printed numbers live B48] The box that holds the label is
right-aligned, wraps at spaces and grows downwards, so a longer label wraps over the "Calibrations" line [live B48].
Widths and what fitted: [Text length limits](text-length-limits.md). Translations differ in length (Polish "Wer.",
Russian "Версия"), so check each language.

### When a change loads

| What | When it is read | Evidence |
|---|---|---|
| `.fmg` texts | **at game start.** Build first, then launch | [live B12 for `bonfirename.fmg`: built with the game up, old names stayed]. For `mapevent`, `npcmenu` and `pluralselect` the project always restarted, so "needs a restart" is [inference] |
| Event and talk scripts | at the next load of their map, no restart | [live B20d] |
| Per-map params and maps | at each map load | [live B12, B13] |

One oddity: on 5 Nov one restart still showed the old bonfire names although the file was right. Later restarts showed
the new names. We never found the cause [live B13, B15; open].

### Language folders and fonts

The exe holds a table of twelve folder names, as wide strings from file offset 0x10f8f80: Japanese, English, French,
Germany, Italian, Spanish, Korean, Chinese, Russian, Polish, Portuguese, NeutralSpanish. [code, read 18 Nov] It builds
the text paths from format strings next to it (`Menu/Text/%s/talk/%s.fmg`, `Menu/Text/%s/bloodmes/%s.fmg`,
`text:/Text/%s/%s.fmg`) [code]. Steam's own language names for ten of them sit in the exe too (`english`, `french`,
`german`, `italian`, `spanish`, `koreana`, `tchinese`, `russian`, `polish`, `brazilian`). The natural pairing is
`koreana` with `korean`, `tchinese` with `chinese` and `brazilian` with `portuguese`, and the game takes its language from
the Steam setting (Properties, Language). Both are [inference]: we did not trace the code that pairs the names with the
folders, and B40 planned a language switch but recorded no result. There is no Steam name in the exe for Latin American
Spanish (an ASCII search for `latam` found nothing; a wide-string search was not run). [code, 18 Nov]

| Text folder | Language | Font folder (`font\<Folder>\FeFont_Big`, `FeFont_Small`) | Note |
|---|---|---|---|
| `english` | English | yes | |
| `french` | French | yes | |
| `germany` | German | yes | the game's spelling |
| `italian` | Italian | yes | |
| `spanish` | Spanish (Spain) | yes | |
| `neutralspanish` | Latin American Spanish | yes | one file fewer: no `bloodmes\m10_15_00_00.fmg` |
| `portuguese` | Brazilian Portuguese | yes | the mod's translations are Brazilian; the exe's Steam name is `brazilian` [inference] |
| `polish` | Polish | yes | |
| `russian` | Russian | yes | |
| `korean` | Korean | yes | |
| `chinese` | Traditional Chinese | yes | the texts are Traditional; the exe's Steam name is `tchinese` |
| `japanese` | Japanese | **none** | no text folder and no font in the global depot, see below |

[data, an unpacked global copy, 18 Nov] The text folders are lower case and the font folders capitalised in an
unpacked copy; Windows does not care, and the mod ships lower-case text folders [live B49 on English]. The 11 PC languages
have 70 `.fmg` files each (neutral Spanish 69): 27 in the folder itself, 26 in `talk\` and 17 in `bloodmes\`.

**The ranges are the same in every language, the strings are not.** For the five files the mod changes, the ranges of
all 11 PC languages are identical to English's. But a slot can be empty in one language and filled in another (the
Korean and Chinese `bonfirename.fmg` lack two names English has; six languages lack `mapevent.fmg` 1007). So write each
language from that language's own original, never copy a file between languages. [data, 18 Nov] The PS3-derived Japanese files
have a different range layout (its `npcmenu.fmg` has 96 ranges to English's 87), so the writer must read each file as it is.

**Font gaps.** Each font draws only the characters in its glyph table, and the fonts differ: no PC font but Portuguese
and Chinese has the en dash, the guillemets exist only in French, Portuguese and Korean, and Chinese has no 篝. The
table, the measuring recipe and the pixel limits are on [Text length limits](text-length-limits.md); do not repeat
that work. What the game draws for a missing glyph was never seen.

### Names and sentences (translating)

A text in a mod can be a *name* (a boss, an item, a place) or a *sentence*. The two are handled differently.

**Names come from the game, in every language.** The game already has an official name for each thing in each
language, so the mod uses that name and never its own wording. It writes each text as a template with tokens, and looks
the name up in the *original* file of the language being built:

| Token | File | Example (English) |
|---|---|---|
| `{c:3340}` | `talk\charaname.fmg` | Throne Watcher |
| `{cs:3330}` | `talk\charaname.fmg`, the part before ", " | "Velstadt, the Royal Aegis" gives Velstadt |
| `{i:50960000}` | `itemname.fmg` | Soul Vessel |
| `{m:20210002}` | `mapname.fmg` | Throne of Want |
| `{w:21}` | `weapontype.fmg` | Staff |
| `{t:29201}` | `npcmenu.fmg` | Sorcery |
| `{bn:id}`, `{cm:40000008}` | `bonfirename.fmg`, `common.fmg` | Company of Champions |

[code, `tools/i18n.py`; ids and English names read from the game's files, 18 Nov] The short form does not travel: it
works where the name comes first, but French writes "Velstadt l'Égide du Roi" with no comma, Korean puts the title
before the name and Chinese puts the title in brackets. Where the short form fails the mod uses the whole name, or in one
case (Korean Velstadt) a hand-written short name. [data] If a language's file has no string for the id, the build
stops; it never falls back to English silently.

**Sentences are AI translations, keyed by the English text.** The mod keeps one table: the exact English text is the
key, and each language's sentence is the value. Names inside the sentence are tokens (a number is `{n}`). Two rules
make it safe to change the English later:
- a text with no entry stays English in every language file, and the build prints the list, so an edited English
  sentence never ships with a stale translation;
- a check renders every entry in every language and fails on a missing name, a missing glyph or a window that is too
  long (see [Text length limits](text-length-limits.md)).

The mod's readme says the sentences are AI translations, and they need a read by a speaker of each language before
anyone relies on them. Keep the game's own style: French puts a space before `: ? !`, and the game's Japanese questions
end in `？` with no full stop. [data] The other ten languages were never seen in the game (see Open questions).

### Japanese

**Why it is missing.** The global Steam depot (the one the project's copy came from) has no Japanese menu text and no
Japanese font. The project's note says Japan-region Steam copies add them. The exe knows the language: Japanese is the
first of its twelve folders. The Boss Arena's first release shipped the other 11; a player's report found the gap, and
the next version added Japanese. [data, 16 Nov]

**Where the originals came from.** The PS3 Scholar of the First Sin disc has Japanese FMGs. They are the
same format as the PC's with three differences, and `tools/japanese.py` `convert()` undoes them: big-endian numbers,
version byte 0 (PC 1), a `ff` byte at +9 (PC 00), and UTF-16BE text. The conversion was checked on six English files
that are the same on both platforms: byte for byte the PC's [data, 16 Nov, from the tool's note]. For the four files the
mod ships, the PC's English, French, Korean and Chinese text agrees with the disc's text for text, except ids that only the
PC has: `mapevent` 9100, `npcmenu` 6940, 6950 and eleven ids from 8870 to 8980 [data, 16 Nov; the ids from the tool's
manifest]. Those 13 ids get the PC's English text in the Japanese files, because no disc we have holds a Japanese
version. `titlemenu.fmg` is not made for Japanese: the disc's legal text differs from the PC's, so the Japanese title
screen has no mod label.

**Font and widths.** There is no Japanese font to measure. The glyph set is every character used by the disc's own
Japanese FMGs (2,053 characters, listed by `glyphs()`), and each is given its width from the Chinese font, which has kana.
Limits are set the same way: [Text length limits](text-length-limits.md).

**What was not seen.** The Japanese files were never run in a game set to Japanese [untested]. A loose FMG replaces the
whole file [inference], so on a Japan-region copy the PC-only ids above would show English where the copy's own Japanese
text may have had Japanese.

### Developer messages and the other FMGs

- **`bloodmes\<map>.fmg`** holds short per-map messages: 17 files, 1 to 13 strings each, at most 24 characters in the
  base-game maps and 65 in the three DLC maps (`m50_35` to `m50_37`). The exe has a path format for them
  (`Menu/Text/%s/bloodmes/%s.fmg`) [code, data]. **How they are placed is not known.** The script command table has
  `1:132221` (enable event blood message) and `1:131101` (enable a no-blood-message area), but no PC event or talk script
  uses either, and the one MSB we looked at (`m10_10`) has no message section. [data, 5 Nov; open] An OK window from an
  event script is the working substitute ([YES/NO windows](yes-no-windows.md)).
  The four `bloodmessage*.fmg` files in the text folder are probably the word lists of the player-message system
  [inference from the names].
- **`talk\<map>.fmg`** holds an NPC's lines. Majula's has 716 strings whose ids look like `<npcMenu id> x 10000 + n`
  (70000000 to 78600710) [data; inference that they are talk lines]. The mod changed none.
- **The other menu FMGs** (`bofire`, `charamaking`, `dconlymessage`, `detailedexplanation`, `iconhelp`, `ingamemenu`,
  `ingamesystem`, `keyguide`, `prologue`, `shop`, `simpleexplanation`, `staffroll`, `titleflow`, `win32onlymessage`)
  were not changed or traced; do not take a role from a file name as fact.
- **The regulation holds 12 tiny `regulation<Language>.fmg` files**, one per language, Japanese included. The three we
  opened (English, French, Japanese) each hold a single string, id 100 [data, 18 Nov]. They are not menu text. Whether
  the game reads them from a plain-DCX regulation is open ([Editing params](../modding-scripts/editing-params.md)).

## How to do it

For one language:

1. **Get the original FMG** for that language: from an unpacked game folder, or extract it from the archives. Keep a
   copy as `<file>.orig` and always build from that copy. Every build is then repeatable, an undo is a file copy, and a
   checker can compare the result with the original.
2. **Rebuild it with no change and compare the bytes** (`fmgwrite.py check`, or the snippet above). If they differ,
   stop: your reader is wrong for this file.
3. **Choose ids.** For a window text, find ids no script names (see [YES/NO windows](yes-no-windows.md)) that sit in a range
   with an empty slot. For a list, shop page or bonfire, use the id the table above names.
4. **Write it.** In a range: `python tools/fmgwrite.py set in.fmg out.fmg 10655="New name"`. Passing the same path as
   `in` and `out` keeps the first copy as `.orig`. For new ids, from Python:
   `fmgwrite.add_ids(open(p, "rb").read(), {3650: "Darklurker", 10685: "Giant Lord"})`.
5. **Read it back.** Parse the new file: only the ids you meant to change differ, the ranges are what you expect.
   (`tools/version.py` does exactly this for the title label.)
6. **Put the file** in the language's text folder of the game, or in the same relative path of your Mod Engine override
   folder ([Mod Engine for DS2](../modding-scripts/mod-engine-ds2.md)). **Restart the game.**
7. **Open the window.** A string nothing asks for never shows, and a too-long window text crashes only when the
   window opens ([Text length limits](text-length-limits.md)).

For all languages, add these to the loop:

1. Build each language from *its own* original, using `set` where the id is in a range and `add_ids` where not (the
   mod's `i18n.apply` chooses per id).
2. Put names in as tokens and look them up per language (table above). Never translate a name the game already has.
3. Translate the sentences once, key them by the English text and review them with a speaker of each language.
4. Check every result against each language's glyphs and window limits before it is written.
5. Fail loudly on a missing translation, or keep English and report it. Do not ship a stale translation.
6. Say in your readme which languages were machine translated.

## Pitfalls and what failed

| Test | What happened | Lesson |
|---|---|---|
| B12 (4 Nov) | Names built while the game ran did not change | Texts load at start. Build, then launch |
| B13 and B15 (5 Nov) | One restart still showed old names although the file read back right; later restarts showed new names | Cause unknown. The file was not at fault, so a restart that shows old names is not proof of a bad build |
| B36b read, B37 (7 Nov) | The fight list had its own title strings in new `npcmenu.fmg` ids. The exe always asks for 80000, so every list read "Pay how much?" until 80000 itself was renamed | The list window ignores the title id a script passes [code]. A new id is useless if no code asks for it |
| B37 (7 Nov) | Renaming 80000 renamed Gilligan's list title too | A fixed id is shared by every user of the window. The only lookup in the exe is at 0x1400c47fc, so the change was safe |
| B51 (8 Nov) | The confirm after a pick read "Souls needed: 0". 89101 is always shown with the row's cost | Only the text can change, so the mod rewrote 89101 in all 11 languages |
| B42, B43 | A spare menu entry ("Move path", 10113) was renamed "More items" for the six paged vendors | "Talk" (10116) is one string for every NPC [data], so renaming it would rename everyone's: use a spare entry ([NPC menus](npc-menus.md)) |
| B41 (8 Nov) | A 131-character question crashed the game as its window opened | 127 characters is the limit. [Text length limits](text-length-limits.md) |
| B48 (9 Nov) | "Boss Arena v1.0  Ver." wrapped over the Calibrations line | The title label has a narrow box. Measure it per language |
| 16 Nov | The first release had no Japanese: it worked from the depot's files | Check that every language folder the exe lists is in your source files |
| B40 | The other ten languages were built, installed and checked by tools only | No live result for any language but English. Look at least one in the game |

Other traps:
- **An unfilled language.** An empty slot probably shows an empty window, and an id with no range in that language's file
  is untested. The mod fills all 11 (12 with Japanese) so it never met this. [inference; untested]
- **A guard against stray edits.** The project's checker compares every string of a built file with the original and
  fails if one that already held text changed and is not on an allow-list (the deliberate renames). [code, `tools/ids.py`]
  Copy the idea: it catches a rename you did not mean.
- **Names are not always derivable.** See the short-form note above.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| A loose FMG replaces the game's own | live | B1; B45, B49 (Mod Engine, English only) |
| The PC layout and the reader/writer round-trip 773 files | data | 12 language folders of the test copy, 18 Nov |
| `set` fills empty slots in `mapevent.fmg` | live | B20d, B32 |
| New ranges work in `bonfirename.fmg` and `pluralselect.fmg` | live | B2, B36c |
| New ranges in `mapevent.fmg` | not tested | open |
| Names load at game start | live (`bonfirename`) | B12 |
| The list window uses 80000, 89101 and 89901 only | code (read after B36b); live for the renames | B37, B51 |
| "Move path" (10113) is one string, renamed for the "More items" entry | data, live | B42, B43 |
| The title label is 2000060010; the exe prints `%d.%02d` after it | code, live | exe, 18 Nov; B48, B57 |
| The exe lists 12 language folders, Japanese first | code | file offset 0x10f8f80, 18 Nov |
| No `font\Japanese` in the global copy | data | font folder, 18 Nov |
| No Japanese menu text in the global depot | data (depot index) | `tools/japanese.py`, 16 Nov |
| PS3 FMGs convert to PC layout exactly | data | six English files, byte for byte, 16 Nov |
| PS3 and PC text agree in English, French, Korean and Chinese | data | `tools/japanese.py`, 16 Nov |
| The ranges of the five changed files are the same in all 11 PC languages | data | 18 Nov |
| Developer messages are not placed by any script | data | event command table, scripts, MSB sections; 5 Nov |
| Sentences were translated by AI, names taken from the game | design, code | `tools/i18n.py` |
| Texts in a language other than English shown in the game | **not seen** | B40 optional step has no result |

## Tools

The project's own scripts.

| Tool | What it does for this topic |
|---|---|
| `tools/fmgwrite.py` | `check <file>` rebuilds with no change and says if it is byte-identical. `set <in> <out> <id>=<text> ...` changes or fills strings (refuses an id outside every range; keeps `.orig`). `restore <file>`. Importable: `parse`, `build`, `index_of`, `add_ids(bytes, {id: text})` |
| `tools/i18n.py` | `localise()` writes every changed id into the other languages' files; `translate()` finds the sentence or the official name; `render()` expands the tokens; `apply()` sets or adds per id; `check()` renders every text in every language |
| `tools/japanese.py` | `make` builds the Japanese originals from the PS3 files; `convert()` is the PS3 to PC layout step; `verify`; `install`; `glyphs()` |
| `tools/version.py` | Puts the label before "Ver." in `titlemenu.fmg`, reads the file back, refuses a label that is too wide |
| `tools/textexport.py` | Exports every changed text, one file per language, with line widths against the limit, for a human read |
| `tools/ids.py` | `check_build()` fails a changed original string that is not on the allow-list |
| `fmg.py` | Read-only dump of an FMG as `id<TAB>text` (PS3 or PC) |

## Open questions

- Does a new range in `mapevent.fmg` work (the window FMG)? Never needed. [untested]
- What shows for an empty slot, or an id with no range, in one language? [untested]
- Does the game need ranges sorted by id, or would any order work? [untested]
- How are the `bloodmes` files placed? Try `1:132221` with guessed arguments. [untested]
- What are the `talk\<map>.fmg` ids to the talk script, and the 12 `regulation<Language>.fmg` files for? [untested]
- Which language does Latin American Spanish take, and how do Steam's names pair with the folders? [untested]
- How do the other ten languages look in the game? Only English was seen. [untested]
- How does the Japanese set look in a Japan-region game, and do the PC-only ids matter there? [untested]
- Why did one restart on 5 Nov still show the old names? [open]

## Related pages

- [Text length limits](text-length-limits.md): the 127-character crash, the pixel limits, font gaps
- [YES/NO windows](yes-no-windows.md): `mapevent.fmg` texts and the windows that show them
- [Adding dialogue options](adding-dialogue-options.md): the list window's title, confirm text and row labels
- [NPC menus](npc-menus.md): the shared `npcmenu.fmg` entry labels and the rename
- [Vendor shops](vendor-shops.md): the page-name windows
- [Adding a bonfire](../modding-world/adding-a-bonfire.md) and [Moving a bonfire](../modding-world/moving-a-bonfire.md): `bonfirename.fmg`
- [Bonfire lit state](../modding-world/bonfire-lit-state.md): the travel menu entry
- [Mod Engine for DS2](../modding-scripts/mod-engine-ds2.md): loading loose text files
- [Editing params](../modding-scripts/editing-params.md): the regulation that holds the 12 small FMGs
