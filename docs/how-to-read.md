# How to read this site

## Names and codes

| Term | Meaning |
|---|---|
| **NT** / Network Test | The PlayStation 3 Network Test of Dark Souls II, August to October 2013. Western build **NPUB90979**, Japanese build **NPJB90625**. |
| **Retail v1.00** | The launch PlayStation 3 disc, US release **BLUS41045**. |
| **Scholar** | *Scholar of the First Sin*. PS3 build **BLUS31528**, PC build 1.0.3.0 (Steam). |
| **PC vanilla** | The original Windows release, builds 1.01 to 1.12, plus a pre-release build dated 31 March 2014. |
| **Asia / Japan discs** | **BLAS50640** (Asia) and **BLJM61113** (Japan) - the same game as US v1.00 in other regions. |
| **Params** | The game's data tables (weapons, enemies, items, spells) that most findings are read from. |
| **c1234** | A character (enemy, NPC or player) model id. |
| **m10_16** | A map id. |

## Things you will run into

- **Ids and numbers.** Item, enemy and event ids are given as the game stores them, so you can look them up. Where a
  name is a developer working name rather than a shipped one, the page says so.
- **Translations.** Japanese developer text is translated by an AI assistant and checked where the game's own English
  exists. Treat the translations as good guesses.
- **Dates.** Pages carry the date the work was done. Newer pages sometimes correct older ones.
- **Scripts.** Pages sometimes name small scripts the notes were produced with (for example `tools\something.py`). The
  scripts are not published here; the pages are written so the finding stands without them.
- **Images.** Some pages mention renders of game models and icons. Those images are not published.

## The modding pages

The three sections called *Modding* (and two pages in *Tools and patches*, the [memory map](tools/memory-map-pc.md)
and the [teleport recipe](tools/teleport-and-facing.md)) are
written for people making their own mods of the **PC** game, *Scholar of the First Sin* 1.0.3.0. They come from
building one mod, the Boss Arena, so each result says which test showed it. Unless a page says otherwise, everything on
them is for that PC build.

- **Test ids.** Ids such as A7, B36c or B57 point to the author's own test log, which is not published. They are there
  so you can tell a result that was seen in the game from one that was only read from files.
- **Evidence tags**, in square brackets: `[live B36c]` seen working in the game; `[code]` read in the game's code (an
  exe address is given); `[data]` read in the game's files; `[inference]` reasoned but not proven; `[untested]` an idea
  that was never tried. The status label at the top of a page still applies to the page as a whole: a page marked
  *Confirmed live* can hold single claims tagged `[inference]`.
- **Dated tags.** A tag with a date instead of a test id, such as `[live 30 Oct]`, is a result seen in the game before
  test ids were kept. "The trainer" is the author's own memory-editing tool, which keeps a log of the values it wrote.
- **Script commands** are written `1:132125(...)` (a command that does something) and `f130421()` (a function that reads
  state). Their names are the developers' own debug labels, mostly Japanese, given with a short English gloss. The
  ones the mod used are listed on [Event and talk script commands](modding-scripts/script-command-reference.md).
- **Addresses.** `+0x1234` is an offset from the start of the game's exe, or from the start of the object the page names.
  In Ghidra, add `0x140000000` to an exe offset.
- **Mod tools.** Pages name small scripts the mod was built with (`tools/something.py`). They are not published; the
  pages give the formats and rules, so the findings stand without them.

## What this site is not

- It is not a guide to playing the game, and it is not a replacement for the community wikis.
- It publishes findings, not game files. There are no extracted game assets, text dumps, patches for the game files,
  or copies of the game here.
