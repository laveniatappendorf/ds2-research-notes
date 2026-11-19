# Lore check: "Tark and Najka were duo bosses, code name Hell Scorpions" (2026-11-04)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author asked for a check on this community comment:

> In ds2 tark and najaka were duo bosses but if you travel in the past and helped tark he will be your ally instead
> and fight najaka alongside you much like the current game the code name for them was hell scorpions.

Builds checked: Network Test (NT, US `NPUB90979`) and JP NT (`NPJB90625`) text; NT, PS3 v1.00, PC pre-release
31 Mar 2014 (keeps the developers' row names) and PS3 Scholar params; v1.00 / Scholar talk FMGs; the community site's
dialogue dump (`Misc\Dark-Souls-Documents`). Translations are mine.

## Verdict

| Claim | Verdict | Evidence |
|---|---|---|
| Tark and Najka were a duo boss | **Not supported as a duo. There is a sign Tark was planned as a boss at some point** | see "Duo boss" |
| You travel to the past and help Tark | **No evidence tied to Tark.** But his forest did have a planned past stage (added 2026-11-04) | no Tark text in either NT; his "trapped / you saved me" voice lines are a generic template; the Shaded Woods' working name "living forest" has a past debug loadout ([two-soul bosses](two-soul-bosses.md#3-the-shaded-woods-had-a-past-stage) section 3) |
| He then fights Najka with you, like retail | **True of retail** (a summon) | EnemyParam 502009 = マンスコーピオン♂：白霊：虚ろの影の森, "male man-scorpion: white phantom: Shaded Woods" |
| Their code name was "Hell Scorpions" | **Wrong name.** "Hell Scorpion" is a real NT soul, but it is tied to nothing | the developers' name for both is マンスコーピオン ♂ / ♀, "man-scorpion" |

## Duo boss

For Tark once being a boss:
- **His sword is tagged as a boss-soul weapon.** PC 3/14 `ItemParam` 1520000 = 大蠍の黒針（DS：マンスコーピオン♂：刺突剣）,
  "great scorpion's black needle (DS: male man-scorpion: thrusting sword)" = the Black Scorpion Stinger. All 24 "DS："
  items are planned boss-soul weapons ([community cut-content sheet](community-cut-sheet-crossref.md)). In retail the sword is not a soul trade:
  you get it from Tark's drop (v1.00 lot 50200000, with the Second Dragon Ring) or a mimic (lot 1333000) (see [PC Scholar's change](../builds/placements-balance-pc.md#2-items)). Najka's soul
  trades for Soul Shower at Straid (ShopLineup 76801102).
- **He has boss rows no other non-boss has.** v1.00 `BossParam` 502000 / 502001 / 502009 (NPC, ghost, white phantom)
  and `BossPartsParam` 50200001-005 (upper body, scorpion body, left and right claw, tail). Every other character in
  these two params is a boss or a cut boss. The `BossParam` rows are only template values (Najka's 503000 is the one
  filled in). His summon form has one too, so the rows may just be there because he fights in the arena.

Against a duo:
- **The NT already treats him as an NPC.** NT `NpcEventParam` 50200000 is his row, next to Agdayne's (50600000) and
  the present-day King's (51400000). He has AI scripts (`ai502000` / `ai502010`, 下位 / 上位：マンスコーピオン（男）) and
  basic enemy rows, but no boss rows, boss parts, model or text. NT boss rows are Najka's only (503000, and 503010,
  which PC 3/14 names マンスコーピオン♀（デバッグ用）, "for debugging").
- **Every fight-level row is Najka alone**, in every build: `BossBattleParam` 1032000 m10_32 マンスコーピオン♀,
  `LockOnParam`, `DeadTypeParam` ボス：マンスコーピオン♀, the camera row, the boss region ([boss working names](boss-region-names.md)) and the
  soul. A two-boss fight gets its own row elsewhere: `BossBattleParam` m20_21 重騎馬兵：兵士のみ 2体戦, "two-body fight",
  the castle Dragonriders. The scorpions have none.

So the data allows "Tark was once planned as a boss, possibly with Najka". It doesn't show a duo fight, and by the NT
(Aug-Sep 2013) he was already an NPC.

## "Travel in the past"

