# Bosses: vanilla 1.10 vs PC Scholar (with vanilla 1.00 for context)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-24. The author's question: were bosses such as the Last Giant and the Ruin Sentinels quietly nerfed in Scholar? Compared:
**vanilla 1.00** (BLUS41045 disc) → **vanilla 1.10** (the last vanilla patch; [its regulation is also PS3 Scholar's](../builds/1.08-to-1.10.md#3-109-110-data-built-26-nov-2014-with-scholar-two-layers), Cal.
1.13) → **PC Scholar 1.0.3.0** (regulation 2.02). The "Scholar step" below is 1.10 → PC, so it covers the Scholar
re-balance on the new-generation versions and the later PC patches. PS3 / 360 Scholar never got these changes.

Full output: `bosses_pc.txt` (every fight), `bosses_pc.tsv` (every changed field the fights reach, Scholar step),
`ai_compare_pc.tsv` (every AI script, both steps).

## Short answer

- **Last Giant: yes, a damage nerf.** Its damage-to-player multiplier went 1.0 (1.00) → 0.9 (1.10) → **0.8 (PC)**, on top
  of the 1.10 patch's cut to its attacks (659 → 622, 631 → 594 ...). So it hits about 11 % softer than in vanilla 1.10 and
  about 25 % softer than at launch. HP, defences, poise, AI, animations and placement are unchanged.
- **Ruin Sentinels (Lost Bastille boss): no change at all.** Stats, attacks, AI, animations and the three generators are
  identical in 1.10 and PC (the AI file differs only by an unused option of a shared routine).
- **The Drangleic Castle Ruin Sentinels were nerfed hard.** These are the five regular ones in the castle (row 325010, same
  AI and attacks as the boss): **HP 3,260 → 2,070 (−37 %)**, NG+ 4,564 → 2,898. This is probably where "Scholar nerfed
  the Ruin Sentinels" comes from.
- **No boss's HP changed in the Scholar step.** The nerfs are to damage: the Last Giant, Giant Lord, Royal Rat Authority and
  Ancient Dragon got lower damage-to-player multipliers, and single attacks of the Smelter Demon, Throne Watcher and
  Freja's spiders were cut.
- **No boss's own fight AI was rewritten.** The large AI differences come from new behaviour modes for Scholar's new
  [enemy placements](../builds/placements-balance-pc.md#1-enemies) (extra Pursuers, Dragonriders, a second Chariot, more Gargoyles), which the boss fights never run.
  The real fight-AI changes are small: Aldia gained retry / timeout fail-safes, the Lost Sinner and Sir Alonne had a
  close-range escalation counter reworked, and the Lost Sinner had a broken range check fixed.

## Scholar step (1.10 → PC): every boss change

| Boss | Change | Effect |
|---|---|---|
| Last Giant | damage-to-player multiplier 0.9 → 0.8 | −11 % damage (−25 % vs 1.00, with the 1.10 attack cuts) |
| Giant Lord | damage-to-player multiplier 1.0 → 0.8 | −20 % damage |
| Ancient Dragon | damage-to-player multiplier 1.0 → 0.6; an unnamed target flag (`target4`) set on all 13 attacks | −40 % damage |
| Royal Rat Authority | damage-to-player multiplier 0.9 → 0.7 (1.00: 1.0) | −22 % damage |
| Smelter Demon | 2 of 12 attacks 747 → 515 (1.00: 686) | −31 % on those two |
| Throne Watcher | 1 of 23 attacks 581 → 317 | −45 % on that one |
| Freja (spiders) | 1 of 8 spider attacks 302 → 220; spider AI reworked (see below) | Freja herself unchanged |
| Skeleton Lords | minion spawn points in the arena: Bonewheels 4 → 2; Falchions skeletons 2 → 3 (new row, 0 souls); Estoc & Shield skeletons 4 → 12 (new row, HP 290 → 220, 0 souls); Falchion & Shield 2 (same) | fewer Bonewheels, more but weaker skeletons (how many each Lord releases is set by map events, not compared) |
| Flexile Sentry | body capsule height 3.8 → 3.3; Suspicious Shadows (the NG+ ones in the fight and the Wharf's): 12 attacks' second damage part 50 → 65, souls 250 → 350 | |
| The Rotten | body capsule radius 2.2 → 3.0 | wider body collision |
| Prowling Magus & Congregation | Undead Supplicants: 3 attacks gain a second damage part of 15 (type 6) | |
| Pursuer | one animation (75017100): invincibility 0.000–0.067 → 0.000–0.167 s | tiny |
| Nashandra | weapon effect id set (visual) | none |
| Vendrick, Chariot's mages and skeletons, Flexile's shadows | co-op time reduction lowered (120 → 100, 60 → 48, 12 → 10 / 8) | co-op only |

Stats and attacks unchanged in the Scholar step: Dragonriders (Heide's and the Drangleic twins), Old Dragonslayer,
Royal Rat Vanguard, Belfry Gargoyles, Ruin Sentinels, Lost Sinner, Mytha, Covetous Demon, Najka, Old Iron King, Demon
of Song, Velstadt, Looking Glass Knight, Throne Defender, Darklurker, Guardian Dragon, Freja herself, Executioner's
Chariot, Pursuer, Aldia, and every DLC boss (Fume Knight, Sir Alonne, Sinh, Elana, Aava, Lud & Zallen, Burnt Ivory
King, Graverobber / Varg / Cerah, Blue Smelter Demon). AI changes for some of these are listed under AI below.

The NG+ damage tables (ChrRoundDamageParam) are unchanged for every boss. The one table that changed (row 0) is used by
no enemy on PC.

**What "damage-to-player multiplier" is.** EnemyParam `vs_player_damage_mult`, next to `vs_enemy_damage_mult` and
`vs_npc_damage_mult`. Reading it as damage the enemy deals to players fits every value seen: Brume Tower's Barrel
Carriers carry 6.5 against enemies (their barrels are meant to blow up other enemies), many bosses carry 0.01–0.25
against other enemies (little friendly fire), DLC NPC invaders such as Rockshield Baldyr and Quicksword Rachel carry
1.3 against players, and the Suspicious Shadows that come in groups carry 0.65. This has
not been measured in game yet. A hit from the Last Giant with the value patched in memory would confirm it.

## AI (ezstate scripts)

How a boss's AI is wired: its generator names two scripts. `ai<LogicParamID>` is the "upper logic" that picks a mode;
`ai<DefaultLogicParamID>` holds the behaviour groups the logic calls (for example, the Last Giant's logic ai309610 calls
groups 10 / 20 / 30 of ai309600). Each script also carries copies of shared routines (approach, circle, guard ...).

Every script was decoded on both platforms into one form (`tools\esd.py`). The comparison then removes compiler
artefacts: register caching, the numbering of internal sub-machines, and renumbered states. Changes inside shared
routines are counted once. Last, each change is checked for whether the groups the boss's own logic calls (in both
builds) can reach it. Of 528 / 485 scripts, 182 have own changes in the Scholar step, but for the bosses:

- **Unchanged fight AI** (identical, or only shared-routine updates whose new option the boss never passes): Last Giant,
  Ruin Sentinels, Belfry Gargoyles, Skeleton Lords, Dragonriders, Executioner's Chariot, Royal Rat Vanguard, Smelter
  Demon, Flexile Sentry, Demon of Song, Vendrick, Nashandra, Ancient Dragon, Darklurker, Sinh, Guardian Dragon, Freja.
- **New modes that only Scholar's new placements use.** These look large, but the boss fights don't run them:
  - the Pursuer: groups 110 / 120 / 130 with "evading" and "evading at range" states, for the Bastille Pursuers and
    others with think ids 110–130. The boss uses think ids 10 / 100.
  - the Dragonriders: groups 200 / 210, for the new Undead Crypt and [Shrine of Amana riders](../builds/shrine-of-amana-pc.md#3-the-other-enemies).
  - the Executioner's Chariot: group 200, for a second Chariot in Drangleic Castle.
  - the Belfry Gargoyles: new modes for the gargoyles in Drangleic Castle and Belfry Sol. PC also deletes four groups
    the boss logic never called.
  - the Skeleton Lords: groups 200–213. No PC generator uses them.
  - the Congregation's priests, prisoners and supplicants: modes for their new placements.
- **Changes that do reach a boss fight:**
  - **Aldia**: three attack states (warp, strengthen the flame aura, all-round attack) now leave after a failed start or a
    timeout (1–8 s). The new debug labels read "attack failed" and "action N: … failed". In 1.10 these states waited
    for the attack to finish with no way out, so this looks like a fix for Aldia stalling.
  - **The Lost Sinner and Sir Alonne: close-range counter.** No other enemy has this counter. Their upper logic (group 11)
    runs rounds of checks; at each check, if the target is within 5 m (`f7001() < 5`, very likely the distance), it
    raises logic slot 4 by one step (0 → 37 → 38 → 39). Their approach / circling routine reads 39 ("close at three checks
    in a row") and breaks off after 2 s so they commit to an action. In 1.10 a round has three checks. On PC it has four,
    and a fourth close check raises the counter to 40, which no script reads. The effects:
    - staying close the whole round now holds 39 only from the third check to the fourth, not to the end of the round;
    - close / close / far / close now also reaches 39;
    - when the second close check falls on the third check, the counter reaches 38 and the round ends without its fourth
      check (that branch jumps past check 4, which looks like a slip).
    Alonne's PC logic also takes the Sinner's start condition (slot 5 = 0). It carries an exact copy of the old
    three-check counter as group 100, and nothing in the scripts shows when that group runs.
  - **The Lost Sinner: impossible test fixed.** Two of her branches tested `slot 4 > 38 and < 39`, which no step can
    pass, so they never ran in 1.10. PC tests `< 40`, so at 39 she now repositions (50 % roll) or turns to face you
    (60 %) instead of carrying on.
  - **Freja's spiders** (all of Brightstone Cove's, including the 12 in the boss fight): the behaviour script was renamed
    ai203000 → ai203001 and largely rewritten (~500 changed lines: new approach and attack-choice routines). Freja's
    own AI is unchanged.
  - **The Congregation** (Prowling Magus fight): its members' modes now call a revised copy of a routine many enemies
    share. It adds conditions on f5641 / f5642 / f7007 before its 5204(30) / 5204(31) actions, so their reactions differ
    slightly in some situations.
  - **Every "x010" upper-logic script** of the Smelter Demons, Pursuer, Darklurker, Ancient Dragon, Chariot, Nashandra and
    Rat Authority: three parallel watcher groups (11–13) became four (11–14), renumbered 0 / 1 / 2 → 0 / 2 / 3 / 4. Their
    job isn't decoded yet (each runs a sub-machine whenever slot 2 is in mode 39).

The script language has no names for its functions (f5900, f7001 ...), so these are structural readings. Only the
debug labels and the arithmetic in conditions give the meaning.

## Animations (TAE)

`timeact\chr\c<behaviour id>_pl.tae` was compared per animation: event windows exactly, and added or removed events. For
the bosses, the only timing change in the Scholar step is the Pursuer's one-animation invincibility window (above).
Freja's spiders gained a few effect events in their 7501xxxx animations. The Last Giant's and Ruin Sentinels' files
are identical. Parameter bytes are only compared in their first 16 bytes across platforms, because later words are
32-bit (PS3) or 64-bit (PC) pointers. Of the 135 TAE files that differ, almost all differ only in those pointer words.

## Vanilla patches (1.00 → 1.10), for context

- Every boss attack [lost weapon durability damage](../builds/1.01-to-1.03.md#102-103-data-scripts-and-a-second-balance-pass-calibrations-103-104) (5 → 4), and all bosses moved to new NG+ tables (ids +30).
- **Attack damage was tuned both ways**:
  - down: Last Giant 659 → 622 on its main attacks, Freja two attacks 880 → 697;
  - up: Smelter Demon two attacks 686 → 747 (then 515 on PC), Pursuer one attack 427 → 464, Chariot's Black Hollow
    Mages +9 %, Congregation priests and magus +9 % (with more poise damage).
- [Lost Sinner HP](../builds/balance-vanilla-vs-scholar.md#7-enemies) 3,900 → 3,560. Last Giant damage-to-player multiplier 1.0 → 0.9, Royal Rat Authority 1.0 → 0.9.
- AI edits: Ruin Sentinels 4 changed lines; Looking Glass Knight 18; Freja's spiders 95; the Skeleton Lords' skeletons
  81 (`ai_compare_pc.tsv`).

## How it was done

- `tools\bosses_pc.py`: fights from the community SOTFS sheet, rows from each build's own generators (GeneratorParam →
  GeneratorRegistParam). Each row's whole chain is compared: EnemyParam, Common, Move, LogicCom, the four behaviour
  params, AttackMove, Damage, Bullet, DamageCtrl, EquipWeapon, DefaultLogic and NG+ tables; LogicEzStateSharedParam is
  compared whole (no change). PS3 rows are read through `balance_pc.ps3_decoder`.
- `tools\ai_compare_pc.py`: all AI scripts, both steps. `show <id>` prints one script's changes, `lib` the shared-routine
  changes; `summary`, `focused` and `logic_roots` are importable.
- `tools\tae_compare_pc.py <id>...`: TAE per animation, both steps.
- `..\DS2 Network Test\tools\esd.py`: ezstate reader for both platforms. Bytecode decoded to expressions; registers
  substituted; sub-machines paired by content; states aligned by content. `dump`, `diff`.

Open: the damage-to-player multiplier is still to be confirmed in game. The upper-logic watcher groups (11–14) and the
function ids are not decoded. Map event scripts, which time things like when a sentinel joins, were not compared.
