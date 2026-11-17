# When the cut talk files (girl + dealer questline) were removed

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author, 2026-10-23): the m20_04 talk text (the little girl and the man who sells her, see
`..\..\DS2 Scholar\findings\cut-content-scholar.md` and `cut-dialogue-translation.md`) is in v1.00 and gone from
Scholar: how early was it cut?

## Answer

**Between 24 February and 1 April 2014: patch 1.03 is the first build without it.** The game came out on 11 March
(US) / 13 March (Japan), so the leftover files were cleaned out within about three weeks of launch.

The questline itself was cut much earlier: no build has a map for its "village", and the talk files are already
byte-identical in the Japanese network test (August 2013), the Western test and v1.00, with Japanese text only
(the other languages' files are empty). What 1.03 removed is the dead text left behind.

## Evidence

Every patch's `PatchData` lists whole categories, unchanged files included, either as byte-identical copies or as
0-byte stubs that fall back to the disc (1.01: 1,368 copies and 1,442 stubs among 3,054 files; see
`v1.00-vs-1.01.md`). The cut talk files were full copies in 1.01 / 1.02 and are not listed at all from 1.03. So a
file that stops being listed while its neighbours carry on was deleted from the developers' build.

| Build | Data build date (archive stamp) | Cut talk files shipped | Byte-identical to v1.00 |
|---|---|---|---|
| JP network test | Aug 2013 | yes | (v1.00 = these) |
| v1.00 disc | 27 Jan 2014 | yes (GameData) | — |
| 1.01 | 24 Feb 2014 | **yes**, 31 files: m10_21 / m20_04 / m90_00 in 10 languages + m99_97 Japanese | yes, all 31 |
| 1.02 | (PatchData byte-identical to 1.01's; new EBOOT + regulation only) | yes | yes |
| 1.03 | **1 Apr 2014** | **no** | — |
| 1.04-1.07 | not obtainable (never archived) | — | — |
| 1.08 | 9 Sep 2014 | no | — |
| 1.09 | 8 Oct 2014 | no | — |
| 1.10 | 26 Nov 2014 | no | — |
| PS3 Scholar disc | 26 Nov 2014 | no (deleted from GameData too) | — |

1.01 -> 1.02 stops shipping nothing; **1.02 -> 1.03 stops shipping exactly 51 files, all text**: the cut talk
files m10_21 (old Shanalotte dialogue), m20_04 (girl + dealer), m90_00 ("cutscene test text") and Japanese m99_97,
plus `bofireName.fmg` ("Test") and `regulation.fmg` ("The letters are worn beyond recognition."), in every
language. Every other talk file is still shipped (275). Later patches only add files. So 1.03 was a single,
deliberate clean-up of unused text, done alongside its other changes (it also starts shipping the player's TAE files
c000100_pl/_sfx/_snd and drops 5 params from the regulation, 252 -> 247).

## Caveats

- A patch cannot delete disc files: on a PS3 with 1.03+ installed, the v1.00 disc's copies are still in GameData,
  just never referenced. The text is physically gone only from the Scholar disc, whose GameData was built from the
  same tree as 1.10, 78 minutes later on 26 Nov 2014.
- Missing 1.04-1.07 does not matter: the removal is already in 1.03, and 1.08 / 1.09 / 1.10 confirm it stayed out.
- PC Dark Souls II, checked 2026-10-27: no PC build has the files, from a pre-release build on 31 Mar 2014 on. That
  build's regulation is 1.04, the same as PS3 1.03's. So the cut was in the tree by 31 March and came before the PC
  version. See `..\..\DS2 Vanilla PC\findings\cut-text-on-pc.md`.

Scratch tools used (session scratchpad, not kept): talkscan.py (talk files per PatchData), patchdiff.py (what each
patch stops / starts shipping), patchinfo.py (stamps, regulation versions).
