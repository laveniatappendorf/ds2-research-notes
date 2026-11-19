# Dark Souls II PS3: Asian disc (BLAS50640, En/Zh/Ko) vs US disc (BLUS41045)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Broad pass, 2026-10-27, after the Japanese disc ([Japanese disc vs US disc](jp-disc-vs-us.md)). Result: **the Asian
disc is the US v1.00 game, byte for byte.** Only the disc identity and the server list's last field differ.

## 1. Files

- Same 152 files as the US disc. Against the **original** US disc (`..\..\DS2 Vanilla\iso-backup`, SHA-1 b17a8e7c...),
  **147 are byte-identical**: EBOOT, regulation, GameData.bdt / .bhd, InstallData.bdt / .bhd, sound, movie, fonts,
  shaders, effects, trophies, the XMB pictures, the system update.
- The 5 that differ are the disc identity: PARAM.SFO, LIC.DAT, PS3_DISC.SFB, and the two server lists.
- Checked file by file through the archive too (`..\..\DS2 Japan\tools\build_compare.py vanilla asia`): all 11,457
  files identical; `bdt_gaps.py`: not one byte of GameData.bdt differs, gaps included.
- Pitfall: the US ISO in RPCS3's `games` folder is **not** the original disc any more: its GameData.bdt carries the
  showcase patches (`..\..\DS2 Vanilla\findings\showcase-patches.md`). A whole-disc hash against it shows a false
  GameData.bdt difference. Compare against `DS2 Vanilla\GameData.bdt` (copied before the patches) or the backup ISO.
- Text: the "(En,Zh,Ko)" label is packaging. The data holds all 11 languages, exactly as the US "(En,Fr,Es)" disc does.
  As the same program runs both, the menu languages on offer cannot come from the disc data; whether the code picks
  them by console language or by title id is not checked.

## 2. Disc identity

| | Asia | US | Japan |
|---|---|---|---|
| TITLE_ID | BLAS50640 | BLUS41045 | BLJM61113 |
| PARAM.SFO VERSION (disc revision) | **01.00** | 01.01 | 01.02 |
| APP_VER | 01.00 | 01.00 | 01.00 |
| PARENTAL_LEVEL | 9 | 5 | 8 |
| disc image created (UDF, probably UTC) | **27 Jan 2014 09:29** | 29 Jan 04:36 | 31 Jan 09:40 |
| EBOOT / GameData file dates | 27 Jan 06:39-06:44 | 29 Jan 03:40-03:49 | 31 Jan 07:52-08:24 |
| server list | `UP0700-BLUS41045_00`, `0x01023100` (18 Jan 21:00) | `UP0700-BLUS41045_00`, `0x01021100` (18 Jan 20:57) | `JP0506-BLJM61113_00`, `0x01020102` (25 Jan) |

- **The Asian master was made first** (disc revision 01.00, 27 Jan, the same day the shared game data was packed); the
  US master re-used the same program and data two days later (01.01), and Japan followed on 31 Jan with newer data
  (01.02). The disc revision numbers follow the mastering order.
- **Online**: Asia logs in to the same host as the US under the **US online title** (`UP0700-BLUS41045_00`), unlike
  Japan, which has its own. The last field differs (US `...1100`, Asia `...3100`, Japan `...0102`); PS3 Scholar's is
  `0x0102110A`. It looks like a protocol / region code (US `..11..`, Asia `..31..`), so the server can probably tell
  Asian and US players apart; not checked in code.
- Both programs (US = Asia, and Japan) name their downloadable-content folder `BLJM61113DOWNLOADCONTENTS`: every
  region keys its DLC folder to the Japanese title id. Not a region check.

## Open

- How the program chooses the menu language (console language vs title id).
- What the server list's last field means.
