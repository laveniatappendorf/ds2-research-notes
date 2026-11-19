# Aava leap live test plan (PC Scholar 1.0.3.0, planned 2026-11-13)

!!! warning "Status: Plan, not run"
    A test or idea written down. Nothing here has been tried. See [status labels](../status-labels.md).

Follow-up to [Aava's leap](../mechanics/aava-leap-distance.md#8-run-off-distances-and-the-race-13-nov), section 8. The frame-by-frame model of Aava's leaps failed its own control: it
had leaps 14 and 15 missing a player standing still 12-20 m away. The data can't settle whether a sprint outruns a
launched leap, so this session measures it in game.

## Questions

1. **How long does the distance keep updating?** Event 125 sets the counter `[chr+0xB8]+0x504` while it runs. As clip
   fractions (what [the Najka work](../mechanics/najka-damage-cuts.md) found) the windows are about 1.0 s (14), 0.9 s (15) and 0.67 s (16) of clip time.
   With the slowed take-off (findings section 9, added 13 Nov) that is about **1.37 / 1.27 / 0.83 s real**. Read as
   seconds they would be 0.40 / 0.36 / 0.235 s of clip (0.80 / 0.72 / 0.29 s real), which would freeze each leap's
   length at take-off. The counter's rise and fall settle this, and its length also checks the slow-down
   (`attack_speed` 0.5 / 0.5 / 0.8). Same check: her landing hit should open about 1.10 s (14, 15) / 0.92 s (16)
   after the counter rises.
2. **Standing control.** Does each leap land on a player standing 10, 15 and 20 m away?
3. **Sprinting straight away**, started (a) as she lines up, (b) as she leaves the ground. Which versions land, and
   from which starting distances?
4. **Real reach.** Where her landing hitboxes are relative to her ground point, and their radii, read from the live
   damage colliders. This is the piece the model was missing.
5. **The run-off.**
   - Which of the two spots she picks: the one nearer her, the one farther from you, or random.
   - Her dash speed (the clip says about 15 m/s).
   - The gap when she rolls her next attack, and what she picks.
   - Whether a leap picked from beyond 20 m is dropped when she can't reach 15 m within 5 s.

Side questions the same logs answer:
- your sprint speed (about 6 m/s assumed) and how long you can keep it up;
- whether her real travel matches the clips' Master bone (6.5 / 20 m for leap 14 and so on), which settles the
  `Character` node question;
- the hop-back length (20 m expected).

## What's needed

- **From the author:**
  - The game at Aava's fight. Either a save at her fog, or the Boss Arena's Aava entry (Outer Wall 37650, which sets
    the Eye of the Priestess flag 537000012 so she is visible). PC saves are disposable.
  - Steam in Offline mode.
  - At the controller for the whole session. No rolling needed; only standing, walking and sprinting.
  - About 45-60 minutes, mostly waiting for her to leap or run off.
- **From me, before the session (no game needed, about 1-2 hours):**
  - **`tools\aavalog.py log <s> <out.jsonl>`, read-only.** It builds on `hitlog_pc.py`'s frame loop: one record per game
    frame, read between updates with a stability check. Fields:
    - the frame counter (GameManagerImp +0x104);
    - you: position (`[[PlayerCtrl+0xF8]+0xF0]+0x20`), HP, stamina (+0x1AC);
    - Aava, found with `pcmem.live_chrs([679100])`:
      - the position the game measures distance from (`[chr+0xB8]+0x30`) and her position object (`[[chr+0xF8]+0xF0]+0x20`);
      - her HP;
      - the distance counter `[chr+0xB8]+0x504` and the Target_X / Y counter +0x508;
      - the attack id her AI last requested: ChrAi +0x35C, chain `[[[chr+0xE8]+0x18]+0x20]` from the
        repeat-move work, not yet checked live;
    - every live damage collider: position, shape (radius, capsule ends) and damage id. Her leaps are told apart by
      damage id: 1679004xx = 14, 1679005xx = 15, 1679009xx-10xx = 16.
    - While logging it prints the live distance to her once a second, so you can stand at a given distance.
  - **`aavalog.py analyse <log>`.** It splits a log into leaps (the counter rising, and the damage ids) and
    run-offs (her speed above about 8 m/s outside an attack). Per leap:
    - version, gap when the counter rises, gap when it falls, gap when the first hit capsule appears;
    - her travel; how close the capsule surfaces came to your hurtbox ([the ragdoll capsules](../mechanics/phantom-hitboxes-pc.md) `hitlog_pc` already reads);
    - whether your HP dropped.
    Per run-off: start and end points, the nearer spot, speed, gap at her next roll, the next attack id, and whether
    a leap was dropped.
  - **Offline checks.** The analyser runs on made-up logs with known answers. The memory chain is tested against a
    saved snapshot, the way `ct_pcmocktest.py` does for the CT.
  - **Integrity check** (read-only, in the session): `pcmem.py attackswap 679000 14 14` only shows the sites, and says
    whether the loaded `ai679000` matches the file. That confirms the arena build (if used) leaves her AI untouched.
  - **Stretch, only if quick: read the Target_Distance value itself.**
    - The setter `FUN_1404e2890` passes the network instance `[driver+0x40]` and the parameter's node id
      `[[driver+0x48]+0x24]` (index 0x12; 16 expected) to `FUN_1404db590`.
    - The driver is the object whose update is `FUN_1403a4950` (vtable slot at 0x1410E6A40) with `+8` = Aava's chr.
    - In a probe, find the value inside the network instance by matching the logged distance.
    - Questions 1-5 don't need this. The counter and the positions are enough.

## Session steps

0. **Pre-flight.**
   - Steam Offline. `pcmem.py status` (version, offline check).
   - Load at her fog or the arena bonfire.
   - Start `pcmem.py nodeath 3600 topup` in the background so a leap can't end the session. It WRITES min HP 1 and
     tops HP up, offline and logged. If you're in the Boss Arena build, its Can't die option works instead.
1. **Probe, 2 minutes.** Enter the fight. `aavalog.py probe` finds her and prints both positions, the counters and the
   attack id while she moves. Check that the fields move sensibly. If the ChrAi chain is wrong, the damage ids still
   identify the leaps.
2. **Run A, standing control.** Stand still at about 10, 15 and 20 m (the logger calls out the distance) and don't
   move when she leaps. Aim for 2 or more leaps of each version. I log 3-5 minutes at a time and report after each.
   Leaps are most likely at 12-18 m, just after her ranged attack (its 25 s cooldown pushes her rolls toward leaps).
3. **Run B, sprint as she lines up.** As soon as she turns to face you and crouches, sprint straight away. Same
   distances, 2 or more per version.
4. **Run C, sprint at take-off.** Wait until she leaves the ground, then sprint straight away.
5. **Run D, the run-off race.**
   - Stay within 8 m until she dashes off. That's about a 1-in-5 chance each time she starts a new attack cycle.
   - Then sprint the opposite way and keep going.
   - 4-6 run-offs, from different parts of the arena: middle, near the entrance, near the far end.
6. **Run E, optional: the hop back.** Stay within 7 m in front of her or at her side; log a few hops.

After each run I analyse at once and say which questions are answered, so the session can stop as soon as it has
what it needs.

**If leaps are too rare** (optional, only with the author's yes): `pcmem.py attackswap 679000 22 14 on` WRITES her
ranged-attack sites as leap 14 (see [attack swap candidates](../mechanics/attack-swap-candidates.md#how-a-test-works)). That changes her behaviour, so it is used only for extra leap 14 samples and switched
`off` straight after. Checked offline first: the tool must find the literal 22 passed to her attack routine.

## What the results mean

| Result | Meaning |
|---|---|
| Counter up about 1.37 / 1.27 / 0.83 s (real) | Clip fractions and the slowed take-off confirmed; the distance updates through most of the leap |
| Counter up about 1.0 / 0.9 / 0.67 s | Clip fractions, but no slow-down (findings section 9 wrong) |
| Counter up about 0.80 / 0.72 / 0.29 s | Seconds; each leap's length is fixed at take-off |
| Standing: every version lands at 10-20 m | Expected; the model's failure was the reach or the distance rule, and the logs show which |
| Running from line-up: a version misses | That version can be outrun; your friend probably meant it (the grab, 16, is the likeliest) |
| Running: all versions land | Outrunning isn't possible with sprint alone; the friend's result came from something else (e.g. a leap picked from beyond 20 m and dropped) |
| Capsule reach measured | Feeds back into `race.py`, which can then predict distances not tested |
| Run-off spot choice | Tells you which end of the arena she runs to, so you can plan where to fight |

## Risks

- **Offsets not yet checked live:** her measured position (`[chr+0xB8]+0x30`) is by definition what the game uses,
  but may sit off her visual centre; the ChrAi chain; her position object. The probe checks all three before any run.
- **Polling:** `hitlog_pc` logged about 60 records a second with a stability check on 6 Nov. A missed frame only
  thins the data.
- **Your HP:** `nodeath` (or the arena's Can't die) keeps the session going. Damage numbers still show as HP drops,
  which is how hits are counted.
- **Writes:** only `nodeath`, and `attackswap` if agreed. Both are offline, logged in `saves\pcmem_writes.log`, and
  undone by restarting the game.
- **Lud & Zallen** aren't covered: same animations, but they run to a different set of spots and don't grab.
