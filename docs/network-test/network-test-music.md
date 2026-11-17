# Network Test music: what is in the sound files (2026-10-23)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question: what are the names of the BGM tracks in the network test?

**There are no track names.** No text file in the Western or Japanese build titles a piece of music (the only hits for
"music" are the volume option labels), and the sound banks name tracks by number only. What can be given is a map-by-map
list of the tracks, with their lengths, and the tracks themselves as playable files:

**`<local path> Network Test\music\`**: `music\` (54 MP3/WAV tracks) and `ambience_y\` (the `y` streams, split into stereo pairs).
File names are `<bank>_<event>.mp3`, so `sm1002_m100200001.mp3` is event `m100200001` in bank `frpg2_sm1002`.

## How the files are laid out

- Sound lives in FMOD banks in `sound\`. Each `.fev` event file names events; each `.fsb` (FSB5 format) holds the audio.
- **`frpg2_sm<map>.fsb`** holds the map's streamed tracks. Events named **`m<map>0000n`** are the music; `a…` ambience loops,
  `v…` voices and `o…`/`s…`/`c…` effects share the banks. Calling the `m…` events the music is my reading of the naming
  and the track lengths (1 to 6 minutes); I have only checked the audio's length, not listened to it.
- **`frpg2_xm<map>.fsb`** holds one long `y<map>00000` stream per map. These are not songs: they hold three stereo
  MP3 streams interleaved frame by frame, the third of them silent, and many maps share byte-identical copies. I have split
  them into `_A` / `_B`. What they are (ambient layers is a guess) is unknown.
- **`frpg2_smain.fsb`** holds the system tracks: title / lobby music and jingles.
- Almost all audio is MP3 at 44.1 or 48 kHz, 160 kbps. Three banks are uncompressed PCM (`sm2011`, `xm1001`, `xm2034`).
- The FSB header's sample count is reliable for the short ambience loops but is **too small for the long music streams**
  (for `m100200001` it says 81 s, the MP3 frames add up to 229 s). The lengths below are counted from the MP3 frames.
- The sample offsets in the banks are a few hundred bytes off the real frame boundaries, because the samples run on
  contiguously. Each track here starts at its first whole frame.

## Music by map

Map names are **retail's** `MapName` text (the demo names only Huntsman's Copse and Undead Purgatory). A map named here
is the map id's retail name; the demo's own version of that map may differ or be unfinished.
Banks `sm20xx` belong to maps `m20_xx`.

| map id | retail name | bank | track (event) | length | file |
|---|---|---|---|---|---|
| m10_01 | (no retail name) | `sm1001` | `m100100001` | 3:07 | `sm1001_m100100001.mp3` |
| m10_02 | Things Betwixt | `sm1002` | `m100200001` | 3:49 | `sm1002_m100200001.mp3` |
| m10_03 | (no retail name) | `sm1003` | `m100300001` | 1:11 | `sm1003_m100300001.mp3` |
| m10_03 | (no retail name) | `sm1003` | `m100300011` | 1:43 | `sm1003_m100300011.mp3` |
| m10_04 | Majula | `sm1004` | `m100400001` | 1:39 | `sm1004_m100400001.mp3` |
| m10_04 | Majula | `sm1004` | `m100400002` | 1:11 | `sm1004_m100400002.mp3` |
| m10_10 | Forest of Fallen Giants | `sm1010` | `m101000001` | 1:27 | `sm1010_m101000001.mp3` |
| m10_10 | Forest of Fallen Giants | `sm1010` | `m101000002` | 1:11 | `sm1010_m101000002.mp3` |
| m10_11 | (no retail name) | `sm1011` | `m101100001` | 1:41 | `sm1011_m101100001.mp3` |
| m10_11 | (no retail name) | `sm1011` | `m101100002` | 4:06 | `sm1011_m101100002.mp3` |
| m10_14 | Brightstone Cove Tseldora / Lord's Private Chamber | `sm1014` | `m101400001` | 1:39 | `sm1014_m101400001.mp3` |
| m10_15 | Aldia's Keep | `sm1015` | `m101500001` | 1:17 | `sm1015_m101500001.mp3` |
| m10_16 | The Lost Bastille / Sinners' Rise / Belfry Luna | `sm1016` | `m101600001` | 1:40 | `sm1016_m101600001.mp3` |
| m10_16 | The Lost Bastille / Sinners' Rise / Belfry Luna | `sm1016` | `m101600002` | 1:27 | `sm1016_m101600002.mp3` |
| m10_17 | Harvest Valley / Earthen Peak | `sm1017` | `m101700001` | 1:42 | `sm1017_m101700001.mp3` |
| m10_17 | Harvest Valley / Earthen Peak | `sm1017` | `m101700002` | 6:20 | `sm1017_m101700002.mp3` |
| m10_18 | No-man's Wharf | `sm1018` | `m101800001` | 1:33 | `sm1018_m101800001.mp3` |
| m10_19 | Iron Keep / Belfry Sol | `sm1019` | `m101900001` | 1:14 | `sm1019_m101900001.mp3` |
| m10_19 | Iron Keep / Belfry Sol | `sm1019` | `m101900002` | 4:34 | `sm1019_m101900002.mp3` |
| m10_21 | (no retail name) | `sm1021` | `m102100001` | 2:23 | `sm1021_m102100001.mp3` |
| m10_22 | (no retail name) | `sm1022` | `m102200001` | 3:03 | `sm1022_m102200001.mp3` |
| m10_23 | Huntsman's Copse / Undead Purgatory | `sm1023` | `m102300001` | 1:07 | `sm1023_m102300001.mp3` |
| m10_23 | Huntsman's Copse / Undead Purgatory | `sm1023` | `m102300002` | 1:11 | `sm1023_m102300002.mp3` |
| m10_25 | The Gutter / Black Gulch | `sm1025` | `m102500001` | 2:51 | `sm1025_m102500001.mp3` |
| m10_26 | (no retail name) | `sm1026` | `m102600001` | 2:26 | `sm1026_m102600001.mp3` |
| m10_27 | Dragon Aerie / Dragon Shrine | `sm1027` | `m102700001` | 3:38 | `sm1027_m102700001.mp3` |
| m10_28 | Firelink Throne | `sm1028` | `m102800001` | 1:45 | `sm1028_m102800001.mp3` |
| m10_28 | Firelink Throne | `sm1028` | `m102800002` | 4:12 | `sm1028_m102800002.mp3` |
| m10_29 | Majula | `sm1029` | `m102100001` | 2:23 | `sm1029_m102100001.mp3` (same audio as `sm1021_m102100001`) |
| m10_31 | Heide's Tower of Flame / Cathedral of Blue | `sm1031` | `m103100001` | 1:29 | `sm1031_m103100001.mp3` |
| m10_31 | Heide's Tower of Flame / Cathedral of Blue | `sm1031` | `m103100002` | 6:28 | `sm1031_m103100002.mp3` |
| m10_32 | Shaded Woods / Shrine of Winter | `sm1032` | `m103200001` | 5:20 | `sm1032_m103200001.mp3` |
| m10_33 | Doors of Pharros | `sm1033` | `m103300001` | 3:32 | `sm1033_m103300001.mp3` |
| m20_03 | (no retail name) | `sm2003` | `m100300001` | 1:11 | `sm2003_m100300001.mp3` (same audio as `sm1003_m100300001`) |
| m20_03 | (no retail name) | `sm2003` | `m100300011` | 1:43 | `sm2003_m100300011.mp3` (same audio as `sm1003_m100300011`) |
| m20_10 | Memory of Vammar / Memory of Orro / Memory of Jeigh | `sm2010` | `m201000001` | 1:27 | `sm2010_m201000001.mp3` |
| m20_11 | Shrine of Amana | `sm2011` | `m201100001` | 3:21 | `sm2011_m201100001.wav` PCM, 44100 Hz |
| m20_11 | Shrine of Amana | `sm2011` | `m201100002` | 3:14 | `sm2011_m201100002.wav` PCM, 35109 Hz |
| m20_15 | (no retail name) | `sm2015` | `m101500001` | 1:41 | `sm2015_m101500001.mp3` |
| m20_21 | Drangleic Castle / King's Passage / Throne of Want | `sm2021` | `m202100001` | 3:03 | `sm2021_m202100001.mp3` (same audio as `sm1022_m102200001`) |
| m20_24 | Undead Crypt | `sm2024` | `m202400001` | 2:26 | `sm2024_m202400001.mp3` |
| m20_26 | Dragon Memories | `sm2026` | `m202600001` | 4:19 | `sm2026_m202600001.mp3` |
| m20_27 | (no retail name) | `sm2027` | `m102700001` | 3:49 | `sm2027_m102700001.mp3` |
| m20_30 | (no retail name) | `sm2030` | `m203000001` | 3:24 | `sm2030_m203000001.mp3` |
| m20_63 | (no retail name) | `sm2063` | `m206300001` | 4:22 | `sm2063_m206300001.mp3` |
| m40_03 | Dark Chasm of Old | `sm4003` | `m400300001` | 1:37 | `sm4003_m400300001.mp3` |

Notes:

- **Added 2026-11-03:** m20_62 / m20_63 are the E3 2013 demo's two maps ("E3 Fort of Resistance" / "E3 Rock-peak Castle"
  in the 2014 PC developer rows). `sm2063`'s `m206300001` is byte-identical to retail Drangleic Castle's `m202100001`,
  and decoding gives it 2:44, not the 4:22 above; the two `sm1010` tracks decode to 2:55 / 2:24. See
  `e3-demo-leftovers.md` (durations not resolved).
- Maps with two or more tracks (for example `sm1003`: `m100300001` and `m100300011`) probably have a normal / boss or
  a quiet / combat pair, but nothing in the banks says which is which.
- `sm1029` repeats `sm1021`'s track, `sm2003` repeats `sm1003`'s two tracks and `sm2021` repeats `sm1022`'s: the same
  audio shipped under a second bank. `sm2027` carries an event named `m102700001` that is **not** the same audio as
  `sm1027`'s event of that name; `sm2015` likewise reuses `m101500001` with different audio.
- `sm2011` (retail: Shrine of Amana) is the only music stored uncompressed: two 16-bit stereo tracks of 3:22 and 3:14
  (35 MB and 27 MB). The second reports 35,109 Hz in its header, which is unusual; I played it back at that rate, so its
  speed is unconfirmed. I checked only that the waveform is smooth, not that it sounds right.

## System tracks (`frpg2_smain`)

| event | length | file | note |
|---|---|---|---|
| `bossout` | 0:06 | `smain_bossout.mp3` | short jingle (name suggests the end of a boss fight) |
| `gamestart` | 0:01 | `smain_gamestart.mp3` | very short jingle |
| `m000000001` | 1:11 | `smain_m000000001.mp3` | unlabelled |
| `m000000002` | 1:54 | `smain_m000000002.mp3` | unlabelled |
| `m000000003` | 3:29 | `smain_m000000003.mp3` | unlabelled |
| `m000000010` | 0:04 | `smain_m000000010.mp3` | short sting |
| `m000000011` | 0:01 | `smain_m000000011.mp3` | short sting |
| `m000000014` | 0:10 | `smain_m000000014.mp3` | short sting |

The three long ones (`m000000001` to `m000000003`) are the likely title screen / lobby / menu themes; I haven't matched
them to a screen. The event file `frpg2_stationary.fev` would show which event the game plays where, but I haven't read it that far.

## The `y` streams (`frpg2_xm<map>.fsb`)

One stream per map, about 1:36 to 1:37 each (1:28 in two banks), as two stereo files. Of the 40 `xm` banks,
**`xm1001` and `xm2034` are 12-channel PCM and were not converted**; the rest are MP3. Many are byte-identical:
- 11 bank(s): xm1002, xm1017, xm1018, xm1019, xm1021, xm1029, xm1031, xm1032, xm1033, xm1034, xm2024
- 7 bank(s): xm1003, xm1013, xm2003, xm2013, xm2026, xm2027, xm2028
- 4 bank(s): xm1014, xm1025, xm1027, xm1028
- 3 bank(s): xm1010, xm1030, xm2062
- 2 bank(s): xm1015, xm2021
- 2 bank(s): xm1022, xm1026
- 1 bank(s): xm1004
- 1 bank(s): xm1011
- 1 bank(s): xm1016
- 1 bank(s): xm1023
- 1 bank(s): xm2010
- 1 bank(s): xm2011
- 1 bank(s): xm2015
- 1 bank(s): xm2063
- 1 bank(s): xm4003

So there are only 15 different `y` streams in all among the 38 converted banks, shared out over the maps.

## Limits

- No track has a title anywhere in the demo; matching these to the retail soundtrack needs listening.
- I did not play any of the audio. The MP3s were checked structurally (every file is an unbroken run of valid frames);
  the PCM files were checked for waveform smoothness only.
- Which event the game plays where (normal vs boss vs lobby) is in the `.fev` event files and the map data; not decoded here.
- Only the sound folder of the Western test was used. The Japanese test's sound folder was not compared.
