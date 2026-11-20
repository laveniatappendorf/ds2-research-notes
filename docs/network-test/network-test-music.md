# Network Test music: what is in the sound files (2026-10-23, redone 2026-11-18)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

> **Redone 2026-11-18.** The first conversion (23 Oct) read each sample's data offset at half its real value (×16
> instead of ×32). So 47 of its 54 music files were cut in the wrong places (half of one track plus half of the next),
> and its lengths and duplicate list were wrong. Everything below comes from the redo,
> `tools\fsb_ysplit.py batch` (Reproduce, at the end). Checks: for all 141 files the FSB header length, the counted MP3
> frame / PCM length and the decoded length (FFmpeg via PyAV) agree within 0.2 s; none is silent; the 76 ambience files
> October made are byte-identical to the new ones. October's files are kept in
> `..\..\DS2 Network Test\music_old-miscut-2026-10-23\` (safe to delete).

Question: what are the names of the BGM tracks in the network test?

**There are no track names.** No text file in the Western or Japanese build titles a piece of music (the only hits for
"music" are the volume option labels), and the sound banks name tracks by number only. What can be given is a map-by-map
list of the tracks, with their lengths, and the tracks themselves as playable files:

**`<local path> Network Test\music\`**: `music\` (46 map tracks + 12 `smain` sounds), `ambience_y\` (the `y` streams,
83 files), and `music.tsv` / `smain.tsv` / `ambience_y.tsv` (bank, sample, file, codec, rate, channels, header length,
length, SHA-1 of the sample data, and "same as" for byte-identical samples). File names are `<bank>_<event>.mp3`, so
`sm1002_m100200001.mp3` is event `m100200001` in bank `frpg2_sm1002`.

## How the files are laid out

- Sound lives in FMOD banks in `sound\`. Each `.fev` event file names events; each `.fsb` (FSB5 format) holds the audio.
- **`frpg2_sm<map>.fsb`** holds the map's streamed tracks. Events named **`m<map>0000n`** are the music; `a…` ambience loops,
  `v…` voices and `o…`/`s…`/`c…` effects share the banks. Calling the `m…` events the music is my reading of the naming
  and the track lengths (2 to 7 minutes); I have not listened to them. Four `sm` banks have no music: `sm1034`
  (ambience `a103410001` 1:16 + effect `o341800100`), `sm2062` (ambience `a206210001` / `a206210006` + effect
  `s000003650`), `sm9000` / `sm9997` (short voices).
- **`frpg2_xm<map>.fsb`** holds one `y<map>00000` stream per map, 6 channels: three stereo MP3 streams interleaved frame
  by frame. The third stream is silent in 37 of the 40 banks (every frame codes no audio) and is left out; in `xm1022`
  and `xm1026` (48 kHz) it has sound and is kept as `_C`. What the streams are (ambient layers is a guess) is unknown.
- **`frpg2_smain.fsb`** holds the system tracks: title / lobby music, jingles, and four short effects.
- Almost all audio is MP3 at 44.1 or 48 kHz. Three banks are 16-bit PCM, written as WAV: `sm2011` (two stereo tracks),
  `xm1001` (6 channels, 48 kHz: three audible stereo pairs) and `xm2034` (6 channels at 27,403 Hz: two pairs, the third
  all zero). Byte order was picked by waveform smoothness; all three came out little-endian.
- The FSB header's sample counts are right for every sample, and every sample starts on a whole MP3 frame. (October's
  "header too small" and "offsets a few hundred bytes off" notes were the offset bug.)
- A rate in a sample's extra header chunk (type 2) overrides the rate index: `sm2011` `m201100002` = 35,109 Hz and
  `xm2034` = 27,403 Hz come from there.

## Music by map

Map names are **retail's** `MapName` text (the demo names only Huntsman's Copse and Undead Purgatory). A map named here
is the map id's retail name; the demo's own version of that map may differ or be unfinished.
Banks `sm20xx` belong to maps `m20_xx`. "Same audio as" = byte-identical sample data.

| map id | retail name | bank | track (event) | length | rate | file |
|---|---|---|---|---|---|---|
| m10_01 | (no retail name) | `sm1001` | `m100100001` | 3:08 | 44.1 kHz | `sm1001_m100100001.mp3` |
| m10_02 | Things Betwixt | `sm1002` | `m100200001` | 2:42 | 44.1 kHz | `sm1002_m100200001.mp3` |
| m10_03 | (no retail name; a [working map number](../unused/unused-map-pieces.md#patterns)) | `sm1003` | `m100300001` | 2:24 | 48 kHz | `sm1003_m100300001.mp3` |
| m10_03 | (no retail name) | `sm1003` | `m100300011` | 3:27 | 48 kHz | `sm1003_m100300011.mp3` |
| m10_04 | Majula | `sm1004` | `m100400001` | 3:18 | 44.1 kHz | `sm1004_m100400001.mp3` |
| m10_04 | Majula | `sm1004` | `m100400002` | 2:24 | 44.1 kHz | `sm1004_m100400002.mp3` |
| m10_10 | Forest of Fallen Giants | `sm1010` | `m101000001` | 2:55 | 44.1 kHz | `sm1010_m101000001.mp3` |
| m10_10 | Forest of Fallen Giants | `sm1010` | `m101000002` | 2:24 | 44.1 kHz | `sm1010_m101000002.mp3` |
| m10_11 | (no retail name) | `sm1011` | `m101100001` | 3:23 | 44.1 kHz | `sm1011_m101100001.mp3` (the bank's name has a trailing space) |
| m10_11 | (no retail name) | `sm1011` | `m101100002` | 1:38 | 48 kHz | `sm1011_m101100002.mp3` |
| m10_14 | Brightstone Cove Tseldora / Lord's Private Chamber | `sm1014` | `m101400001` | 3:20 | 48 kHz | `sm1014_m101400001.mp3` |
| m10_15 | Aldia's Keep | `sm1015` | `m101500001` | 2:35 | 44.1 kHz | `sm1015_m101500001.mp3` |
| m10_16 | The Lost Bastille / Sinners' Rise / Belfry Luna | `sm1016` | `m101600001` | 3:22 | 44.1 kHz | `sm1016_m101600001.mp3` |
| m10_16 | The Lost Bastille / Sinners' Rise / Belfry Luna | `sm1016` | `m101600002` | 2:56 | 44.1 kHz | `sm1016_m101600002.mp3` |
| m10_17 | Harvest Valley / Earthen Peak | `sm1017` | `m101700001` | 3:25 | 48 kHz | `sm1017_m101700001.mp3` |
| m10_17 | Harvest Valley / Earthen Peak | `sm1017` | `m101700002` | 3:37 | 48 kHz | `sm1017_m101700002.mp3` |
| m10_18 | No-man's Wharf | `sm1018` | `m101800001` | 3:07 | 44.1 kHz | `sm1018_m101800001.mp3` |
| m10_19 | Iron Keep / Belfry Sol | `sm1019` | `m101900001` | 2:30 | 44.1 kHz | `sm1019_m101900001.mp3` |
| m10_19 | Iron Keep / Belfry Sol | `sm1019` | `m101900002` | 2:56 | 44.1 kHz | `sm1019_m101900002.mp3` (same audio as `sm1016_m101600002`) |
| m10_21 | (no retail name) | `sm1021` | `m102100001` | 2:24 | 44.1 kHz | `sm1021_m102100001.mp3` (same audio as `sm1010_m101000002`) |
| m10_22 | (no retail name) | `sm1022` | `m102200001` | 3:04 | 44.1 kHz | `sm1022_m102200001.mp3` |
| m10_23 | Huntsman's Copse / Undead Purgatory | `sm1023` | `m102300001` | 2:14 | 44.1 kHz | `sm1023_m102300001.mp3` |
| m10_23 | Huntsman's Copse / Undead Purgatory | `sm1023` | `m102300002` | 2:23 | 48 kHz | `sm1023_m102300002.mp3` |
| m10_25 | The Gutter / Black Gulch | `sm1025` | `m102500001` | 2:51 | 48 kHz | `sm1025_m102500001.mp3` |
| m10_26 | (no retail name) | `sm1026` | `m102600001` | 2:27 | 48 kHz | `sm1026_m102600001.mp3` |
| m10_27 | Dragon Aerie / Dragon Shrine | `sm1027` | `m102700001` | 2:58 | 44.1 kHz | `sm1027_m102700001.mp3` |
| m10_28 | Firelink Throne | `sm1028` | `m102800001` | 3:32 | 48 kHz | `sm1028_m102800001.mp3` |
| m10_28 | Firelink Throne | `sm1028` | `m102800002` | 2:26 | 44.1 kHz | `sm1028_m102800002.mp3` |
| m10_29 | Majula | `sm1029` | `m102100001` | 2:24 | 44.1 kHz | `sm1029_m102100001.mp3` (same audio as `sm1010_m101000002`) |
| m10_31 | Heide's Tower of Flame / Cathedral of Blue | `sm1031` | `m103100001` | 2:58 | 44.1 kHz | `sm1031_m103100001.mp3` |
| m10_31 | Heide's Tower of Flame / Cathedral of Blue | `sm1031` | `m103100002` | 4:01 | 44.1 kHz | `sm1031_m103100002.mp3` |
| m10_32 | Shaded Woods / Shrine of Winter | `sm1032` | `m103200001` | 3:02 | 44.1 kHz | `sm1032_m103200001.mp3` |
| m10_33 | Doors of Pharros | `sm1033` | `m103300001` | 3:32 | 44.1 kHz | `sm1033_m103300001.mp3` |
| m20_03 | (no retail name) | `sm2003` | `m100300001` | 2:24 | 48 kHz | `sm2003_m100300001.mp3` (same audio as `sm1003_m100300001`) |
| m20_03 | (no retail name) | `sm2003` | `m100300011` | 3:27 | 48 kHz | `sm2003_m100300011.mp3` (same audio as `sm1003_m100300011`) |
| m20_10 | Memory of Vammar / Memory of Orro / Memory of Jeigh | `sm2010` | `m201000001` | 2:55 | 44.1 kHz | `sm2010_m201000001.mp3` (same audio as `sm1010_m101000001`) |
| m20_11 | Shrine of Amana | `sm2011` | `m201100001` | 3:22 | 44.1 kHz PCM | `sm2011_m201100001.wav` |
| m20_11 | Shrine of Amana | `sm2011` | `m201100002` | 3:14 | 35.109 kHz PCM | `sm2011_m201100002.wav` |
| m20_15 | (no retail name) | `sm2015` | `m101500001` | 3:23 | 44.1 kHz | `sm2015_m101500001.mp3` |
| m20_21 | Drangleic Castle / King's Passage / Throne of Want | `sm2021` | `m202100001` | 3:04 | 44.1 kHz | `sm2021_m202100001.mp3` (same audio as `sm1022_m102200001`) |
| m20_24 | Undead Crypt | `sm2024` | `m202400001` | 2:26 | 44.1 kHz | `sm2024_m202400001.mp3` (same audio as `sm1028_m102800002`) |
| m20_26 | Dragon Memories | `sm2026` | `m202600001` | 4:20 | 48 kHz | `sm2026_m202600001.mp3` |
| m20_27 | (no retail name) | `sm2027` | `m102700001` | 3:49 | 48 kHz | `sm2027_m102700001.mp3` |
| m20_30 | (no retail name) | `sm2030` | `m203000001` | 3:24 | 48 kHz | `sm2030_m203000001.mp3` |
| m20_63 | (no retail name) | `sm2063` | `m206300001` | 2:44 | 44.1 kHz | `sm2063_m206300001.mp3` |
| m40_03 | Dark Chasm of Old | `sm4003` | `m400300001` | 3:15 | 44.1 kHz | `sm4003_m400300001.mp3` |

Notes:

- **Shared tracks** (byte-identical, under a second bank or event): Iron Keep's second track = Lost Bastille's second;
  m10_21's and m10_29's (Majula) track = Forest's second; Memory of Vammar's = Forest's first; m20_03's two = m10_03's
  two; Drangleic Castle's = m10_22's; **Undead Crypt's = Firelink Throne's second**. `sm2027` carries an event named
  `m102700001` that is **not** the same audio as `sm1027`'s event of that name; `sm2015` likewise reuses `m101500001`
  with different audio. (October listed only three pairs; its mis-cut files hid the rest.)
- **2026-11-03 E3 note, now resolved:** m20_62 / m20_63 are the E3 2013 demo's two maps. `sm2063`'s `m206300001` (2:44)
  is byte-identical to retail Drangleic Castle's `m202100001`; the `sm1010` tracks are 2:55 / 2:24
  ([E3 2013 demo leftovers](e3-demo-leftovers.md)).
- Maps with two or more tracks (for example `sm1003`: `m100300001` and `m100300011`) probably have a normal / boss or
  a quiet / combat pair, but nothing in the banks says which is which.
- `sm2011` (retail: Shrine of Amana) is the only music stored uncompressed. The second track's 35,109 Hz is the bank's
  own rate for it (a rate chunk in its header), so that is the rate the files use; whether it sounds right is unchecked.

## Network Test music in retail (PS3 v1.00, 2026-11-18)

19 of the 46 NT map tracks are **byte-identical** to a sample in the retail PS3 disc's `SOUND\FRPG2_SM*.FSB` banks. Most
keep their bank and event (sm1002, sm1004 ×2, sm1010 #1, sm1015, sm1016 #1, sm1018, sm1019 #1, sm1023 #1, sm1031 ×2,
sm1032, sm1033, sm4003). The moves:

| NT | retail |
|---|---|
| `sm1001` `m100100001` (m10_01) | Undead Crypt `sm2024` `m202400002` |
| `sm1023` `m102300002` | `sm1034` `m103400001` |
| `sm1027` `m102700001` (Dragon Aerie) | Drangleic Castle `sm2021` **`m102800002`** |
| `sm2063` `m206300001` (E3 Rock-peak Castle) | Drangleic Castle `sm2021` `m202100001` |
| `sm2010` `m201000001` | retail `sm1010` / `sm2010` (both) |

**Firelink Throne's event numbers live on in retail** (traced in `firelink-throne.md`: they are the Throne of Want
boss music, `m102800002` Throne Watcher & Defender, `m102800001` Nashandra): Drangleic Castle's bank `sm2021` holds
`m102800001` (4:34) and `m102800002` (2:58). Retail's `m102800002` is the NT's Dragon Aerie track. The NT's own [Firelink Throne](../unused/unused-models.md) tracks are not in
retail byte for byte. By loudness-envelope match (`tools\fsb_music.py compare`; same piece ≈ 0.8-1.0, different < 0.5):
NT `m102800001` vs retail `m102800001` = 0.46 (inconclusive); NT `m102800002` vs retail `m760203000` (an 18 s cue in
the same castle bank) = **1.00**, so the cue looks like an excerpt of it. 76020300 is a sung "Ahh...Ah..." line in
retail's m20_21 talk file. Which retail scene plays these events is not decoded.

## System sounds (`frpg2_smain`)

| event | length | rate | file | note |
|---|---|---|---|---|
| `bossout` | 0:13 | 48 kHz | `smain_bossout.mp3` | jingle (name suggests the end of a boss fight) |
| `gamestart` | 0:02 | 44.1 kHz | `smain_gamestart.mp3` | short jingle |
| `m000000001` | 2:23 | 44.1 kHz | `smain_m000000001.mp3` | unlabelled |
| `m000000002` | 3:49 | 48 kHz | `smain_m000000002.mp3` | unlabelled; retail has an `m000000002` of the same length |
| `m000000003` | 6:58 | 44.1 kHz | `smain_m000000003.mp3` | unlabelled |
| `m000000010` | 0:10 | 48 kHz | `smain_m000000010.mp3` | short sting |
| `m000000011` | 0:04 | 44.1 kHz | `smain_m000000011.mp3` | short sting |
| `m000000014` | 0:22 | 48 kHz | `smain_m000000014.mp3` | sting |
| `s000000238` | 0:03 | 44.1 kHz mono | `smain_s000000238.mp3` | effect (new in the redo) |
| `s000000239` | 0:04 | 44.1 kHz mono | `smain_s000000239.mp3` | effect (new in the redo) |
| `s000000310` | 0:06 | 48 kHz | `smain_s000000310.mp3` | effect (new in the redo) |
| `s999999999` | 0:13 | 48 kHz | `smain_s999999999.mp3` | effect / placeholder id (new in the redo) |

The three long ones (`m000000001` to `m000000003`) are the likely title screen / lobby / menu themes; I haven't matched
them to a screen. The event file `frpg2_stationary.fev` would show which event the game plays where, but I haven't read it that far.

## The `y` streams (`frpg2_xm<map>.fsb`)

One stream per map, 1:37 (1:28 in `xm1011` / `xm2011`, 1:36 in `xm2063`). All 40 banks are converted now (October
skipped the two PCM ones). Grouped by byte-identical sample data, **17 different streams** in all:
- 11: xm1002, xm1017, xm1018, xm1019, xm1021, xm1029, xm1031, xm1032, xm1033, xm1034, xm2024
- 7: xm1003, xm1013, xm2003, xm2013, xm2026, xm2027, xm2028
- 4: xm1014, xm1025, xm1027, xm1028 (retail PS3 v1.00's leftover `XM1028.FSB` is this stream too)
- 3: xm1010, xm1030, xm2062
- 2: xm1015, xm2021 (48 kHz)
- 2: xm1022, xm1026 (48 kHz, three audible streams A / B / C)
- 1 each: xm1001 (PCM 48 kHz, three pairs A / B / C), xm1004, xm1011, xm1016, xm1023, xm2010, xm2011, xm2015 (48 kHz),
  xm2034 (PCM 27,403 Hz, pairs A / B), xm2063, xm4003

## Limits

- No track has a title anywhere in the demo; matching these to the retail soundtrack needs listening.
- I did not listen to any of it. Every file was decoded start to finish and its loudness measured (none silent).
- Which event the game plays where (normal vs boss vs lobby) is in the `.fev` event files and the map data; not decoded here.
- Only the sound folder of the Western test was used. The Japanese test's sound folder was not compared.

## Reproduce

From `<local path> Vanilla`:

```
uv run --no-project --python 3.12 python -I -X utf8 tools\fsb_ysplit.py batch "<local path> Network Test\sound" "<local path> Network Test\music\music" "frpg2_sm[0-9]*.fsb" --names m --tsv "<local path> Network Test\music\music.tsv"
uv run --no-project --python 3.12 python -I -X utf8 tools\fsb_ysplit.py batch "<local path> Network Test\sound" "<local path> Network Test\music\music" "frpg2_smain.fsb" --tsv "<local path> Network Test\music\smain.tsv"
uv run --no-project --python 3.12 python -I -X utf8 tools\fsb_ysplit.py batch "<local path> Network Test\sound" "<local path> Network Test\music\ambience_y" "frpg2_xm*.fsb" --tsv "<local path> Network Test\music\ambience_y.tsv"
```
