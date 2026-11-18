# Text that crashes the game, and missing glyphs (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B41 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags are explained in the Evidence section.

## Summary

- **A window text of more than 127 characters crashes the game the moment the window opens.** Count UTF-16 code units
  in the whole text, line breaks included. The window keeps its text in a string with a fixed buffer of 128 elements;
  a longer text makes the game throw a C++ `std::length_error` ("string too long") that nothing catches, and the game
  aborts (exception 0xc0000409, fast-fail code 7). The player sees no error message, only the game closing. The Boss
  Arena's crashed text had 131 characters; the longest text that worked had 124. [live B41; code, read 17 Nov 2026]
- **It is the character count, not the pixel width.** The first reading of B41 blamed the widths (lines of 771 and 791 px
  against 759 px that had worked). All three measures were over together, so the test could not tell them apart. The
  exe can: the length check sits on the text store, before any layout, and looks at nothing but the count. [code]
  The 759 px rule the mod still keeps is a display precaution, not a crash limit. The OK windows of the shop pages
  showed English lines of up to 808 px without trouble. [live B32; see Pitfalls]
- **The trap: the crash comes when the window opens, not when the text is installed.** A bad text sits quietly in
  `mapevent.fmg` until the player reaches that window. The mod's setup question crashed only after the player said YES
  to the question before it. Check every text before it ships, in every language you fill.
- **The mod's rule for window texts:** at most 3 lines, 124 characters (3 under the real limit), every line at most 759 px
  in the language's own `FeFont_Big` (Japanese 673 px), and every character present in that language's fonts. Names in
  lists and on the travel menu: at most 571 px. [live B20d, B36c, B41 run 3; the tool `tools/i18n.py`]
- **Fonts have gaps.** No PC font has the en dash except Portuguese and Chinese; the guillemets exist only in French,
  Portuguese and Korean; Chinese has no 篝. The mod swaps a missing character for one the font has. What the game
  draws for a missing glyph was never seen. [data; untested]
- **The title menu's version label wraps** when it is wider than its box (B48): 266 px fitted, 353 px did not. That is
  a wrapping fault, not a crash.

### The crash, step by step

All addresses are for the Steam 1.0.3.0 exe. Ghidra names (`FUN_14xxxxxxx`, image base 0x140000000) are followed by
the RVA. Steps 1 to 4 were read in a headless decompile on 17 Nov 2026 (never run live). Step 5 and the stack
addresses come from the minidump of the 8 Nov crash, as the project's notes record them; the dump file itself was not
re-read for this page. [code; live B41]

| Step | Where | What happens |
|---|---|---|
| 1 | script command `1:130441` (YES/NO window), handler `FUN_1401987f0` (+0x1987f0) | Reads the front-end manager at `[GameManagerImp+0x22E0]`, then asks the next function to store the text. The handler belongs to a class the exe's own symbols call `EventWindowManager` |
| 2 | `FUN_140198270` (+0x198270) | Looks the text up by id in the text table (`FUN_140503620`, +0x503620). For a plain text (the `kind` argument 0) it measures the text with a wide `strlen` and passes pointer and length to step 3 |
| 3 | `FUN_1400efa30` (+0xefa30): the string's "assign" routine | If the length is above 0x7F (127) it calls `FUN_140c288ec` (+0xc288ec) with the text "string too long". Otherwise it copies the text into the window's string (object +0x60 of the window manager) |
| 4 | `FUN_140c288ec` | Builds a `std::length_error` and throws it (ThrowInfo at +0x15576f8) |
| 5 | the C++ runtime (from the dump) | Nobody catches it: `terminate`, then `abort()`: `mov ecx, 7 ; int 29h` at +0xc39b1c. That is exception 0xc0000409 with fast-fail code 7 |

Why 127: the window manager's string uses the engine's fixed-size allocator (`DLFixedStdAllocator`). The class's
destructor (`FUN_140198600`, +0x198600) asserts that the string's capacity plus one never passes 128 ("Invalid item
number"). So the buffer is 128 elements, which hold 127 characters and the terminator. [code; the 128-element reading
is an inference from that assertion and from the 0x7F check in step 3]

