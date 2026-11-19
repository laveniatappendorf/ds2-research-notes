# The name "Amana" (Shrine of Amana, `m20_11`)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

Question (the author, 2026-10-27): is there any reference to Amana, in Japanese or English, that suggests where the name came
from or ties it to other characters?

**Answer: the game never explains the name.** Every build from v1.00 to PC Scholar uses "Amana" in the same three
player-facing texts, and the developers' files use the same name in five internal labels. The Network Test has no
"Amana" at all, although it already has the Shrine's dialogue. In Japanese the place is アマナの祭壇, "the Altar of
Amana"; "Shrine" comes from the English localisation. The links to other characters come through the place, not the
name:
- the priestesses (the singing enemies);
- the Demon of Song;
- the Milfanito, whose Japanese name contains the Undead Crypt's Fenito (ファニト);
- Rhoy the Explorer, who died there;
- in the NT only, Gavlan.

## Every use of the name

### Player-facing text

Searched every FMG (loose and inside binders) in the NT, the JP NT, v1.00, the JP disc, PS3 Scholar and PC Scholar. All
11 PS3 languages carry the same three entries, unchanged from v1.00 to PC (PC has no Japanese):

| Id | Japanese | English |
|---|---|---|
| MapName 20110000 | アマナの祭壇 "the altar of Amana" | Shrine of Amana |
| Item 3810000, name | アマナの杖 | [Staff of Amana](../mechanics/staff-of-amana-hollowing.md) |
| Item 3810000, description | アマナの巫女が用いていた杖 … 祭壇には魔物が封じられていたが 儀式はやがて失われた … 人を誘う唄を覚えた: "the staff the shrine maidens of Amana used; a demon was sealed at the altar, but the rite was lost in time; freed, the demon grew cunning and learned a song that lures people" | "Staff held by a priestess of Amana … learned to lure humans into its lair using song" |
| Item 64190000 (Demon of Song soul) | アマナの祭壇で唄うデーモンのソウル … それを祀り、鎮める者は絶えてしまった: "soul of the demon that sings at the Altar of Amana … those who enshrined it and kept it calm have died out" | "Soul of the Demon of Song at the Shrine of Amana … the line of priestesses who looked after the shrine and appeased the creature have died off" |

No dialogue, no other item and no NPC says Amana. The Milfanito, who live there, never name it.

Three details are only in the Japanese:
- **祭壇 is an altar, not a shrine building.** The developers' signs point at one in the map: sign 023 says "the altar
  itself is here" and sign 026 says "light the altar" (`developer-signs.md`).
- **祀る means to enshrine or worship.** In Japanese, the demon was the object of the priestesses' rites; the English
  "looked after the shrine" softens that.
- **アマナの only means "of Amana".** It doesn't say whether Amana is a place, an order or a deity.

### Internal names

These are UTF-16 strings, found by a byte search of every file and binder member in v1.00 and Scholar; the NT has none:

| File | String | Meaning |
|---|---|---|
| `map\m20_21_00_00\m20_21_00_00.msb` (Drangleic Castle) | シームレスワープ【アマナの祭壇】 | seamless warp [Altar of Amana] |
| `ezstate\event_m20_21_00_00.edd` | 昇降機⇔アマナの祭壇 | lift ⇔ Altar of Amana |
| `ezstate\ai205002.esd` | ai205002_毒角虫（アマナ） | Poison Horn Beetle (Amana): the Shrine's beetle variant; retail only (the NT has 205000 / 205010) |
| `PrefabEditor\o00_0240.pfbbin` | アニメ114：14.7m落下（アマナの祭壇） | animation 114: 14.7 m drop (Altar of Amana) |
| developer sign, `m20_21` model `m8000`, #000 | アマナの祭壇への接続 | connection to the Altar of Amana |

