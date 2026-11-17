# Unused attacks to try in game: swap candidates

!!! warning "Status: Plan, not run"
    A test or idea written down. Nothing here has been tried. See [status labels](../status-labels.md).

PC stage 3, 2026-10-25. A shortlist from `unused-attacks-pc.md`: attacks that are set up and still have an animation
path in the network, so swapping one in for an attack the enemy already uses should show it in play. That is how the
Amana sorceress's volley was proven.

Full table for all 104 playable unused attacks: `attack_swap_candidates.tsv` (tool `tools\attack_swaps_pc.py`).

**12 Nov re-run:** 23 of the 104 turned out to be combo moves. The AI never asks for them alone, but a combo it does
use plays their animations (`unused-attacks-pc.md`, "Combo moves"). The TSV now holds the remaining 81. Below, the
affected rows are marked *combo*:
- where the combo starts with the same hit, a swap shows nothing new;
- where it hits with its own rows (the Smelter Demon's, Lost Sinner's and Sinh's shot versions), the single version's
  shots may still be unseen.

## How a test works

```
uv run --no-project --python 3.12 python -I -X utf8 tools\pcmem.py attackswap <script> <donor> <attack> on
```

Run it from `<local path> Scholar PC`, with the game running (Steam offline) and the area loaded:
- It rewrites every place the enemy's loaded behaviour script asks for the **donor** attack so that it asks for the
  unused **attack** instead. The enemy then uses the unused one wherever it used to use the donor, until the area
  reloads.
- `off` puts the donor back. Leaving out `on` / `off` just shows the current state.
- Writes are offline-gated and logged in `saves\pcmem_writes.log`, as for the volley.

Things to know:
- **The donor is lost for the test.** The enemy can't use the donor attack while the swap is on.
- **The donor's choice logic stays.** The unused attack is used at the donor's range and odds. A shot swapped into a
  melee slot is fired at melee range.
- **"Sites"** = request bytes + readiness checks rewritten. A lot of sites means the donor is used in many modes.
- **Network Test** = the 2013 Network Test's script for that enemy asked for the attack, the best evidence that it was a
  real move that got cut.

## Best to test: complete on PC and visibly different (checked 2026-10-25)

What makes an attack testable:
- **Complete**: every projectile and damage row it names exists on PC, and its projectiles have a real effect (not
  0, not the placeholder 1300).
- **Distinct**: either a projectile attack on an enemy whose normal attacks never fire one, or an effect none of its
  normal attacks use.

Twelve pass both. The Giant Acid Horn Beetle's 15 is one, but its new effect (4024) turned out to look like its usual
acid clouds (4027 etc.) in game, so sibling effect ids can still look alike.

| enemy | attack | why it should stand out | projectile | command | where |
|---|---|---|---|---|---|
| **Undead Aberration** | 21 | a melee enemy firing a fast projectile | effect 1698, 30 m/s, 2 s | `attackswap 141000 14 21` | Aldia's Keep, Sinners' Rise |
| Undead Aberration | 20 | a melee enemy releasing a slow cloud | effect 1696, 1 m/s, 3 s | `attackswap 141000 14 20` | same |
| **Laddersmith Gilligan** | 21 | an NPC throwing something (if made hostile) | effect 174, 30 m/s, 5 s | `attackswap 704000 17 21` | Majula, Earthen Peak |
| **Basilisk** (NT) | 21 | a moving projectile instead of its usual stationary curse cloud | effect 1639, 12 m/s, 5 s | `attackswap 210000 20 21` | Amana (PC), Majula, Things Betwixt, Shaded Woods |
| **Black Hollow Mage** (NT) | 20 | a fast spell with the maidens' orb effect | effect 5000, 50 m/s, 2 s | `attackswap 118000 21 20` | Huntsman's Copse, Tseldora |
| Black Hollow Mage | 24 | a long-lived fast spell, new effect | effect 1640, 50 m/s, 8 s | `attackswap 118000 23 24` | same |
| Captive Undead / Hollow Prisoner | 20 | a melee enemy releasing a slow cloud | effect 1667, 1 m/s, 3 s | `attackswap 127000 12 20` | Huntsman's Copse, the Gutter |
| Giant Basilisk | 22 | new effect | effect 1638, 12 m/s, 5 s | `attackswap 222000 20 22` | Aldia's Keep, Huntsman's Copse |
| Giant Acid Horn Beetle | 15 | tested: looks like its other attacks | effect 4024 | `attackswap 220000 23 15` | Amana, Aldia's Keep |
| Sinh (DLC boss) | 16 *combo* | new effects (the animation plays as his combo 50, whose group names no rows; the shots may still be new) | effects 8153 / 8155 | `attackswap 681000 21 16` | Dragon's Sanctum |

**Broken on PC** (rows deleted after 1.00, or only a placeholder effect): 37 of the 104 playable attacks, listed by
`attack_swaps_pc.py`'s companion check:
- the Ogre's 20 / 21;
- the Giant Acid Horn Beetle's 22;
- the Prowling Magus's 21 / 23 / 25 and the Stone Knight's 20, though the NT's scripts requested both Magus 21 and
  Knight 20;
- the Manikins' 23 and the Gyrm's 20;
- the Dark Stalker's 13 / 16 and the Desert Sorceress's 22;
- the Giants' 13-15 (two variants);
- bosses: the Rotten, the Smelter Demon, the Demon of Song's 12 / 13, the Dragonrider, the Lost Sinner, Aava and the
  Burnt Ivory King's 16.
- Effect 0 or the placeholder: the sorceress's volley, the Black Hollow Mage's 12 / 22, the Giant Basilisk's 21, the
  Looking Glass Knight's 7, and the Leydia Pyromancer's 20 / 23 (these may draw their effect from the animation).

## Shortlist (by evidence; check the broken list above before testing)

### Near Amana (you are there)

| enemy | attack | kind | NT | command (script donor attack) | sites |
|---|---|---|---|---|---|
| **Basilisk** (4 at Amana; also Things Betwixt, Shaded Woods, Aldia's Keep) | 21 | shot | yes | `attackswap 210000 20 21` | 4 |
| Basilisk | 22 | shot | yes | `attackswap 210000 20 22` | 4 |
| Basilisk | 23 | shot | no | `attackswap 210000 20 23` | 4 |
| **Giant Acid Horn Beetle** (Amana, Aldia's Keep) | 22 | shot | yes | `attackswap 220000 23 22` | 1 |
| **Ogre** (Amana, Things Betwixt, Forest of Fallen Giants, Aldia's Keep) | 20 | shot | no | `attackswap 300000 15 20` | 2 |
| Ogre | 21 | shot | no | `attackswap 300000 15 21` | 2 |
| Undead Supplicant (Amana, Tseldora) | 10 | melee | no | `attackswap 147000 11 10` | 4 |

The Ogres' 20 / 21 are projectile attacks: a heavy enemy that never throws anything in the shipped game. The
Basilisk's three are extra shots beside its usual one (20).

**Checked 2026-10-25:**
- **Broken on PC:** the Ogre's 20 / 21 and the Giant Acid Horn Beetle's 22.
  - Their animations only fire projectiles, and the projectile rows (130000410 / 510, 122000410) were deleted between
    1.00 and PC. So the animation would play and fire nothing.
  - In 1.00 they existed: the Ogre's 20 was an invisible 0.3 s burst, its 21 a 2 s projectile with effect 1980, and
    the beetle's 22 a 0.6 s spray with effect 4025.
  - Restoring them would mean re-adding the rows.
- **Complete on PC:** the Giant Acid Horn Beetle's **15**. Two projectiles with effect 4024, 30 m/s, 6 s, and an
  animation that fires them. Live test: `attackswap 220000 23 15` (its normal spit, 23, becomes 15).
- **Scripts load only near the enemy:** `attackswap` reports "not loaded" until the enemy is close. At the Amana
  bonfire area the Ogre, the giant and small beetles, the Aberrations and the clerics were loaded; the Basilisks, the
  Supplicants and the Demon of Song were not.

### Network Test moves, elsewhere

| enemy | where | attack | kind | command | sites |
|---|---|---|---|---|---|
| Heide Knight (sword) | Heide's Tower | 10 (13 *combo*, same hit) | melee | `attackswap 301000 12 10` / `12 13` | 2 |
| Heide Knight (spear) | Heide's Tower, Lost Bastille | 10 (13 *combo*, same hit) | melee | `attackswap 301000 11 10` / `11 13` | 5 |
| Black Hollow Mage / White Hollow Mage | Huntsman's Copse, Tseldora | 12 | shot | `attackswap 118000 14 12` | 2 |
| Black Hollow Mage / White Hollow Mage | Huntsman's Copse, Tseldora | 20 | shot | `attackswap 118000 21 20` | 3 |
| Gyrm | Doors of Pharros | 12 | melee | `attackswap 134000 11 12` | 2 |
| Gyrm | Doors of Pharros | 13 | melee | `attackswap 134000 14 13` | 8 |
| Bonewheel Skeleton | Huntsman's Copse | 12 | melee | `attackswap 133000 11 12` | 3 |
| Lion Clan Warrior | Shaded Woods | 14 | melee | `attackswap 308000 13 14` | 3 |
| Prowling Magus | Aldia's Keep, Tseldora | 21 | shot | `attackswap 137000 20 21` | 2 |
| Stone Knight | Drangleic Castle | 20 | shot | `attackswap 150000 17 20` | 3 |
| The Last Giant (boss) | Forest of Fallen Giants | 15 | melee | `attackswap 309600 14 15` | 2 |

(12 Nov: the Last Giant's 15 is nothing new. It is the first swing of his combo 3, which he already uses, so the swap
only makes that one swing more frequent. See unused-attacks-pc.md.)

### Other good ones

| enemy | where | attack | kind | command | sites |
|---|---|---|---|---|---|
| Parasite Spider | Tseldora (99 spawns) | 14 | melee | `attackswap 203001 13 14` | 1 |
| Captive Undead / Hollow Prisoner | Huntsman's Copse, the Gutter | 20 | shot | `attackswap 127000 12 20` | 10 |
| Manikin (knife and shield) | Earthen Peak | 23 | shot | `attackswap 124000 22 23` | 3 |
| Leydia Pyromancer | Undead Crypt | 20 / 23 | shot | `attackswap 512000 21 20` / `22 23` | 2 |
| Desert Sorceress | Earthen Peak, Drangleic Castle | 22 | shot | `attackswap 316000 21 22` | 4 |
| Grave Warden (spear) | Undead Crypt | 18 (used in 1.00, removed in 1.10) | melee | `attackswap 312000 15 18` | 6 |

**Bosses** (once per playthrough, so only if not yet killed on the test save, or in NG+):

| boss | attack | kind | command |
|---|---|---|---|
| The Rotten | 20 | shot | `attackswap 326000 22 20` |
| The Rotten | 21 | shot | `attackswap 326000 22 21` |
| Smelter Demon | 13 *combo* (own rows) | shot | `attackswap 305000 14 13` |
| Throne Watcher | 17 | melee | `attackswap 334000 16 17` |
| Lost Sinner | 10 *combo* (own rows) | shot | `attackswap 626000 15 10` |
| Demon of Song | 12 / 13 *combo* (own rows) | melee | `attackswap 602000 11 12` / `14 13` |
| Sir Alonne | 15 *combo* (own rows) | melee | `attackswap 680000 14 15` |
| Sinh | 16 *combo* (no rows) | shot | `attackswap 681000 21 16` |
| Burnt Ivory King | 16 *combo* (own rows) | melee | `attackswap 690001 15 16` |

The melee *combo* ones (Demon of Song, Sir Alonne, Burnt Ivory King) are already seen inside their combos; a swap
only shows the first swing on its own.

The Varangian Sailor's 15 (row 113010, found on 12 Nov) isn't in the TSV, because it sits on a row no spawn starts
with. An untried command: `attackswap 113000 16 15`. 16 is the other attack only that row sets up, so the swap would
show only while the Sailor is using row 113010 (which weapon that is isn't known).
| Looking Glass Knight | 7 | shot | `attackswap 504000 6 7` |

Looking Glass Knight's 7 is a 0-9 "Action" attack, so it's less certain.

## Not checked

- None of these has been seen yet except the volley. Some may play oddly: missing effects, wrong range, an animation
  left unfinished.
- `attackswap` is tested only against files. Its first in-memory run will show whether each script sits in memory byte
  for byte, as ai105000 did; it refuses to write if any site differs from the file.