**What else shares the limit.** Four more handlers of the same class call the same step-2 function: `FUN_140198720`,
`FUN_140198920`, `FUN_140198a10` and `FUN_140198f90`. `FUN_140198920` opens a window with a single button label, which
fits the YES-only ("OK") window `1:130443`. We did not trace which command reaches which handler, so "every window of
the class takes 127 characters" is [inference from shared code]. Three more functions use the same string routine
(+0xef700, +0x2e6e60, +0xb13040); we did not look at them.

**The `kind` forms.** `kind` 1 and `kind` 2 (a number, an item name in the text; see [YES/NO windows](yes-no-windows.md))
first format the text into a 128-element buffer on the stack, then assign the result. In the code, form 1 formats a
number into the text and form 2 looks a name up in a second text table first, which fits the YES/NO page's reading
of `kind` 1 and 2 (that this byte is the `kind` argument is itself an inference). What happens if the formatted
text is longer than the buffer was not read to the end. [code; open question]

**`[ChrName]`.** When a count in the command's arguments is positive, the window code replaces the literal token
`[ChrName]` with the player's name. The replace routine has the same 127 check, so a text that holds the token must stay
under the limit with the name in it. Which argument sets the count was not traced; the mod never uses the token. [code]

### What the check does not look at

The check counts characters. It does not look at lines or at width. Layout happens after the text is stored, in the
front-end window code we did not read, so what the game does with a very wide or a very tall text is not known from the
code (see Open questions). The mod's limits on lines and width come from what was shown on screen, not from the exe.

### Counting characters

- Count UTF-16 code units. A Chinese, Japanese or Korean character is one unit, like a Latin letter.
- A line break is one unit: it is stored as 0x000A in the `.fmg` string area (checked on an English `mapevent.fmg`). [data]
- `len(text.encode("utf-16-le")) // 2` in Python gives the number, line breaks included.
- The length limit is a property of the window, not of the `.fmg` file. The game's own `mapevent.fmg` holds texts of
  136 and 132 characters (ids 4012, a stamina tip, and 2304, a covenant text), so they must reach the screen by another
  window. We did not trace how. [data; inference]

### How the three measures lined up in B41

| Measure | Worked | Crashed | Trigger? |
|---|---|---|---|
| Characters in the whole text | at most 124 | 131 | **yes**: the limit is 127 [code] |
| Characters in the longest line | 44 | 46 | no: not in the code path |
| Width of the widest line | 759 px | 771 and 791 px | no: not in the code path |

The crashed text was 3 lines of 46, 37 and 46 characters (771, 609 and 791 px in English). The fix on 8 Nov shortened it
to 121 characters and 703 px, which cured the real cause and the two false ones together. Because one reword changed
all three measures, the test proved the fix but not the cause.

### What the game's own windows hold

Read from the unmodified event and talk scripts, for the text ids written as literals (the choice library passes its id
as an argument, which we did not trace): the game's YES/NO windows show at most 48 characters and 345 px, its OK
windows at most 25 characters and 446 px. [data, 17 Nov] The Boss Arena's questions are two to three times longer than
anything the game puts in these windows. That is why the project met a limit the game never reaches.

### Measuring pixel widths

The width rule needs each language's font. [data]

1. The fonts are `font/<Language>/FeFont_Big.fontbnd.dcx` (and `FeFont_Small`): a DCX around a BND4 with the glyph table
   `FeFont_Big.ccm` and texture files.
2. `.ccm` header: total size at +0x04, glyph count at +0x10, offset of the glyph table at +0x18 (English: 770 glyphs, 24 520
   bytes, table at 0x1798; Chinese: 6 292 glyphs).
3. The table is 24-byte records. At record +0 the code point (u32). At +8 the texture index, +10 the pre-space (the
   space before the glyph), +12 the glyph width and **+14 the advance** (all s16). The rest of the record is zero in
   the records we looked at.
4. A line's width is the sum of the advances of its characters. Examples in English `FeFont_Big`: space 8, `i` 11,
   `a` 19, `A` 27, `W` 35, widest glyph 40.