So the working name was already the shipped one (but see [the map list's 妖精のほとり, "the fairy waterside"](e3-demo-leftovers.md#9-past-and-present-the-other-time-labelled-areas-added-2026-11-03)). The bosses have earlier labels ([boss working names](boss-region-names.md)); Amana has
none. The game program (EBOOT of the NT, v1.00 and Scholar) doesn't contain the name.

## Network Test (Aug 2013)

- **No Amana anywhere.** Neither NT build has "Amana" or アマナ in any text:
  - English or Japanese;
  - the language-less draft talk folder;
  - any other file (byte search of all 3,444 files: ASCII, UTF-16, Shift-JIS).

  The NT's map-name table has only two entries, so this doesn't prove the name came later.
- **The Shrine's people were already there:**
  - `talk\m20_11_00_00.fmg` (Japanese, 92 lines) has the Milfanito's lines, almost all as retail.
  - [The AI scripts already exist](nt-ai-labels-retail-enemies.md) for:
    - the shrine maidens: `ai105000` 妖精の巫女（亡者）, plus an NT-only `ai105020` variant, 篝火罠 "bonfire trap";
    - the Milfanito: `ai760000` / `ai760100` 吟遊詩人の少女１／２ "minstrel girl 1 / 2", and `ai760200` 虜囚の少女
      "captive girl";
    - the Demon of Song: `ai602000` 美声カエル "sweet-voiced frog".
  - BonfireName has the Shrine's five slots, 11650-11670, still unnamed "Bonfire". Retail names four of them: Tower of
    Prayer, Crumbled Ruins, Rhoy's Resting Place and Rise of the Dead (死者の塔).
- **Gavlan was meant to visit the Shrine.** The NT's m20_11 file also carries NPC 7260's whole dialogue (30 lines,
  72600000-72609600). It is identical to his copies in m10_17, m10_18, m10_25 and m10_33; the NT spells him ガバランス
  and gives him the laugh シッシッシッ, where retail has ガヴァラン and グハハハッ. Retail kept the other four files' lines
  and dropped the Shrine's. A developer sign in the Shrine reads 廃屋（NPCは祭壇へ移動）, "ruined house (NPC moved to the
  altar)", without naming the NPC.

## Characters tied to the place

### The priestesses of Amana are the singing enemies

[Enemy 105010 (model c1050)](../builds/shrine-of-amana-pc.md#2-the-spellcasters-amana-shrine-maidens) runs the 妖精の巫女（亡者） AI, "fairy shrine maiden (Hollow)". The Staff of Amana is
the first entry in its drop row 10501000, at 2 % as the param reads.

The developers' signs describe what the maidens do:
- kill the shrine maiden and the singing stops, so the enemies in range attack as normal (sign 038);
- where there's no shrine maiden, the submerged enemies just wander (sign 022).

The "fairy" in the name is probably the glowing bugs over the water. The signs call them 光虫, and the Milfanito call
them "the little ones", who dance when they sing.

### The Demon of Song

The demon was sealed at the altar and kept by the priestesses' line, which died out (the two Amana texts). Its
working names are 美声カエル "sweet-voiced frog" (its AI and its boss-music region) and, on its NT soul, 大蛙のデーモン
"giant frog demon".

### The Milfanito and the Fenito

The Milfanito (ミルファニト, NPCs 7600-7602):
- sing for "those who bear death and Dark";
- were "taught song by the Great Dead One" (大いなる死者);
- say the little ones were born from the Great Dead One.

In Japanese the Undead Crypt's Fenito is ファニト, Fanito: "I am a Fanito, one who weaves death and guards it" (talk
50600300). So ミル + ファニト shows the tie plainly; "Milfanito" vs "Fenito" in English hides it a little.

The developers' signs make the Shrine the Crypt's front door:
- sign 007: "this building is the entrance to the Undead Crypt … an ancient tomb built before the royal castle";
- sign 023: "the altar itself is here; whatever can't go into the mausoleum is left here".

The Singer's Dress (23040101) also names the Great Dead One: "singers given song by the Great Dead One … little concept
of self". It is found in the Shrine (item lot 20116200). DS2's text never says who the Great Dead One is.

### Rhoy the Explorer

The Shrine's bonfire Rhoy's Resting Place is 探索者ロイの最期の地 in Japanese, "the place where Rhoy the Explorer met
his end". His belongings:

| Item | Where |
|---|---|
| Ring of Whispers | Sweet Shalquoir's shop |
| Bracing Knuckle Ring | Lost Bastille, lot 10166200 |
| Ash Knuckle Ring (its Japanese says 探究者 "researcher", not 探索者) | Grave of Saints, lot 10345000 |
| Rhoy's Stone of Knowledge | no item lot or shop |

The NT already had him: ロイの魔石, "Rhoy's magic stone", which "detects chest traps with high probability". That's
the same effect as retail's Stone of Knowledge.

The Milfanito's own death-drop row (76000000) lists Rhoy's Ash Knuckle Ring with amount 0 and chance 0, the same in
v1.00, PS3 Scholar and PC. It's a disabled entry; one guess is that the singers were once meant to hold his ring.

## Outside the game files (searched 2026-10-27)

No published source explains the name. Searched in English and Japanese: interviews, the Design Works book, the official
guides, Japanese lore wikis and Q&A sites.

- **Design Works interview** (Udon, 2015; English translation by Peter Barnard, barnardtranslations.com). Speakers:
  Tanimura, Satake, Katayama, Tomari, Tonaki and Ou.
  - **Concept.** Tomari says the Shrine is almost exactly its first concept. Glowing insects in the trees were cut. The
    area was meant to be quiet and lovely, with enemies coming out to attack once you disturb the silence; that fits
    developer sign 038. Objects were kept to a minimum to keep it serene.
  - **Mood.** Tonaki loves the song drifting over the silence.
  - **Demon of Song.** Based on an African frog with loose skin, with a skull-like face coming out of its mouth; Ou
    picks it as a favourite.
  - **No word on the name.** The only naming story is The Gutter's: its working name was "D-suke village", after
    Satake's nickname, and Tanimura gave Satake a list of names to pick from. So at least one area name came from the
    director's list.
  - **Not checked:** the Japanese official guide (ダークソウルII ザ・コンプリートガイド) and the book's captions beyond
    this interview.
- **Yahoo! Chiebukuro** "ダークソウル2の「アマナの祭壇」について" (2015; read through the Wayback Machine, since Yahoo!
  Japan is blocked in the UK): it asks why players hate the area. Nothing on the name.
- **Japanese lore wiki** (seesaawiki mousou_darksouls2, page アマナの祭壇): describes the area and its people, nothing
  on the name.
  - It points out that the DLC 1 Shulva shares the bonfire name Tower of Prayer, and has a "shrine of the dead" (死者の
    祠).
  - A 2015 comment guesses Amana was built, like Shulva, to seal something powerful.
  - Checked in Scholar: Shulva's bonfire 35685 is 祈りの塔 / Tower of Prayer, the same name as Amana's 11650. The DLC's
    Sanctum Priestess Tiara says those priestesses "sang to preserve the dragon's deep slumber": the same pattern of
    singing priestesses keeping a monster sealed, written for the 2014 DLC. No DLC text names Amana.
- **The Milfanito-Fenito-Nito reading** is a known fan theory in English (Steam forums, the fan wikis), drawn from the
  "-nito" endings and the Great Dead One. The Japanese ミルファニト / ファニト makes the first half of it plain.
  Nothing official names the Great Dead One.

## Where the name might come from (outside the game files)

Nothing in the data or the sources above supports any of these; they are the obvious candidates:
- Hebrew אמנה *amanah*, "covenant, faithfulness" (the same root as *amen*).
- The Biblical Amana: a mountain (Song of Songs 4:8). It is also a variant reading of Abana, a river of Damascus
  (2 Kings 5:12): the waters Naaman thought better for washing off his leprosy.
- Arabic *amāna*, "a trust, something entrusted". The Milfanito say their song was 預けられ, "entrusted", to them.
- Japanese アマナ (甘菜), *Amana edulis*, a small white spring flower of the lily family.

## Ruled out

- **NT hex 34142000 仮・大いなる死者の活性:** "Great Dead Again", the stronger form of NT 34112000 死者の活性 (retail
  Dead Again). It follows the NT's 怨念 / 大怨念 pattern, so it is probably not about the Great Dead One.
- **ASCII "amana" byte matches:** these are all in object models' index buffers (o00_9040, o03_9000, o17_9120 …), so
  they are noise.
- **The leminerva site copy:** it has only the map name.

## Correction made on the way

[Network Test bosses not in retail](nt-bosses-not-in-retail.md#5-dialogue-nt-only-talk-text-all-japanese-unless-noted) section 5 had two rows wrong because it compared lines by id:
- The Milfanito's thanks for calming the cursed singing is in retail, renumbered to 76007000 / 76017000.
- Gavlan's カエル？ オマエ is "You? Go home?" (retail English), not "Frog? You".

Both rows are now corrected there.

Tools (scratch, not kept): FMG regex search over every build, a byte search of every archive file and binder member,
AI-label and generator listings, `paramdef.py` item-lot rows.
