# Sir Alonne's moves: the range slash, the parry counter and every attack recorded (PC Scholar 1.0.3.0, live 24 Nov 2026)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

**Questions.**
1. Attacks 5, 6 and 7 are on his AI's list but seem to have no animation of their own. What are they?
2. His pause-slide-slash varies in start time, speed and distance. Is it one move stretched to fit, like
   [Aava's leap](aava-leap-distance.md), or several?
3. Turning and running away when he winds up that move always works. Why, and when does he stop tracking you?
4. His "parry": he holds his sword out, and if you attack he knocks you down and stabs you. Forced with a practice
   tool, he knocks you down with no stance. What is it?
5. Attack 13 looks like a slower copy of a move he uses often, but no render shows the other one. Which is it?

**Answers.**
1. **The delayed copies of the range slash.** Attacks 5, 6 and 7 play exactly the same animations as 14, 18 and 19,
   with the same hitbox settings and damage. Only the wind-up is slower: 70 % speed against 110 %.
2. **Both.** Six attack numbers play one move. His AI picks one of three animations by your distance (close 14,
   middle 18, far 19). Each animation blends a short and a long slide by your distance, set up the same way as
   Aava's leap. On top of that, about one time in three he uses the delayed copy (5, 6, 7).
3. **He commits to your distance the instant he launches forward.** Up to then he re-reads it; after that the slide
   is fixed, and the close and middle versions have a longest slide they can't exceed. In 12 run-aways with no roll,
   the gap while the slash could hit was 3.6-8.4 m and he never hit; his slash reaches about 2.5 m. He stops turning
   about 0.2 s into the slash.
4. **A grab thrown from a guard stance.** The counter (attack 17) is set up as a grab, the same way as his dashing
   lunge (16). The stance is a separate part of his AI: he takes it now and then, and throws the counter only if you
   act within 3 m in front of him. A tool that swaps his attack ids skips the stance, so a forced 17 grabs at once.
5. **The slash that opens his sideways combos (54 and 56).** It is the same animation bone for bone, used on its own
   with a slightly slower wind-up. The renders only show it inside the combo clips.

**How this was found.** Two recorded fights in the Boss Arena on 24 Nov 2026: 23 minutes of ordinary fighting, then
a shorter run that forced the four attacks the first never showed (4, 13, 17, 19). A recorder logged each attack his
AI chose, when his hitboxes came out, the player's distance and angle from him, both health bars and the player's
rolls, on the same clock as the video. That was read alongside his AI script, animation network, time-acts and
params. Each claim below is tagged **(live)** when the fights showed it and **(files)** when it rests on the game
files. The logs, videos and clips are kept with the source notes, not on this site.

## 1. The range slash: six attack numbers, one move

He pauses, slides forward and slashes upward. He doesn't use it when you're right next to him. The further away you
are, the later the slash comes and the further he slides, and during the pause he keeps adjusting to how far away you
are. About one time in three he holds the pause for roughly half a second longer.

| How far away you are | Usual version | Delayed version (about 1 in 3) |
|---|---|---|
| Close (about 4-11 m) | attack 14, hits about 0.95 s after he starts | attack 5, about 1.4 s |
| Middle (about 11-15 m) | attack 18, about 1.05 s | attack 6, about 1.6 s |
| Far (about 15-25 m) | attack 19, about 1.25 s | attack 7, about 1.85 s |

Times run from his choice of attack to the moment his hit can land **(live)**: medians over 15 / 5 / 4 / 5 / 3 picks.
The far usual version was only recorded forced at close range (1.26-1.38 s, 3 picks), and the files predict 1.23 s.

- **One set of animations, two speeds (files).** In his network (c6800), each of AttackCategory 5, 6 and 7 has its own
  state machine, but its distance blend and play-speed nodes are identical byte for byte to those of 14, 18 and 19. They play `a65_14_1010`,
  `a65_18_1010` and `a65_19_1010` and their `_L` partners. In the behaviour params, each pair differs only in the
  damage id (1680011x0 against 1680006x0, rows identical apart from the id) and in `attack_speed`: 0.7 against 1.1.
  For 7 against 19, `recovery_speed` also differs (1.0 against 1.4). The game plays a clip at `attack_speed` while
  morpheme event 150 runs, which here is the wind-up (the first 0.83 s of clip time).
- **How he picks a version (files, live).** His AI script (`ai680000`, the group that requests these six ids) does
  nothing under 4 m; 4-11 m gives 14, 11-15 m gives 18, beyond that (up to 25 m) gives 19. Then a roll of 0-100
  under 35 swaps in the delayed copy. In the 23-minute fight every pick fell inside its band (14 at 5.8-10.8 m, 5 at
  7.0-10.8, 18 at 11.6-12.6, 6 at 11.1-13.9, 7 at 15.8-19.4), and the delayed copy came 13 times in 32 (41 %).
- **How far he slides (files).** Each animation is a Blend2 of a short and a long clip on `Target_Distance`: fully
  short at 7 / 10 / 16 m, fully long at 12 / 18 / 30 m (close / middle / far). The `TD` event track (event 125,
  re-reading `Target_Distance`) covers the same 0.83 s as the wind-up, as on Aava's leap. Alonne's animation exports
  carry no forward movement, so the slide's length in metres wasn't measured from the files; the fights give the gap
  instead.

### Turning and running

**When he commits (live).** In all 32 picks he started moving forward within 0.07 s of the end of the wind-up, the
moment the distance stops being re-read (the recorder samples the distance about every 0.05 s). From that launch on,
the usual and delayed copies are identical:

| Attack | Distance fixed | Starts moving (median) | Slash from | Launch to slash |
|---|---|---|---|---|
| 14 | 0.76 s | 0.75 s | 0.96 s | 0.20 s |
| 5 | 1.19 s | 1.18 s | 1.39 s | 0.20 s |
| 18 | 0.76 s | 0.76 s | 1.02 s | 0.26 s |
| 6 | 1.19 s | 1.23 s | 1.46 s | 0.27 s |
| 7 | 1.19 s | 1.23 s | 1.82 s | 0.63 s |
| 19 | 0.76 s | not seen | 1.23 s (files) | 0.47 s |

**What running did (live).** Every range slash in the fights, by what the player did during the wind-up:

| During the wind-up | Times | Gap while the slash could hit | Hit |
|---|---|---|---|
| ran, no roll | 12 | 3.6-8.4 m | never |
| ran, then rolled | 2 | 1.8-3.4 m | never |
| held or came in, rolled | 15 | 1.1-2.4 m (one at 3.4) | 4 times |
| held, no roll | 3 | 3.6-5.2 m | never |

All four hits came with the player 1.1-2.4 m from him, so the slash reaches about 2.5 m. Running put the player
1.5-5.5 m further away by the time he launched, and he never made that up.

- **The delay doesn't matter if you run until you see him launch.** The delayed copy only gives you longer to run
  before he commits. The delayed close and middle versions missed by more (6.1-8.4 m) than the usual ones (3.6-5.8 m).
- **The way to get caught is to stop too soon**, for example after the usual wind-up when it's a delayed one. He then
  commits to where you stopped.
- **The far version adapts the most.** Its slide keeps growing up to 30 m, so running gives him a longer slide
  rather than taking him to his limit. Its two run-aways still fell 4.0-4.6 m short.
- **Rolling has to be timed per version.** From the wind-up, the slash comes between about 0.95 s and 1.85 s. From
  the launch it is about 0.2 s for the close version, close to a reaction time, so a roll has to anticipate, and that
  is what the delayed copies catch.

**Turning (files; read as degrees per second, which the fights agree with).** Time-act event 280100 for the close
usual version, in real time:

| From the start | What he's doing | Turning |
|---|---|---|
| 0-0.36 s | wind-up | 180 |
| 0.36-0.79 s | late wind-up | 60, then 30 |
| 0.79-0.96 s | launch | a snap, 270 |
| 0.96-1.19 s | slash | 180, then 45 |
| after 1.19 s | recovery | 0 |

The delayed copies follow the same pattern 0.43 s later; 18 and 19 snap at 300. A sidestep late in the wind-up is
caught by the snap; about 0.2 s into the slash he stops turning, so he stays facing where he slashed. Live, the player
stayed within about 15 degrees of his front through every wind-up, and after a roll past him the angle opened to
60-120 degrees within a second without him following.

## 2. The parry counter is a grab

Now and then he holds his sword out in a guard stance for a few seconds. If you act while you're close and in front
of him, he answers with a counter that works as a grab: if it connects he knocks you down and stabs you. The catch
itself does no damage.

- **A grab in the game's terms (files).** The counter swing's damage row, EnemyDamageParam 168000910, has
  `grab_param_id` set (168000910). Among his attacks only the dashing lunge (16, row 168000810) is set up the same
  way; every other hit has -1. The swing does 0 damage (`damage_0` 0, `damage_mult` 0); the damage (876 base) is in
  the follow-up row 168000920, which uses the same DamageCtrlParam row (590030200) as the lunge's follow-up. In the
  network, AttackCategory 17 plays `a65_17_1010` and switches to `a65_17_1020` on the morpheme message
  `CatchAttack`. It doesn't use the player-style parry. For how grabs land against rolls, see
  [grab timing](grab-timing-pc.md).
- **The stance is its own part of his AI (files).**
  - **Taking it:** when you're 3-7.5 m away and a further check passes (`f7023() < 60`, meaning unknown; not his
    health, which was 95-96 % both times), a roll gives the stance 15 %.
  - **Holding it:** a random 4-6 s, unless something ends it sooner.
  - **The counter:** requested only after at least 1 s in the stance, with you within 3 m and within 45 degrees of
    his front, and two checks on you passing (most likely "attacking"; not confirmed).