5. A character the table lacks is a gap (see Glyphs below).

```python
import struct
def advances(ccm: bytes) -> dict[int, int]:
    table = struct.unpack_from("<I", ccm, 0x18)[0]
    return {struct.unpack_from("<I", ccm, o)[0]: struct.unpack_from("<h", ccm, o + 14)[0]
            for o in range(table, len(ccm) - 23, 24)}
def width(line: str, adv: dict[int, int]) -> int:
    return sum(adv[ord(c)] for c in line)        # KeyError = a glyph the font lacks
```

Two things the numbers do not tell you. First, we measured everything in `FeFont_Big`; that the YES/NO window draws in
that font was never checked [inference]. The title label's behaviour in B48 matched the numbers (266 px fitted, 353 px
wrapped), which is some support. Second, the window may scale the text; a pixel count is only meaningful against what
has been seen on screen.

### Limits the mod keeps, and what each rests on

| What | The mod's limit | What is known | Evidence |
|---|---|---|---|
| Characters in a window text | 124 (`WINDOW_CHARS`) | Crash above 127 | B41, code |
| Lines in a window text | 3 | 3 is the most ever shown (the setup question, a page name). Nothing is known about 4 | B20d, B32 |
| Width of a line, YES/NO window | 759 px | The widest line shown fine in that window. Not a crash limit | B20d, B41 run 3 |
| Width of a line, OK windows of the shop pages | English pages up to 808 px accepted (`PAGE_PX` in the export tool); other languages keep 759 px | English lines up to 808 px, 2 to 3 lines, up to 112 characters, shown without complaint | B32 |
| Japanese window line | 673 px | An estimate, see below | `tools/japanese.py` |
| Fight-list rows, window titles | 571 px | The widest row shown fine (English "Prowling Magus and Congregation"). Nothing wider tried | B36c |
| Bonfire names in the travel menu | 571 px, or the game's own widest name in that language | Spanish 579 px, Russian 576 px; every other language 571 px | `tools/i18n.py` |
| Title menu version label | 266 px | 266 px fitted, 353 px wrapped | B48 |
| Rows in an N-choice list | 12 | A 13th row hits the engine's out-of-memory assert. See [the N-choice list](adding-dialogue-options.md) | B36b, code |

The shipped texts keep under the character limit in every language. The only lines over 759 px are English shop-page
names (six texts, 761 to 808 px). Longest text and widest line in each language, counted on the v1.01 package on 17 Nov
(all 52 `mapevent.fmg` texts the mod changes, question windows and page windows together): [data]

| Language | Longest text (characters) | Widest line (px) |
|---|---|---|
| English | 119 | 808 (a shop page window) |
| French | 120 | 757 |
| German | 111 | 753 |
| Italian | 120 | 753 |
| Spanish, Latin American Spanish | 122 | 754 and 753 |
| Portuguese | 121 | 754 |
| Polish | 119 | 759 |
| Russian | 115 | 751 |
| Korean | 76 | 759 |
| Chinese (Traditional) | 50 | 733 |
| Japanese | 58 | 671 (limit 673) |

