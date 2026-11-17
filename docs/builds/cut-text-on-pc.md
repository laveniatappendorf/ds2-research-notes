# Was the text PS3 1.03 cut ever on PC?

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author, 2026-10-27): PS3 patch 1.03 stopped shipping the leftover text from the cut girl + dealer questline
and some test text (`..\..\DS2 Vanilla Patches\findings\cut-talk-timeline.md`). Was that cut made before the PC
version, or did the PC version still have the files?

## Answer

**Before PC. No PC build on Steam has any of the files, starting with a pre-release build from 31 March 2014.** The cut
was made in the console tree between 24 February and 31 March 2014. The oldest PC build carries regulation 1.04, the
same as PS3 1.03 (97 % of comparable rows identical). Launch build 1.01 (22 April) carries 1.05. So PC was built
from the same tree as PS3 1.03, after the clean-up. The early builds are covered in `early-pc-builds.md`.

| Build | Data / exe date | Regulation | Cut text files |
|---|---|---|---|
| PS3 1.01 / 1.02 | 24 Feb 2014 | 1.02 / 1.03 | all 51 |
| **PC pre-release** (manifest 6454945520062524878) | exe 31 Mar 2014 11:49 UTC | **1.04** | **none** |
| PS3 1.03 | 1 Apr 2014 | 1.04 | none |
| PC 1.01 launch (manifest 7916522719781015027) | exe 22 Apr 2014, public 24 Apr | 1.05 | none |
| **PC 1.02** (manifest 1803710700711034863) | exe 25 Apr 2014, public 28 Apr | **1.05** | **none** |
| PC 1.03 (manifest 1212557563318387325) | exe 4 Jun 2014, released 16 Jun | 1.07 | none (data byte-identical to 1.02) |
| PC 1.12 (installed) | exe 13 Oct 2022 | | none |
| PC SOTFS 1.0.3.0 | 2022 | 2.02 | none |

The files: talk `m10_21` (old Shanalotte dialogue), `m20_04` (girl + dealer), `m90_00` ("cutscene test text"),
`m99_97` (Japanese only), plus `bofireName.fmg` ("Test") and `regulation.fmg` ("The letters are worn beyond
recognition."). That's 5 files in each of PC's 10 languages; PC has no Japanese folder, so `m99_97` was never
expected.

## Why the missing files mean "deleted", not "left out by the PC packer"

PC Scholar was packed from references (unused models and AI scripts dropped; `..\..\DS2 Scholar PC\findings\cut-content-pc.md`),
so I checked that launch-day PC wasn't packed the same way:

- **Unused models stay on launch PC.** It still has c2020 (model only), c7015 (child Emerald Herald), c1400, c7210,
  all the unused Peasant set 2481 armour, wp_5510 and objects o00_9111 / o14_6000 / o23_9050. PC 1.12 still has
  them too, so the reference-based purge only came with Scholar.
- **Every PS3 text file with real text is on PC except the cut ones.** In PS3 1.01, 66 English FMGs have data. 61
  are on PC 1.02. The other 5 are the cut set: `bofireName`, `regulation` and talk `m90_00` hold one entry each, and
  `m10_21` / `m20_04` are header-only in English (their dialogue is Japanese only).
- **The 13 per-map English text files PC lacks are 0-byte files on PS3** (blood messages / talk for maps with nothing
  to say). The PC packer skipped empty files, which is unrelated.
- **The regulation message text didn't vanish.** It lives in the regulation binder as `regulation<Language>.fmg`
  (Japanese included), on PS3 and PC alike. The PC exe still names `menu:/Text/%s/regulation.fmg`. The GameData copy
  that 1.03 deleted was a stale duplicate.

Of the 200 PS3 1.03 files with data that launch PC lacks, all but 83 have plain reasons: 67 are Japanese text (PC has
no Japanese), 50 are the cut text, and 4 are PS3-only or trivial (`icon0.png`, the Japanese font sheet,
`c2020.texbnd`, `defaultcubeenv.gtf`). The remaining 79 are TAE files.

## A side finding: launch PC dropped 79 TAE files

79 `timeact` files (71 character files, mostly `_sfx` / `_snd` / `_pl`, for 49 character ids such as c100001,
c106010, c118010, c205010, c309010 and c330000-c330050; plus 8 objects) are on the PS3 disc, in PS3 1.10's DlcData and on the
Scholar disc. They're absent from PC 1.02 and PC 1.12 (one is back in 1.12). PS3 shipped them all the way to Scholar,
so this is PC packaging, not a dev-tree deletion: launch PC pruned some animation-event files even though it kept
unused models. Which characters they belong to, and whether any is used, is still to check.

## Launch PC text vs PS3 1.03

`tools\pc_text_compare.py ps3:v103 1.02 ps3v103_to_pc102` (English, `text_ps3v103_to_pc102.tsv`, 340 entries). Apart
from the cut files, PC adds `win32onlymessage.fmg` (288 PC messages) and changes 7 files. There's no PC-only story
or item content:

- `charamaking` (76): trailing full stops dropped from the sliders' help lines.
- `titleflow` (129) / `titlemenu` (13): PS3 system, save and network-test messages removed or reworded; PC licence text.
- `common` (6): Xbox wording "gamer card" / "gamertag" became "profile" / "Player". The PS3 English file had the Xbox terms.
- `staffroll` (108): PC port credits.
- `detailedexplanation` (2): the Spirit Tree Shield's "god of dream" capitalised to "God of Dream" (11370000), and
  the Dragonrider gauntlets' first line re-wrapped (26100102).
- `talk\m10_15` (Aldia's Keep, 6): five "(no line)" placeholder entries dropped and one changed.

## PC 1.02 -> 1.03 (16 June 2014)

Only the exe and the regulation changed (manifests: the other 320 files are byte-identical). Regulation 1.05 -> 1.07:
403 rows in 15 params (`..\..\DS2 Vanilla Patches\findings\regulation_pc102_to_pc103.tsv`): parry windows on 14
weapons, player attack damage / durability (71 rows), 45 weapon attack motions, DEX scaling on 10 curves, defence
stat curves (99 rows), HP of 2 enemies. All of these actually shipped on 2 June as regulation 1.06; 1.03 only added
the exe and one event file (`early-pc-builds.md`).

`regulation_compare.py` PS3-vs-PC output is still noise: vanilla PC params are little-endian with the old 12-byte row
table and PS3-era layouts, but `paramdef.py` gives little-endian files PC Scholar's layouts.
`regulation_ps3v103_to_pc102*.tsv` is therefore noise. Comparing decoded rows directly, limited to the 175 params
whose layout fits both platforms, does work; `early-pc-builds.md` uses that.

## Correction: which text changes are pre-launch PC edits

The 31 March build's GameData was downloaded later the same day (`early-pc-builds.md`). Its English is already PS3 1.03
minus the cut files, plus `win32onlymessage`, "profile" and the PC title-screen text. But it still has PS3's credits,
the charamaking full stops and the two old item descriptions. Those were changed between 31 March and 1.01.

## Method

`tools\pcindex.py` (probe / diff / list): reads the RSA-encrypted BHD5 indexes, names entries by hashing every PS3
build's file names plus UXM's Scholar dictionary, and maps vanilla PC's Hq archives as `model_hq\<model path>`
(names 100% of HqChr / HqMap / HqObj / HqParts). Only the indexes, keys, exe and regulation of each build were needed
(`tools\index-files.txt` as DepotDownloader `-filelist`, about 20 MB a build).
