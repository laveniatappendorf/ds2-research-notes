# Leading edge of roll i-frames: test plan (2026-11-07, not run)

!!! warning "Status: Plan, not run"
    A test or idea written down. Nothing here has been tried. See [status labels](../status-labels.md).

**Question.** DS2's roll i-frames are said to "come out really early" compared with DS1 and DS3. The calculation
itself is ordinary (`live-pc.md` section 3), so where could extra early protection come from, and how many frames is it?

Builds on `phantom-hitboxes-pc.md` (live test of 6 Nov, sections 3-5). PC Scholar 1.0.3.0, same tools and logs.

## Candidates

| | what | expected size | status |
|---|---|---|---|
| L1 lag | Damage on frame F is judged on positions drawn at F-1, but with the counter of frame F. So the first counter frame covers a contact drawn on the frame **before** the roll is invincible. | 1 frame every roll (16.8 ms at 60 fps; 33 ms on PS3 at 30 fps) | follows from rules 3 + 4, no clean example yet |
| L2 late contact | 3 of 12 clean hits landed a frame **after** the physics touch. If that happens at a roll's start, the first counter frame also blocks a touch that physics found on the frame before. | a 2nd frame, some of the time (~1 in 4 if the rate holds) | open; may be the same mechanism as the end-of-roll carry-over |
| L3 blend | The roll's i-frames run from its first TimeAct update (event 110100 starts at -0.167 s). The model may still be blending out of the previous pose, so it looks like it hasn't rolled yet. | visual only, 0-3 frames? | not measured |
| L4 input | Frames from the button to the counter. Only matters for feel. | ? | not measured |

## Terms

- **D** = first frame the drawn hitbox touches the hurtbox.
- **P** = first physics touch (= D + 1 for a hitbox alive 2+ frames; a hitbox's own spawn frame doesn't count, section 5).
- **s** = first frame with the i-frame counter (+0x5F0) on.
- **k = s - P**. Expected outcomes:

| k | meaning | L1 only | L1 + L2 |
|---|---|---|---|
| -1 | counter on a frame before the touch | blocked | blocked |
| **0** | drawn touch on the frame before the counter (**the visual leading frame**) | blocked | blocked |
| **+1** | physics touch on the frame before the counter (**a real leading frame**) | hit at P | usually hit at P, sometimes blocked |
| +2 | counter well after the touch | hit at P | hit at P (or P+1) |

## Prep (no game needed)

1. **T0, existing logs.**
   - (a) Spawn-frame rule: every clean hit whose hitbox appeared already overlapping should drop HP on spawn + 1, never
     on the spawn frame.
   - (b) The 3 late hits (31617, 31750, 72761): how deep was the physics touch at P? If they're all shallow (a few cm),
     our capsule reconstruction is slightly off and L2 is probably not real. If they're deep, it's timing.
2. **`hitlog_pc.py iforce <s> <out> <offset> [hold]`** (forced counter while standing still; isolates the timing rule
   from the roll's animation).
   - Logs every frame like `log`, and works out drawn overlaps live.
   - When an enemy hitbox (alive 2+ frames; not a map hazard, not dmg 9000) first shows a drawn touch at D, it adds 1 to
     +0x5F0 right after reading frame D + offset. The counter is then on from D + 1 + offset, so k = offset.
   - k = -1 needs a forecast: write when the drawn gap is under one frame's closing distance. The log gives the
     actual k anyway.
   - Holds for `hold` frames (default 30), then subtracts 1 (read-modify-write, so a real roll's counts are kept).
   - Re-reads the frame counter after each write; a write that straddles a frame tick is flagged and not counted.
   - Writes go through pcmem (offline check + write log); restores on exit and stops on its own time limit. (The 6 Nov
     hard stop of `nodeath` skipped its clean-up.)
   - The reader of +0x5F0 in the damage code isn't traced, and +0x608 moves with it. If +0x5F0 alone doesn't protect
     in a first standing check, write both.
3. **`hitlog_analyse.py starts <log>`**. For every i-frame window, natural or forced, and every enemy hitbox that
   touches near s, list:
   - k;
   - depth at P;
   - the HP drop frame;
   - whether the hitbox is on its spawn frame;
   - then a table by k.
4. **For T3:**
   - Player animation id + TimeAct time per frame. PC offsets aren't known: read where `FUN_140326240` gets its event
     time t (headless Ghidra; the Ghidra MCP didn't connect on 7 Nov).
   - The pad, if it can be seen from outside the game: XInput polled by the logger works for an Xbox-type pad. Steam
     Input's virtual pad may only exist inside the game. Keyboard via GetAsyncKeyState always works.
5. **Death guard:** `nodeath topup` tops HP up to +0x174 (797). The game clamps that back to 598, which shows up as
   fake HP drops. Fix it to the real maximum first, or run with min HP 1 only.

## Live tests (Steam offline)

- **T1, forced counter, standing still, large knight** (slow, wide capsules that stay out for many frames):
  - one run each at offset 0, 1 and 2 (k = 0, +1, +2), plus one forecast run (k = -1);
  - aim for 10+ touches per k: about 4 x 60-90 s;
  - just stand there; no blocking, rolling or attacking.
- **T2, natural late rolls:** 3 x 60 s against the knight, rolling as late as possible (start the roll as the blade
  arrives).
  - The 46 rolls of 6 Nov were all early, so none fell at k = 0 or +1. Late rolls should land one or the other in
    roughly 1 in 10 rolls each.
  - A hit just before the roll's super armour may stagger and cancel the roll, so no window gets logged. Count those
    from the input or animation log.
- **T3, how early it looks:** 20 rolls from standing and 20 from running, no enemy, logging animation id / time,
  root and hurtbox. Per roll:
  - frames from the roll's start (the animation switch) to the counter;
  - how far each hurtbox capsule has moved from the pre-roll pose on each of the first i-frames;
  - frames from the button to the counter, if the pad is visible.

## Reading the results

- **k = 0 blocked, k = +1 always hit** → L1 only: one visual leading frame, no real one.
- **k = +1 sometimes blocked** → L2 is real: sometimes a second leading frame. Then check that the late fraction
  matches the clean-hit late rate and the end-of-roll carry-over (likely one "contact handled a frame late"
  mechanism).
- **T3: counter on for several frames before the hurtbox visibly leaves the standing pose** → L3 adds to the early
  feel, on top of the lag.

## Not covered

- **DS1 / DS3.** "Earlier than DS1 and DS3" needs the same two numbers from those games: whether their hit check
  lags a frame, and their roll i-frame start. Our tools are DS2-only.
- **PS3 (vanilla / Scholar).** The same rule is expected, but at 30 fps (33 ms a frame). The overlay can't do this
  test yet.

Effort: prep 1-5 about one session; live T1-T3 about 30 minutes of play.
