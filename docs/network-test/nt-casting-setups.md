# Network Test spell casting: eight cast setups, aiming catalysts and catalyst power stance (2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

Status: from the data, and live in RPCS3 (US Network Test NPUB90979, LLVM boot) on 17 November 2026. Draft for
review: the readings marked as readings are not proven.

Prompted by a Japanese "matome" blog article of 12 July 2014 (dark-soku.blog.jp, article 9600318), which reposts a
forum thread of 10 July 2014 where players listed things they remembered from the Network Test and could not find in
the finished game. Section 6 checks every claim in it.

## Summary

- **Every NT spell has eight cast setups**, each with its own animations, projectile and damage row. The damage rows
  step from ×0.875 to ×2.1 of the first one. Retail keeps the eight slots but makes them identical, so its casts all
  hit the same.
- **Live, setups 0-3 are button × grip, in either hand:** light cast (R1 / L1) = 0, heavy cast (R2 / L2) = 1,
  two-handed light = 2, two-handed heavy = 3. Two-handed heavy Soul Arrow did 328-336 where a light cast did 137-153.
- **Two-handing a staff or chime gives a bow-style aiming mode** on the opposite trigger, with a reticle, in either hand
  (found live by the author). In the data it is the ranged weapons' aim action. Retail removed it from catalysts.
- **Catalysts cannot power stance**, even at 99 in every stat, although the NT data gives staves and chimes
  power-stance categories of their own (retail sets both to 0). Setups 6-7 start with the power-stance animation, so
  they are most likely the power-stance casts, which is why they never fired.
- **The NT Lightning Spear bursts on impact:** a burst of follow-up projectiles adds two extra hits of about 30 %.

## 1. Eight cast setups per spell

### The data

Each NT SpellParam row (320 bytes) holds eight blocks of seven values, one block every 0x1c bytes from +0x18:

| Offset in block | Value |
|---|---|
| +0x00 | time A (0.025, 0.2 or 0.55 s by setup; the community calls it start-up duration) |
| +0x04 | time B (the same in all eight blocks of a spell, e.g. Soul Arrow 0.15, Lightning Spear 0.25) |
| +0x08 | start-up animation |
| +0x0c | cast animation |
| +0x10 | projectile: BulletParam id = spell id + setup number |
| +0x14 | damage row: PlayerDamageParam id = spell id + setup number (Soul Arrow and Soul Spear put a shared row, 30000110, here; their projectiles name the per-setup damage rows instead) |
| +0x18 | a multiplier, the same on all 69 spells |

The pattern is the same across spells:

| Setup | Time A | Multiplier (+0x18) | Damage row +0x24, relative to setup 0 | Animations |
|---|---|---|---|---|
| 0 | 0.025 | 1 | ×1 | one-handed (3x11xxxx) |
| 1 | 0.2 | 1.25 | ×1.4 | one-handed |
| 2 | 0.2 | 1.25 | ×1.5 | the only setups that use two-handed animations (3x13xxxx), as start-up or cast animation depending on the spell |
| 3 | 0.55 | 1.5625 | ×2.1 | as setup 2 |
| 4 | 0.025 | 0.5 | ×0.875 | one-handed |
| 5 | 0.2 | 0.625 | ×1.225 | one-handed |
| 6 | 0.2 | 1.25 | ×1.125 | the only setups that use the power-stance start-up animation 3x150010 |
| 7 | 0.55 | 1.5625 | ×1.575 | as setup 6 |

- Time A follows this pattern on 64 of the 69 spells.
- The damage factors hold on all 35 spells that have per-setup damage rows. The +0x24 float sits where retail keeps
  its damage multiplier (right after the damage-type bytes), so "damage" is a reading by position, which the live
  results bear out. A second number in the same rows (+0x28, 16 bits) also changes by setup: 183, 199, 274, 299, 160,
  174, 80, 87. Its meaning is unknown.
- The +0x18 multiplier sits where retail keeps the value the community calls stamina cost. The author saw stamina use
  differ between setups live; the amounts were not measured.