As far as the log shows, the live tests were run in English. The other languages were checked with the tools, not seen
in the game (B40's optional check in French has no recorded result). [no live result found]

### Glyphs: what each language's font lacks

The game draws only characters its font has. The project checked, for each language, that every character of every text
exists in **both** `FeFont_Big` and `FeFont_Small` (they differ by a few glyphs: one in Italian, two in Chinese). [data,
fonts read 17 Nov]

| Character | Fonts that have it | The mod's swap |
|---|---|---|
| – en dash, — em dash | Portuguese, Chinese. The other nine PC fonts lack them, and so does the Japanese set (the characters the game's own Japanese text uses) | a hyphen `-` |
| « » guillemets | French, Portuguese, Korean | “ ” |
| „ low quote | German, Portuguese, Polish, Korean | “ |
| “ ” curly quotes, … ellipsis | all 11 PC fonts and the Japanese set | none needed |
| 篝 (the kanji in the Japanese bonfire word) | none of the 11 PC fonts; the Japanese set has it | Chinese uses 營火, the game's own Chinese word |

The first list the project wrote on 8 Nov ("zh lacks 篝; it, es, ru lack « »; en, de, pl lack –") held only the gaps its
own texts ran into. The table above is the full probe set measured on 17 Nov. [data] English also lacks « » and „; the
Latin-script fonts other than Portuguese lack the dashes.

What the game does with a missing glyph (a blank, a box, nothing) was **not seen**: the mod never shipped one. [untested]
Treat a gap as a bug until you have looked.

Names are not translated by the mod: boss, item, place and weapon-type names come from the game's own files in each
language, so they always have their glyphs. Only the mod's own sentences need the check.

### Japanese

The global Steam depot for the PC game has no Japanese text and no Japanese font; Japan-region copies add them. The exe's
language table has 12 folders, Japanese first. Boss Arena v1.0 shipped the other 11; v1.01 added Japanese. [data, 16 Nov]

With no Japanese font on the build machine, widths were estimated:
- the glyphs are those the game's own Japanese text uses (the PS3 Scholar disc's Japanese files supply the text);
- each is given the advance it has in the Chinese font, which has kana, or that font's usual full or half width;
- the window limit is the game's own widest one-line Japanese YES/NO question (mapevent id 1004, the Darksign question,
  22 characters) measured the same way: **673 px**. The stand-in's error cancels out, because the same estimate sets the
  limit and measures the text.

All shipped Japanese window texts are at most 58 characters and 671 px, so the 127-character crash cannot come near.
List rows and travel names keep the other languages' limits; the widest Japanese name measures 502 of 571 px. The
Japanese texts were **not seen in the game**: the project's release notes say so, and no test log entry records a
run with the game set to Japanese. [data; untested live]

### The title-menu label (B48)

The title screen prints a "Ver." label and the game's number after it. The mod puts its own name and version in front
of the label in `titlemenu.fmg` (id 2000060010). The label sits in a right-aligned box of fixed width that wraps at spaces
and grows downwards.
- "Boss Arena v1.0" (266 px) showed whole; "Boss Arena v1.0  Ver." (353 px) wrapped, and the word "Ver." landed on the
  next line over the "Calibrations" line. [live B48, 9 Nov]
- "Arena 1.01 Ver." (256 px in English, at most 261 px in any language; Korean is widest) showed on one line. [live B57;
  one language was seen, the others are measured only]
- "Arena v1.01 Ver." (277 px) was never shown. The box's real width lies between 266 and 353 px; the build refuses any
  label above 266 px. [inference]
- Japanese has no mod label: the disc's Japanese title text has a different legal text from the PC's.

## How to do it

A check for any text you add to a window, before it goes in the game:

1. **Count characters.** UTF-16 units, line breaks included. Stop at 127; keep a few under, as the mod does (124).
   If the text has the `[ChrName]` token, count the name too.
2. **Keep it to three lines**, and each line to about 759 px in that language's `FeFont_Big` (width per the snippet
   above). The pixel figure is what was seen to work, not a measured edge.