- Neither NT has any Tark dialogue: no talk ids 50200000-50209999 in any NT FMG (the only rows in that range are
  placeholder "sack" items).
- Time travel in the NT belongs to other characters: King Vanclad's research into going back in time, and Bellderrick
  the time researcher ([Network Test bosses not in retail](nt-bosses-not-in-retail.md) section 5, `nt-draft-talk-folder.md`). Nothing ties it to Tark.
- **Added 2026-11-04: the Shaded Woods itself had a past stage planned.** Its working name was probably 生きている森
  "the living forest". The debug loadouts list that forest as first half / past / second half
  ([two-soul bosses](two-soul-bosses.md#3-the-shaded-woods-had-a-past-stage) section 3). No man-scorpion row names that past stage, so this makes a past visit to the forest
  plausible, not one involving Tark.
- **Tark's uncaptioned voice lines are a generic template, not his own story.** The community site lists eight audio
  lines with no subtitle, 50209000-50209600: "I am trapped", "Young human, I have a favor to ask", "And thank you,
  young human", "I remember you... you're the one who saved me". The same eight-line set ("open this / let me out /
  I'm trapped / thank you for saving me") exists for 12 NPCs: Benhart, Cale, Carhillion, Chloanne, Gilligan,
  Lenigrast, Licia, Maughlin, McDuff, Pate, Wellager and Tark. That is a cut "free a trapped NPC" mechanic, recorded
  for everyone. Retail talk FMGs have no text for these ids (v1.00 and Scholar m10_32 hold only 50209998 / 50209999).
  The transcriptions are the site's, from the audio; the sound banks were not checked here.
- What retail says about Tark's past is amnesia: "The past is a distant fog... My name was Tark... That is all that I
  remember" and the sword text. v1.00 had one more line, "Since I speak the tongue of men, does this mean I was once
  human?", dropped in 1.01 ([v1.00-only content](../builds/v1.00-only-content.md)).

## "Hell Scorpions"

- The developers' names, in every build that keeps them: マンスコーピオン♂ / （男） and マンスコーピオン♀, "man-scorpion"
  male and female. Najka's soul's working name is マンスコーピオン♀のソウル. In attack rows they are スコーピオン男 / 女.
- **ヘル・スコーピオン "Hell Scorpion" is NT soul 60736000.** It is one soul, not a pair name. Like all 27 NT souls it
  has the placeholder description, no item lot, and nothing ties it to a body ([Network Test bosses not in retail](nt-bosses-not-in-retail.md#2-the-27-nt-boss-souls-itemname-60730000-60755000-and-50140000) table 2).
  The next soul, 60737000, is デス・ストーカー "Death Stalker", which is also the name of a real scorpion. So the NT soul
  list does have two scorpion-themed bosses side by side. That is probably where the comment's name comes from.
  Whether those two were Najka and Tark, two forms of one boss (the same list gives Mytha two souls, larval and adult
  naga) or a scorpion boss that was cut, the data doesn't say.

## Retail story, for comparison

Tark and Najka were made by the same mad "master" (Najka's soul: "born of the misdeeds of an ancient being"). Najka,
his "betrothed" / "better half", went mad and turned on him. They have fought ever since, and "the wounds we exchange
are never lethal". He asks you to kill her and offers to help (summon sign, white phantom 502009). After the fight he
gives the Fragrant Branch of Yore (lot 1502000, developers' name ♀撃破-石化解除アイテム, "♀ defeated: petrification-
release item"). After Freja he calls the spider his master and gives the Second Dragon Ring (lot 1502010, 装備品譲渡,
"equipment handover"). So retail already has the "ally against Najka" half of the comment, and no partnership with her.

## Method

Scratch scripts in the session scratchpad (not kept): `fmg.py` over every talk / menu FMG of nt, jpnt, vanilla and
scholar for ids 50200000-50209999; `paramdef.load` over every regulation param of NT, v1.00 and PC 3/14 for 502xxx /
503xxx ids and their developer names, plus a name search for マンス / スコーピオン / 蠍 / ストーカー / ヘル; item lot and
shop rows that hold 1520000 or 64150000; the site's `eng2\dialogue\*.xml` for "No caption" lines.