**Retail** shrank each block to five values (the two times, the two animations and the multiplier) and moved the
projectile and damage ids to the row, one pair per hand. In 195 of retail's 196 spells all eight blocks are
identical, and each spell has one damage row. So in retail the setup cannot change the projectile or the damage.

The community's field names for retail's eight blocks (left / right hand × one-handed, two-handed and two unknowns)
are guesses, since retail's blocks are identical. For the NT they are wrong: the pairs are light / heavy, not hands.

### Live: which press fires which setup

Tool: `DS2 Network Test\tools\castwatch.py` (section 7). Each cast was identified by a record the game rewrites at
every cast with the projectile id (spell id + setup), and by the setup's projectile and damage ids appearing in
memory. Soul Arrow, INT 40, enemies with 425 HP:

| Setup | Press | Damage (each a separate cast) |
|---|---|---|
| 0 | R1, or L1 with the staff in the left hand | 153, 141, 137, 140, 137 |
| 1 | R2, or L2 | 213, 191, 189, 199, 196 |
| 2 | two-handed R1 / L1, including aimed casts | 225, 202, 232, 202, 209 |
| 3 | two-handed R2 / L2, including aimed casts | 328, 336 on fresh enemies; four more killed enemies with 200-223 HP left |

- Either hand uses setups 0-3 the same way (a run with the staff only in the left hand matched the right-hand runs).
- Casting from the aiming mode used setups 2 and 3. Aiming changes where the cast goes, not its damage.
- Against setup 0's median (140) the ratios are about ×1.4, ×1.5 and ×2.4. The data says ×1.4, ×1.5 and ×2.1: the
  first two match; two-handed heavy came out higher on both clean hits. The same cast varied by about 10 % from enemy
  to enemy, and enemy defences were not read, so this is "close to the data, with two-handed heavy above it".
- **A killing blow is capped.** HP stops at exactly -1, so the last hit only counts as the HP left plus one. The
  uncapped two-handed heavy values are the two fresh-enemy hits.
- Setups 4-7 never fired in any run (see section 4).

## 2. Lightning Spear's burst