- **Forcing skips the stance (live).** A tool that swaps the attack id he asks for turns every attack of his into 17,
  so the counter comes straight away: 0.18 s from his choice to the catch (11 picks), with no stance and no attack
  from you.
- **In the fights (live).** He never took the stance in the 23-minute fight. In the second run he took it twice and
  threw the counter both times; neither caught the player.

## 3. Attack 13 is the opener of combos 54 and 56

- **The same animation (files).** AttackCategory 13 plays `a65_13_1010`. Its bone rotations match `a65_15_1010`
  exactly (0.0 degrees mean difference over the clip), the first part of action 15, which opens combos 54 and 56
  ([the combo table](unused-attacks-pc.md)). The renders show it only as the start of those combo clips.
- **Timing (files, live).** The wind-up runs at 0.65 for 13 and 0.8 in the combos. Live, 13's hit came 0.88 s after
  his choice (9 forced picks) against 0.84 s for 56 (22) and 54 (16).
- **How often (live).** He never chose 13 on his own in the 23-minute fight, while 56 and 54 were among his most
  frequent attacks.
- **Open.** In an earlier practice session with no recorder running, a much quicker version was seen fairly often.
  Nothing recorded matches it. The state machines of 13 and 15 both hold a dash clip (`a65_13_1011`) that goes into
  the slash on the morpheme message `Combination`; a dash into the slash would show no wind-up.

