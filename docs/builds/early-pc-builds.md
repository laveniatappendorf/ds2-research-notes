# The earliest PC builds (31 March to 16 June 2014)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-27. The author found four depot 236432 manifests older than 16 June 2014 on SteamDB (signed in). All four still
download. For each one I fetched the archive indexes, keys, exe and regulation (`tools\index-files.txt`, about 20 MB
a build). 1.02 is downloaded in full. From the 31 March build I also fetched `GameDataEbl.bdt`: 3.58 GB downloaded,
4.61 GB on disk, SHA-1 matches the manifest. Dates and manifest IDs are in `..\BUILDS.md`.

## What changed between builds (whole-file level, from the manifests)

| From -> to | Files that differ |
|---|---|
| 31 Mar pre-release -> 1.01 (22 Apr) | 27: exe, regulation, every archive and key (re-keyed), the three FMOD DLLs, six European readmes |
| 1.01 -> 1.02 (25 Apr, public 28 Apr) | the exe only |
| 1.02 -> regulation 1.06 (28 May, public 2 Jun) | the regulation only |
| regulation 1.06 -> 1.03 (4 Jun, public 16 Jun) | the exe, and the regulation (one event file) |

## 31 March pre-release

- The exe was linked on 31 Mar 2014 11:49 UTC and has no version resource. The manifest was made at 12:42. Its regulation is
  **1.04**, the same number as PS3 1.03's (data 1 Apr 2014).
- **None of [the 51 text files PS3 1.03 cut](cut-talk-timeline.md)** are in it, in any of the 10 languages. The control file is present in
  all 10. So the cut was already in the tree the PC build came from on 31 March, a day before PS3 1.03's data date.
- Its regulation is PS3 1.03's in PC form. 213 of the 220 params both have share identical row IDs. 175 params
  decode on both platforms (their row layout fits both), and in those, 35,218 of 36,331 shared rows (97 %) are
  identical. Against PS3 1.08 the figure is 81 %. The other 1,113 rows are in 40 params. Some are plainly decoding
  artefacts: every row of ItemUsageParam and MapObjectBonfireParam differs, which looks like byte order rather than
  edits. I haven't checked them one by one.

### 31 March -> 1.01: GameData, file by file

Every file in both builds was decrypted, unpacked and compared byte for byte (`tools\pcindex.py cdiff 2014-03-31
1.02`, saved as `gamedata_pc0331_to_pc101.txt`; 1.01's data is byte-identical to 1.02's).

**63 of 11,855 files changed, and 8 were added. Nothing else changed.**

- 1,416 more files differ only in the last 2-14 bytes of an encrypted file: 577 AI scripts, 224 map params, their
  `.edd` debug files and 55 unnamed files. That's the random padding of the last AES block, which changes every time
  the archive is packed. Every one of those AI scripts and params parses to identical contents, and `cdiff` now sets
  them aside.
- **HqParts** (index only): the high-quality legs `LG_1470_F` / `LG_1470_F_L` changed size. Its archive wasn't
  downloaded, so same-size changes there and in HqChr / HqMap / HqObj can't be ruled out.

What the 63 are:

- **Text, 57 files** (`text_pc0331_to_pc101.tsv`, all 10 languages, 1,287 entries), below.
- **Two event scripts** (`esd.py diff`; [the commands](../modding-scripts/script-command-reference.md) have no names on record):
  - `event_m10_02` (Things Betwixt), event 5000 (objects 10021400 / 102000030): a step is wrapped in a new pair of
    commands that set `1:132242` and `1:130432` to 1, then back to 0, with an extra check on `f132202()`.
  - `event_m10_32` (Shaded Woods), event 9000 (objects 10320405 / 10320406): a new routine resets both objects
    (`1:131638(x, 0)`) and plays animation 74 on both (`1:131636(x, 74)`). Another routine now waits on
    `f131653(x, 10, 0)` before going on.
- **Prefab `o00_2240`**: 144 B larger, not examined.

### 31 March -> 1.01: text