The NT Lightning Spear projectile (32172000 + setup) names a follow-up projectile, 32272000 + setup (at +0xd0 / +0xd4
of the NT's BulletParam row). The follow-up lives 0.15 s and spreads over 120° (+0x40 = 2.0944 rad). Its damage row
has +0x24 = 0.6 against the bolt's 1.98 (setup 2), about 30 %.

Live, one two-handed light cast (setup 2) with the Dragonknight's Bell hit for 185, then 56 and 58. So an NT
Lightning Spear that hits lands a burst of extra hits around the target. That fits the 2014 memory of it being "like
a lightning version of Dark Hail".

## 3. Aiming with a two-handed staff or chime

Found live by the author: with a staff or chime held two-handed (either hand), the opposite trigger puts the character in
the same aiming mode as a bow, with a reticle.

**In the data.** WeaponActionCategoryParam says what each button does per grip; its two-handed off-hand light field
(`left_2h_light_1`, field 50) names a WeaponAttackMotionParam row. In the NT:

| Weapon | Two-handed off-hand light (L1) | Two-handed off-hand heavy (L2) |
|---|---|---|
| Witchtree Branch (8407) and every other NT staff action row | 20002510: attack type **16** | 20002500: attack type 13 |
| Dragonknight's Bell (8409) and the other chimes | 20005110, or the staves' 20002510: attack type **16** | 20005100 / 20002500: attack type 13 |
| Disc Bell (8419) | a shield's action (10066310, type 2) | 10066810 (type 2) |
| Short Bow (8411) | 20005800: attack type **16** (the bow's aim) | 20006400: attack type 13 |
| Light Crossbow (8412) | 20007400: attack type **16** | 20008000: attack type 13 |
| Dagger (8400), Longsword (8402) | attack type 13 | 2 / 13 |

Across all NT weapons, attack type 16 on that button appears only for staves, chimes, bows, greatbows, crossbows and
binoculars. Melee weapons use 13 there and shields 2 or 5. So type 16 is the aim action, and the NT gave it to
catalysts.

**Retail** points every staff's two-handed L1 and L2 at 20002500 (type 13), and the chimes' at 20002500 or 20005100
(type 13); the Disc Chime keeps the shield action. Rows 20002510 and 20005110 survive in retail, still type 16 but
re-pointed at the bow's aim animation (20010010), and no weapon uses them. Retail players aim spells with the
binoculars, a workaround; the NT had aiming built into the catalysts.

**Restoring it.** A plan for a PC Scholar mod that puts the aim back is in
`..\..\DS2 Scholar PC\findings\catalyst-aim-mod-plan.md`. It changes one field on 23 rows; whether the PC code still
supports aiming with a catalyst is untested (parked 17 Nov).

## 4. Catalyst power stance, and setups 4-7

**Live.** Power stance works in the NT exactly as in retail (hold Triangle with a matching pair; a pair that does not
qualify two-hands instead). Two daggers power stance. Two staves, two Dragonknight's Bells, and every mix of dagger,
staff and chime did not, even with every named stat at 99 and a level-up to make the game recalculate.

**The data says catalysts were meant to.** WeaponTypeParam gives each weapon type a power-stance category (NT +0x38;
retail's `powerstance_category`), and a pair power stances when the categories match:

| Weapon type | NT | Retail v1.00 |
|---|---|---|
| Dagger | 30 | 30 |
| Straight sword | 50 | 50 |
| Crossbow | 180 | 180 |
| **Staff** | **190** | **0** |
| **Sacred Bell / chime** | **200** | **0** |
| Bow, shields | 0 | 0 |

The NT staff action rows also have their own power-stance actions: the right-hand ones are the normal cast actions and
the left-hand ones (20002200-20002400) are cast actions too. So something outside the params refuses catalysts (a code
check, or a feature not finished for the test), and retail then removed it from the data as well. The code was not
traced.

**The animation numbering ties setups 6-7 to power stance.** In the player's animations the fourth digit marks the
grip:

| | One-handed | Two-handed | Power stance |
|---|---|---|---|
| Dagger attacks | 2101xxxx | 2103xxxx | 2105xxxx |
| Longsword attacks | 2901xxxx | 2903xxxx | 2905xxxx |
| Staff casts | 38110010 (setups 0-1) | 38130020 (setups 2-3, confirmed live) | 38150010 (setups 6-7) |

So setups 6-7 are the power-stance casts, and they never fired because catalysts cannot enter power stance. Setups
4-5 are most likely the other button pair in power stance: one-handed animations, ×0.875 / ×1.225 damage at about
half the stamina multiplier. That would make the scheme: in power stance, one pair of buttons casts a slightly weaker,
cheaper version (4-5) and the other casts with the power-stance animation and two-handed timings (6-7). This is a
reading, not tested; forcing the stance state in memory with staves equipped would test it.

## 5. Side find: a Blue Flame prototype

NT test weapon 9102 (no item, no name) uses model 1280, retail's **Blue Flame** (1280000), the sword that casts
sorceries. Its weapon type (145) casts sorceries and pyromancies and has the staves' power-stance category, 190. In
retail, type 145 casts sorceries only and has the straight swords' category, 50.

## 6. The 2014 players' claims

| Claim (forum thread or comments) | Verdict | Evidence |
|---|---|---|
| Staves could cast pyromancies | Confirmed | NT pyromancies use the staff animations; all cast with a staff live (NT session 11) |
| There was a pyromancy called 炎の波 | Confirmed | NT Flame Lash 炎の波, retail Fire Whip ([armour, rings and spells: Network Test vs retail](armour-rings-spells-nt-vs-retail.md)) |
| War (戦争) was the forerunner of Sacred Oath | Confirmed | matched by description and cast animations (same file) |
| Sacred Oath went from 1 slot to 4 | Confirmed | NT War uses 1 attunement slot, retail Sacred Oath 4 |
| Two-handing a catalyst made spells stronger; R1 and R2 differed; "charged" casts | **Confirmed live** | sections 1 and 4; the heavy cast's longer wind-up may be what players called charging. No separate hold-to-charge was seen |
| Zweihander scaled STR A / DEX D | Half right | [STR 1.7 = A (A is 1.6-2.0)](../mechanics/scaling-letters-and-infusion.md#letter-cut-offs-feweaponthreshold); DEX 0, rising to 0.27, below D's 0.4 (no grade). It shares the general scaling row with 111 weapons, the Great Axe and the catalysts among them |
| Arrows flew much faster | Confirmed (data) | NT arrows and bolts: speed 50 for 1.2 s. Retail arrows 40 for 1.5 s (same 60 m reach), bolts 30 for 1.4 s. Bow-specific modifiers not checked |
| Lightning Spear was like a lightning Dark Hail, with huge knockback | Partly | the impact burst (section 2); knockback not read (the NT damage layout is not decoded) |
| An invader killed someone with lightning sprayed from the hand | Not identified | the NT's provisional 放電 ("discharge") miracle has empty placeholder projectiles; the Lightning Spear burst is a candidate |
| Raising a shield slowed walking (removed after complaints) | Open | no named guard-speed field; the player's ChrMoveParam row differs from retail in dozens of unnamed fields |
| Invaders and white phantoms could drink Estus | Open | the Estus Flask's usage row (ItemUsageParam 1500) differs between the builds, but its flag bits are not understood |
| The weapon screen showed "weapon length" | Not supported | no such stat in the NT menus; the only distance stat is 射程距離 (shot range, for bows), which retail also has |
| 技量転化: "a sorcery used in swordplay, turns INT into DEX" | Quoted correctly | the NT's own text for item 30108000, which has no spell row (`cut-item-descriptions.md`) |
| 肉体強化 ("body strengthening") was also sealed | Not supported | not in any NT text |
| Break items: dissolving stone, dragon's crimson fluid, sulphur jellyfish pills | NT text | their short descriptions match the NT's own; already in [Break element](break-element.md). The same list mixes in [retail-only leftovers](cut-content-scholar.md#3-items) (Barrel, Illusory Rings, Pungent Ooze) |

## 7. How it was tested

- **Character.** A starting-class character (SL 21) was set up by host-side memory writes: stats with
  `statprobe.py set`, the Witchtree Branch, Dragonknight's Bell and Daggers with `weapon_swap.py give`, and Soul Arrow
  and Lightning Spear written into the attunement slots and made permanent with the cheat table's "Make current" entry.
  Attunement written to memory only gave slots after a level-up; spells then had charges after a rest. Save backed up
  first: `DS2 Network Test\saves\20261117_144943_before_castwatch_setup`.
- **Watcher.** `castwatch.py` reads guest memory from outside the game (no debugger, works on an LLVM boot). It finds
  the loaded BulletParam and PlayerDamageParam in memory, scans the heap about ten times a second for new copies of
  the attuned spells' per-setup projectile ids, damage ids and row pointers, polls the cast record (0x31f3a24c /
  0x31f3a25c on this boot), and logs every enemy's HP drop with signed HP. Logs:
  `DS2 Network Test\Ghidra Project\export\win\castwatch\*.jsonl` (four runs: right hand, retest on fresh enemies,
  left hand only, power-stance tries).
- Scattered single row-pointer hits for setups 4-7 in the 0x340xxxxx region came with no cast record and no projectile,
  and are treated as noise.

## Open

- What makes catalysts refuse power stance (code), and whether setups 4-7 really are the power-stance casts: forcing
  the stance state in memory with staves equipped would test it.
- Whether the +0x18 multiplier is the stamina cost (measure stamina per setup).
- The NT damage row's other per-setup number (+0x28) and the knockback values.
- The shield-walk slowdown and phantom Estus claims.