## 4. The kick

Attack 4 is a kick **(live)**. Its damage row (168000710) puts a sphere on dummy point 5, where the other rows
compared (610, 810, 910, 1110, 1210) put a capsule on the sword's dummy point 100 **(files)**. He never chose it on his own in the 23-minute fight.

## 5. How often he used each move (live)

From the 23-minute fight, 179 attacks. One player and one fight, so a rough guide only: how often he uses each move
depends on how far away you keep. Names are working names.

| Attack | Working name | Times |
|---|---|---|
| 0 | Crouch, high jump, sideways landing (no hit) | 38 |
| 14, 18, 19 / 5, 6, 7 | The range slash, usual / delayed | 32 (19 / 13) |
| 56 | Sideways slash, then hop away | 22 |
| 52 | Overhead cuts then forward thrust | 16 |
| 54 | Sideways slashes and rising cut | 16 |
| 9 | Hop away and wide low lunge | 14 |
| 16 | Dashing low lunge (a grab) | 14 |
| 50, 51, 55 | Overhead cut then low lunge (three combo variants) | 19 (8 / 3 / 8) |
| 53 | Leaping cut then crouching slashes | 6 |
| 20 | Sword cut and swing | 2 |
| 4, 13, 17 | Kick, lone sideways slash, parry counter | 0 |

His AI can choose 20 attack numbers. A 21st, 15, only plays inside combos 54 and 56. With the forced run, every one
of the 20 was recorded, and a clip of each was cut from the videos.

## Not checked

- The much quicker version of 13 seen in practice.
- Whether the guard stance also blocks your attack before the counter.
- What `f7023() < 60` checks before he takes the stance.
- Why the delayed middle slash (6) lands up to 0.1 s later than the files predict at 11-12 m.
- How far each slide goes in metres.
- The original release and the console versions.

## Related pages

- [Aava's leap](aava-leap-distance.md): the same `Target_Distance` blend and event 125.
- [Parries and ripostes](parry-and-riposte-pc.md): his one parryable hit (the leaping slam, attack 12) and why he
  can't be riposted.
- [Grab timing](grab-timing-pc.md): grabs against rolls.
- [Unused enemy attacks](unused-attacks-pc.md): which combo plays which action, his included.

## Data reference

- AI script `ai680000`: the six range-slash ids are requested only in group `g2147483627` (`1:5204(5 / 6 / 7 / 14 /
  18 / 19, 0)`), called as `(65, 4, 1, 25)`; the stance is group `g2147483634` (`1:5206(1, 1)` on entry, the counter
  is `1:5204(17, 0)`), entered from `g2147483643` (3-7.5 m branch, roll 40-54 of 100); the dump hides which group a
  call goes to, so the stance's numbers assume the call `(6, 4, 7, 1, 3, 3, 1, 45)` from that branch.
- Network c6800: AttackCategory 5 / 6 / 7 -> state machines 532 / 544 / 556 (Blend2 542 / 553 / 565); 14 / 18 / 19 ->
  690 / 773 / 785 (Blend2 700 / 782 / 794); 17 -> 756; 13 -> 672; 15 -> 702.
- Behaviour row 680000: 5-7 = EnemyBehaviorParam groups 6-8; 13, 14, 17, 18, 19 = EnemyBehaviorSecondParam groups 4,
  5, 8, 9, 10. All six range-slash groups use EnemyAttackMoveParam 1010.
- Time-acts `c680000_pl`: 65141010 / 65181010 / 65191010, hit windows (event 2200) 1.033-1.233 / 1.100-1.300 /
  1.467-1.667 s clip time.
