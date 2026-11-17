# When did Resistance / Agility become Adaptability / derived Agility? (2026-10-16)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author's question: the Network Test (NT) has Resistance and Agility as attributes, with Agility driving Evasion, Action
speed and Trap disabling; retail has Adaptability, with Agility derived from it. Was that changed after the NT build was
taken, or had the main game already diverged before it (an old fork)? Static evidence only; tools are the shared ones in
`..\..\DS2 Network Test\tools` with `DS2_GAME=nt|vanilla`.

**Answer (inference):** the change was made in the main game **after the NT's data was taken (17 Sep 2013) and before
vanilla's final build (14-27 Jan 2014)**, as an edit of the NT-era system, not a separate earlier design. The files
cannot say whether it fell before or after the public tests (planned 12 Oct 2013; the 27 Oct session failed for the
Americas; rerun 9-10 Nov 2013). It happened in at least two steps, with more trap work in between, which points to the
earlier part of that window rather than the last weeks.

## 1. Build dates: the binder stamps

Every BND4 / BHF4 header has an 8-byte stamp at +0x18: two-digit year, month letter, day, hour letter, minute, e.g.
`13J17Q9`. **Month letter B = January** (A is never used, M occurs and must be December), hour letter A = 0. Checks
against the disc file dates: vanilla `GameData.bhd` `14B27P30` = 27 Jan 2014 15:30 on a disc whose files are dated
29 Jan 2014; Scholar `14L26W21` = 26 Nov 2014 22:21, disc files 26 Nov 2014. (With A = January both would postdate
their own discs.)

| | archive packed | content binders |
|---|---|---|
| Network Test | `GameData.bhd` 17 Sep 2013 16:09, `regulation.bnd` 16:10 | 1,040 binders: mostly Jun-Aug 2013 (282 / 69 / 632), menus 9 Aug - 17 Sep 2013 |
| vanilla v1.00 | `GameData.bhd` 27 Jan 2014 15:30 | 4,611 binders rebuilt 14-24 Jan 2014; the rest untouched since 2010 / 2012 |
| Scholar (PS3) | `GameData.bhd` 26 Nov 2014 22:21 | not surveyed |

So the NT is not a stale branch: its content was being built through the summer of 2013 and its menus (status screen
included) into September. Vanilla re-stamped nearly everything in January 2014 (38 shared binders keep identical
stamps, all 2012 sound banks), so the stamps cannot date anything between September and January. The EBOOTs carry no
build date of their own (only Havok licence-key dates and a FIOS library date, 13 Mar 2013, identical in all three).

## 2. No trace of Adaptability in the NT

- NT text, English and Japanese: 0 hits for Adaptability / 適応.
- NT EBOOT: the stats-upload list names ten attributes (`seimeiryoku` .. `taikyuryoku` (Resistance), `binsyo`
  (Agility)) and the derived `avert_invincible_time`, `action_speed`, `trap_fail_probability`; no `tekioryoku`.

## 3. Vanilla is the NT system edited in place

- **Text IDs reused** (`iconHelp.fmg`): Adaptability took Resistance's ID 909012; the derived Agility took Action
  speed's ID 909038. Gaps are left exactly where NT stats were removed: 909013 (Agility attribute), 909037 (Evasion),
  909039 (Trap disabling), 909027 / 909035 (Break ATK / resist). New stats are appended (909050 Soul Memory,
  909051 Curse resist, 909052 Poise).
- **Same param types, extended:** `PhysicalStatsPerLevelStatValuesParam` 136 -> 148 bytes per row (300 -> 99 rows),
  `MenuStatsParam` 56 -> 60, `PlayerStatusParam` 284 -> 296. The NT's Agility block (three percentages + three
  multipliers) is still there, retuned: 15 / 15 / 15 -> 20 / 25 / 25 at row 1, 40 / 65.7 / 40 -> 100 / 100 / 100 at 99.
- **Same stat-to-attribute map** (`RelatePhysicalStatToLevelStatParam`, one byte per derived stat, attribute 1-11):
  NT `4 1 2 3 5 6 7 7 8 8 5 6 6 11 4 11 11 11 9 9 9 9 7 7 8 8 10 10 10 10 10 10 9 11 10 11` (36);
  vanilla is the same with two entries inserted (`4` at the start, a fifth `9`) (38). Every derived stat the NT drove
  from attribute 10 (Agility) is still driven by attribute 10: retail kept the slot and its table and computes it from
  Adaptability and Attunement instead of spending points on it.
- The stats-upload schema was rewritten properly: `tekioryoku` in Resistance's place; `binsyo` and a new
  `physical_toughness` among the derived stats; the three NT derived names gone.

## 4. The change came in steps, and the trap system kept growing first

- Vanilla `common.fmg` (the short status labels; the NT file has none in 909000-909052) is written for the new system
  (`ADP` at 909012, `AGL` at 909038, `Petrify RES`, `Ttl Souls`) and **still lists 909039 `Trap Dsrm`** right after
  `AGL`. So at some point the main game had Adaptability + derived Agility and a Trap disarm stat on the status screen.
- Added after the NT and later cut: key prompt `keyguide.fmg` 508 "Release trap" (505-510 are all new in vanilla), and
  item 60340000 **Rhoy's Stone of Knowledge** ("High chance of detecting treasure chest traps, indicated by a red
  light"): the NT has its ItemParam row but no text; vanilla has the text but no ItemParam row.
- `EventCommonParam` trap-type weights are retuned x10 (250 / 350 ...), with type 4 set to 1 / 1; the fuse rows of
  `EventCommonParamFloat` (3.0 / 4.5) are gone. Whether vanilla's chest code still has a stat-based disarm roll is the
  stage-1 code question.

Order this suggests: NT design (Aug-Sep 2013) -> Adaptability + derived Agility, trap disarm still a stat, trap features
still being added (detection item, release prompt) -> trap disarm cut, leaving the label, the item text, the prompt and
the param column -> vanilla data built 14-24 Jan 2014.

## 4b. The Japanese network test (added the same day)

`..\..\DS2 JP Network Test\findings\jp-vs-us.md`: JP 01.00 data 23 Aug 2013, US 17 Sep 2013, JP update 01.02 regulation
2 Oct 2013 (menus / model 3 Oct). One line of development (JP 01.00 -> US -> JP 01.02). JP 01.02 already took in
retail content (Aurous / Alva / Black Witch armour, Ring of Thorns, retail equip-load values, 15 retail param names as
empty entries) yet keeps the Network Test stat system in params, text and code. Consistent with the change coming after
early October; not proof, since the test's code could only use stat tables in its own layout.

## 5. Limits

- A fork date is not directly visible; only the NT's content and pack dates. An earlier fork with main-branch content
  copied in during August is possible, but then main's Adaptability (if it existed) should have leaked into the shared
  text, params or upload schema, and none did.
- Nothing dates the change inside 17 Sep 2013 - 14 Jan 2014, so "before or after the public tests" stays open.
  Places that might: the Japanese network test build (if it exists in a different version), press previews and
  interviews from Oct 2013 - Jan 2014 (a quick web search found none), or the NT's update package if it carries data
  newer than 17 Sep 2013.
- Text was extracted with `gamedata.py x`, which flattens paths: the per-map `bloodmes` and `talk` FMGs share
  names, so `text\en_menu\m*.fmg` are whichever was written last.

Sources for the public test dates: Game Informer (27 and 29 Oct 2013), PlayStation Blog (23 Oct 2013), Gamewatcher
(planned 12 Oct round), Game*Spark (5 Sep 2013, Japanese schedule change).