The 31 March English text is PS3 1.03's, minus the 51 cut files, plus PC wording that was already in place:
`win32onlymessage`, "gamer card" -> "profile" in `common`, the PC title-screen messages, and the trimmed Aldia's
Keep placeholders (`text_ps3v103_to_pc0331.tsv`). Its credits were still the PS3 ones. What changed by 1.01:

- **Credits** (`Staffroll`, every language): the PC port credits replaced the console credits (28 added, 40 removed,
  40 changed).
- **English**:
  - `charamaking`: the trailing full stops on 76 slider help lines were dropped.
  - Two item descriptions: Spirit Tree Shield "god of dream" -> "God of Dream", and the Dragonrider gauntlets'
    first line re-wrapped.
  - The EULA got "DARK SOULS Ⅱ" and a ™.
  - One title-screen line re-wrapped.

  `ItemName` changed size only because different empty, nameless slots are listed.
- **Six European languages** (French, German, Italian, Polish, Russian, Spanish): `IngameSystem` 10005 changed from
  the console message "the storage device first selected can't be found" to "failed to save the game". The same
  text was added as `TitleFlow` 900004, and a save-failure message was re-worded.
- **Translation fixes**:
  - French Majula dialogue `talk\m10_04` 75101300 was **in German** ("Ah, ja. Es gibt da etwas, das ich Euch
    erzählen wollte.") and became French.
  - French "Titatine Scintillante" -> "Titanite Scintillante".
  - Portuguese:
    - bonfire 17665 was "Purgatório dos Mortos-Vivos" (Undead Purgatory), now "Piscina Venenosa" (Poison Pool);
    - the Second / Third Dragon Ring lost stray "+1" / "+2" suffixes;
    - two gesture names were swapped;
    - Spirit Tree Shield -> "Grand Spirit Tree Shield";
    - "Alma o Antigo Rei de Ferro" gained its "do";
    - 5 descriptions fixed, and a button guide added to `npcMenu`.
  - Russian: the "DLC not found" warning (`TitleFlow` 230211) said **add-on areas** would be unavailable. It now
    says some items will be, as the other languages always did.
- **Not examined:** the other changes to `win32onlymessage` (Chinese 4, French 1, German 15, Italian 3, Polish 13,
  Russian 3, Spanish 3) and `TitleMenu` (French 17, Spanish 17, Korean 2, Italian 1).

So several things I'd put down as "PC vs PS3" in [cut text on PC](cut-text-on-pc.md) actually happened between 31 March and 1.01: the
full stops, the credits and the two English item descriptions.

### 31 March -> 1.01: regulation (`..\..\DS2 Vanilla Patches\findings\regulation_pc0331_to_pc101.tsv`)

The regulation changed in 5 params: 12 rows changed, 197 added, 10 removed.

- **Soul Geyser nerfed before launch.**
  - SpellParam 31170000 (and its twin 71170000): startup 0.5 -> 0.75 s, stamina 2 -> 4, cast speed 1.1 -> 0.95.
  - Its four attack rows (31170010-13): damage multiplier 2.25 -> 1.3, damage floor 145 -> 88, stamina damage
    930 -> 620.
  - On PS3 the strong version lasted through 1.03. PS3 has the same nerf by 1.08, except that its stamina cost is 3,
    not 4.
- **New rows:** 187 PlayerDamageParam rows (20083300 to 35310004) and 10 SpellParam rows (31300000 to 35310000).
  PS3 has all 10 spell rows and 174 of the 187 attack rows by 1.08.
- **Two enemy bullets' tracking:** EnemyBulletParam 110501210 and 111800910.
- **EnemyCommonParam:** 3 rows changed and 10 removed. EnemyParam 502009 points to a different common row. The removed
  rows are DLC placeholders (next section).

## DLC enemies in the spring 2014 builds

### EnemyCommonParam rows for 9 DLC enemies (PS3 1.03 and PC 31 March; gone from PC 1.01)

[PS3 1.03's regulation](1.01-to-1.03.md#102-103-data-scripts-and-a-second-balance-pass-calibrations-103-104) (1 Apr 2014) **added** EnemyCommonParam rows at the IDs that PC Scholar's EnemyParam uses for
these DLC enemies. v1.00 doesn't have them. The 31 March PC build has them, PC 1.01 removed them, and PS3 still has
them in 1.08.

| Row | PC Scholar's EnemyParam name for that ID | DLC |
|---|---|---|
| 650000 | Iron Warrior (Smelter Hammer), Brume Tower | Old Iron King |
| 651000 | Fume Sorcerer, Brume Tower | Old Iron King |
| 653000 | Ashen Warrior (Axes), Brume Tower | Old Iron King |
| 654000 | Ashen Crawler | Old Iron King |
| 656000 | Possessed Armor, Brume Tower | Old Iron King |
| 657000 | Barrel Carrier, Brume Tower | Old Iron King |
| 658000 | Alsanna, Silent Oracle | Ivory King |
| 659000 | Rampart Golem, Frozen Eleum Loyce | Ivory King |
| 660000 | Crystal Golem | |

Row 9400, also removed, is older: it's already in v1.00. So the console data already reserved slots for enemies of
the second and third DLC by late March 2014, months before those DLCs came out (August and September 2014).

### DLC enemy AI on launch PC

Between 31 March and 22 April, PC gained `ezstate\ai650000`, `ai650010`, `ai671000` and `ai671010` (.esd + .edd).
The developers' labels inside them, and the enemy that uses that ID in PC Scholar:

- **ai650000 / ai650010:** `下位 / 上位：炎を吐く鎧の騎士`, a lower and upper "fire-breathing armoured knight". This
  is the **Iron Warrior** of Brume Tower (EnemyParam 650000; character c6500; Crown of the Old Iron King).
- **ai671000 / ai671010:** `下位 / 上位：こけし背負い`, a lower and upper "kokeshi carrier": the **Poison Statue
  Cluster** of Shulva (EnemyParam 671000; character c6710; Crown of the Sunken King). The petrifying statues on its
  back are the "kokeshi". PC Scholar's unplaced object `o99_0036` is c6710 with kokeshi textures
  ([cut content in PC Scholar](../network-test/cut-content-pc.md)).

Neither character's model is in PC 1.02 or 1.03, so on launch PC these were orphan scripts. On PS3, the AI and models
first appear together in 1.08's DlcData.

The launch versions are **early drafts**. Launch PC's script format is `fSSL` (little-endian, 32-bit), and
`esd.py` now reads it.

| Script | Launch PC (Apr 2014): groups / states | Shipped DLC (PS3 1.10 = PC Scholar): groups / states |
|---|---|---|
| ai650000 lower Iron Warrior | 17 / 205 | 48 / 672 |
| ai650010 upper Iron Warrior | 29 / 303; 10 sub-logics (96500010-14, -20-24) | 17 / 164; 2 sub-logics (96500010, 96500020) |
| ai671000 lower Statue Cluster | 3 / 12 (a stub) | 12 / 135, with six new 13-state behaviour groups (100-102, 110-112) |
| ai671010 upper Statue Cluster | 21 / 221; sub-logics 96710010-13 | 16 / 154; adds 96710020 |

The finished lower Iron Warrior script also has four speed labels the draft lacks (`高速 / 中速 / 低速 / 極低速`, fast
to very slow). Line diffs are in the scratch folder of this session (`dlcai\`), not kept here.

## 1.02 -> 1.03

The "1.03" balance changes reached players on **2 June 2014** as a regulation-only update (1.06, same exe as 1.02).
All 403 changed rows, 2 new and 188 gone, match 1.02 -> 1.03 exactly. 1.03 itself (16 June) brought the 1.0.3.0
exe and regulation 1.07. 1.07 differs from 1.06 only in `SpEffectSpell.emevd`: no param changed.

## Not looked at

- The two `LG_1470` leg models (HqParts, 2.6 GB, not downloaded).
- Prefab `o00_2240`.
- What the event-script commands do in game.
- The 1,113 PS3-vs-PC regulation rows that differ.