3. **Check every character** in both fonts of every language you fill. Replace the gaps (table above) or reword.
4. **Repeat for every language.** Lengths differ: the same shop-page text is 112 characters in English and 122 in
   Spanish. CJK texts are short in characters (at most 76 in the mod's windows) but wide in pixels.
5. **Open the longest window of each language in the game.** A crash as the window opens means the count is over. A line
   that looks clipped or off the window means the width is.
6. **Let a tool refuse a bad text.** In the project, `arenasetup.texts()` raises if a text fails `i18n.fits`, so a long
   text cannot reach the game by accident.

## Pitfalls and what failed

| Test | What happened | Result | Lesson |
|---|---|---|---|
| B41 run 1 (8 Nov) | Legacy Mod Engine copy, new character: YES to the setup's first question. The next window held the new question 2 | **Crash**, no error window | Suspects: the Mod Engine route, a fresh character, the new text |
| B41 run 2 (8 Nov) | The same character in the debug copy | The same crash | Not the install route, not Mod Engine: the text |
| B41 diagnosis | Dump read, widths measured: 759 px against 771 / 791 px; 124 characters against 131 | Text reworded to 121 characters and 703 px | The reword cured the crash. The cause named at the time (width) was wrong, as the exe read of 17 Nov shows |
| B41 run 3 | The setup asked, items given | **PASS** | |
| 9 Nov text review | A reworded line of question 1 measured 765 px, over 759 | The line break was moved | The width rule flagged it. Keep it as a display precaution, even though it is not the crash |
| B48 (9 Nov) | "Boss Arena v1.0  Ver." (353 px) in the title label | The label wrapped onto the next line | The label has its own, narrower box (see above) |
| B28-B32 (6 Nov) | English page names with lines up to 808 px in OK windows (the 808 px figure is from the export tool's note) | Shown without complaint | Width alone is not an edge for OK windows. This is consistent with the code, where only the count is checked |

Other things worth knowing:

- **The project's tools and comments still say "771 px crashed".** That sentence is true but it names the wrong cause.
  The tools enforce both numbers (759 px and 124 characters), so nothing in the mod is at risk. If you copy the rule,
  copy it as "127 characters is the crash limit; 759 px is what was seen to display well".
- **The Mod Engine route was a false lead.** The crash was first suspected to belong to the legacy Mod Engine copy or
  to a fresh character. Crashing the debug copy with the same character in B41 run 2 ruled both out the same day.
- **The player sees the game close and nothing else.** If you read a dump, expect 0xc0000409 and fast-fail code 7: that is
  an unhandled C++ exception, not a memory fault.

### How the crash was identified

The method, for anyone who has to do the same (the general recipe is on [Diagnosing crashes](../modding-scripts/crash-diagnosis.md)). [code, B41]

1. **Get the dump.** Windows left a mini dump in `%LOCALAPPDATA%\CrashDumps`, named `<exe>.<pid>.dmp`, which is where its
   LocalDumps setting writes by default.
2. **Read the exception record.** Code 0xc0000409 at +0xc39b1c, where the instructions are `mov ecx,7 ; int 29h`:
   the C runtime's `abort()`, which fast-fails with code 7. A game that dies there has thrown a C++ exception nobody
   caught.
3. **Find the exception's type.** The C++ throw info (`_ThrowInfo`) is a read-only structure in the exe. The throw
   info the crash stack points to, at +0x15576f8, lists the classes `length_error`, `logic_error`, `exception`.
4. **Find where it was thrown.** Scan the crashing thread's stack for values that point into the exe's code. The ones
   that made sense: the string routine +0xefa30 (its slow path loads "string too long"), then +0x198326, +0x19882a,
   +0x463102 and +0xe23e64.
5. **Name the code.** Open those addresses in Ghidra. On 8 Nov the first reading was "a UI text string built with an
   impossible length while a window was being made". The decompile of 17 Nov shows the length is a real one: the window's
   string simply cannot hold more than 127 characters.

The scanner used in steps 2 to 4 was a throw-away script, not kept in the project's tools; any dump reader that gives
the exception record, the module list and the stack memory can do the same.

## Evidence

Tags: `[live Bxx]` seen in the game; `[code]` read in the exe; `[data]` read in the game files; `[inference]` reasoned;
`[untested]` an idea only.

| Claim | How we know | Test or source |
|---|---|---|
| A 131-character text with 771 / 791 px lines crashed the game as the window opened, in two installs | live | B41 runs 1 and 2 |
| The crash is `std::length_error` "string too long", then abort 0xc0000409, fast-fail 7 | live (dump), code | B41; +0xc39b1c, +0x15576f8, +0xefa30 |
| The string routine throws above 127 characters | code | `FUN_1400efa30` (+0xefa30) |
| The window's string has a fixed 128-element buffer | code | destructor `FUN_140198600` (+0x198600) |
| The same routine is reached from the YES/NO handler | code | `FUN_1401987f0` -> `FUN_140198270` -> `FUN_1400efa30`; return addresses +0x19882a and +0x198326 |
| Four sibling handlers share the text-store step | code | +0x198720, +0x198920, +0x198a10, +0x198f90 |
| Which command reaches which sibling | not traced | open |
| The replace routine for `[ChrName]` has the same 127 check | code | `FUN_14019aa50` (+0x19aa50) |
| Width and line count are not checked in this path | code (this path only) | the same functions |
| `kind` 1 and 2 format into a 128-element stack buffer first | code | `FUN_1401984c0` (+0x1984c0) |
| 124 characters, 759 px worked in YES/NO windows | live | B20d, B41 run 3, B43 |
| English lines up to 808 px, 112 characters worked in OK windows | live (a note in the export tool) | B32 |
| Newline is one UTF-16 unit | data | an English `mapevent.fmg` |
| Glyph advance is at record +14 of the `.ccm`; header fields | data | `FeFont_Big.ccm`, English and Chinese |
| The glyph gaps in the table | data | fonts of the 11 PC languages, 17 Nov |
| What the game draws for a missing glyph | not seen | open |
| 266 px fits the title label, 353 px wraps | live | B48 |
| 571 px row shown fine | live | B36c |
| Japanese limit 673 px | data, estimated with the Chinese font's advances | `tools/japanese.py` |
| Japanese texts in the game | not seen | open |
| The game's own YES/NO and OK texts are at most 48 characters | data | event and talk scripts, 17 Nov |

## Tools

All in the project's `tools/` folder.

| Tool | What it does for this topic |
|---|---|
| `tools/i18n.py` | `advances()` reads a language's font; `px()` measures a text; `fits()` applies the window rule (3 lines, 759 px, 124 characters); `missing()` lists characters a language's fonts lack; `safe()` swaps the known gaps; `wrap()` breaks lines by width; `window_px()` gives Japanese its 673 px; `check()` and `check_names()` test every translation. `WINDOW_CHARS` (124) is the project's margin; the engine's limit is 127 |
| `tools/arenasetup.py` | `texts()` refuses a text that does not fit before it is written |
| `tools/textexport.py` | Writes every text the release changes, one file per language, with the width of each line against the limit of the place it shows in, and a `!!` mark on a line that is over or a text that fails `fits()` (3 lines, width, 124 characters) |
| `tools/japanese.py` | Japanese originals from the PS3 disc, and `glyphs()`, the set of characters the Japanese font draws |
| `tools/version.py` | The title label and `LABEL_PX` (266); refuses a longer label |
| `tools/offline_tests.py` | A test that the crashed wording would fail `fits()` and every shipped English window text passes |

## Open questions

- **Is 127 the edge?** Does a text of exactly 127 characters open, and 128 crash? The code says so; no live test.
  [untested]
- **Which windows share it?** Trace the dispatcher from command id to handler for `1:130443` (OK) and the event-message
  window. The handlers share the store step, so the limit very probably holds. [untested]
- **What a too-wide line does.** Does the YES/NO window wrap, clip, or grow? We never made one wider than 759 px. The OK
  windows took 808 px. A live test with one line at 800, 900 and 1 000 px would settle whether 759 is a real edge.
  [untested]
- **What a missing glyph draws,** and whether the front end uses `FeFont_Big` for the YES/NO window. [untested]
- **The formatted forms.** What happens when a `kind` 1 or 2 text formats to more than 127 characters. [code, not read to
  the end]
- **How the game shows its own 132- and 136-character texts** (mapevent 2304, 4012), which is longer than the window
  limit. Another window, with another string? [inference]
- **Japanese in the game,** and the title label in other languages. Both only measured. [untested]
- **Other users of the string type.** Three more functions call the same assign routine. If any is a window or a menu
  you can put text into, it shares the 127 limit. [untested]

## Related pages

- [Asking the player a question: YES/NO windows](yes-no-windows.md): the window this crash happens in, and its commands
- [Adding dialogue options: the N-choice list window](adding-dialogue-options.md): the list, its 12-row limit and label width
- [Vendor shops](vendor-shops.md): the page-name windows (the 808 px lines)
- [Diagnosing crashes from minidumps](../modding-scripts/crash-diagnosis.md): the general method
- [Adding and translating game text](game-text-and-languages.md): the `.fmg` files, ids and languages
- [Script command reference](../modding-scripts/script-command-reference.md): the window commands among the others we proved
