# Contact margin: why a hit needs a few cm of overlap (PC Scholar 1.0.3.0, live 18 Nov 2026)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

**Question.** In the [grab timing test](grab-timing-pc.md), a grab caught on the first physics frame its hand
overlapped the player only when that overlap was more than about 3-5 cm. Shallower first contacts were caught one frame
later. Is that our measurement (hit-model sizes, or when positions were read), or the game?

**Answer: the game.** More exactly, it's Havok, the physics engine, with the settings the game uses. The game's own
hit code never tests depth. Havok reports a new contact to the attack hitbox in one of two ways:

- **Same frame:** if the overlap gets deep enough during the physics step, Havok raises a time-of-impact (TOI) event
  and reports the contact at once.
- **Next frame:** otherwise it is an ordinary contact, which Havok only reports at the start of the next physics step.

So the "margin" is a timing cut-off, not a depth a hit needs before it counts. A shallow contact still counts, one frame
later.

Checked three ways, all on 18 Nov:
- **Code:** static reading of the game.
- **Old logs:** the 17 catches from the grab test.
- **Live:** three runs reading Havok's own contact data from the running game next to our geometry.

## The rule

A contact is reported on the first frame of overlap when

**overlap depth ≥ max(½ P, ¼ P + v × 2.5 ms)**

and otherwise on the next frame (if the overlap is more than about 1.4 mm).

- **P = allowed penetration.** It is the smaller value of the two bodies. Havok sets it by default to 0.2 × the shape's
  smallest box size (0.1 m if that size is 0.5 m or more), and the game never changes it.
- **v = the closing speed** of the two bodies, in m/s.
- **2.5 ms = Havok's minimum TOI time.** A TOI that would fall in the last 2.5 ms of the step is dropped.
- **Havok's reported distance:** on a same-frame contact, Havok's own contact distance is the moment of the TOI,
  −¼ P, not the end-of-frame depth.

The player's 11-part hit model (read live; parts named by their size and shape), against the Ogre's 1 m grab spheres
(allowed penetration 0.1 m):

| hit-model part | shape | allowed penetration | P against the grab | cut-off at least (½ P) | Havok's distance on a same-frame contact (−¼ P) |
|---|---|---|---|---|---|
| 0, head | sphere r 0.125 m | 0.05 m | 0.05 m | 2.5 cm | −1.25 cm |
| 1-4, arms | capsule r 0.07 m | 0.028 m | 0.028 m | 1.4 cm | −0.70 cm |
| 5, chest | capsule r 0.2 m | 0.08 m | 0.08 m | 4.0 cm | −2.00 cm |
| 6, waist | sphere r 0.2 m | 0.08 m | 0.08 m | 4.0 cm | −2.00 cm |
| 7-10, legs | capsule r 0.1 m | 0.04 m | 0.04 m | 2.0 cm | −1.00 cm |

The Ogre's hands close at 10-15 m/s, so the head's cut-off is 3.7-4.9 cm. The hands travel about 20 cm per frame, so a
head-first contact lands below the cut-off roughly 1 time in 4 or 5, depending on where the frame boundary falls. Each
part has its own cut-off, and the first part to cross its own decides a same-frame catch.

## 1. Code (DarkSoulsII.exe 1.0.3.0, static)

- **The game's contact listener doesn't test depth.**
  - The attack hitbox (`PXBulletRigidBody`) has a Havok contact listener at object +0xF0.
  - Its contact-point callback (0x140BDB360) copies the contact's position and normal into a per-hitbox list
    (+0xD0, 0x50-byte entries). The normal's w is the contact distance, and it is never compared.
  - The hit manager (0x1403B1870) turns list entries into hits. Its float compares are dt, timers, an epsilon and an
    angle, never the distance.
  - The callback doesn't look at the contact's event type either. So "did Havok call the listener this step" is the
    whole decision.
- **World settings** (0x140BC0A70):
  - collision tolerance 0.05 (Havok's default is 0.1);
  - minimum time step 1/60;
  - contact point generation left at Havok's default, so a new contact needs about −1.36 mm (0.5 × g × dt²,
    penetrating).
  - Neither the tolerance nor the 1/60 step is what decides the cut-off here.
- **Allowed penetration:**
  - The game sets none for the hit model or the attack hitboxes, so every body gets Havok's automatic value
    (0x1406FD960: 0.2 × the smallest box size, or 0.1).
  - The hkpRigidBody constructor only overrides it for fixed bodies.
  - Each collision agent uses the smaller of the two values.
- **The collision agent:** an attack hitbox against the hit model is a TOI-quality pair. Even sphere against sphere goes
  through the predictive GSK agent (0x140DD0A90), not the plain sphere-sphere agent, which is not registered for
  continuous collision.
- **The TOI routine** (0x140E206D0):
  - The end-of-step overlap must reach ½ P.
  - The TOI time is when the distance crosses −¼ P.
  - A TOI in the last minToiDeltaTime of the step is dropped (0x140E212EB). minToiDeltaTime is 2.5 ms, absolute
    time, from Havok's default quality settings.
- **Delivery:**
  - A TOI's contact reaches the listener in the same step.
  - An ordinary new contact reaches it only in the next step's solver setup.
  - An independent second reading confirmed every claim above, including all 10 places the listener is called from.

## 2. The 17 catches from the grab test (logs)

With nothing tuned, the rule predicts all 17: caught on the first overlap frame or one frame later. No case is within
0.3 cm of its cut-off. The closest:

| grab (spawn frame) | first-frame overlap (head) | closing speed | cut-off | caught |
|---|---|---|---|---|
| 77886 | 5.40 cm | 12.5 m/s | 4.35 cm | that frame |
| 95085 | 5.33 cm | 9.9 m/s | 3.72 cm | that frame |
| 14647 | 2.72 cm | 12.8 m/s | 4.46 cm | next frame |
| 23486 | 0.60 cm | 5.1 m/s | 2.54 cm | next frame |

Two simpler rules each get 16 of 17, and each misses by about 2 mm:
- a fixed 2.5 cm misses 14647;
- P = 0.1 for every part misses 77886.

Our own measurements needed no correction:
- the hand sphere sits exactly at its keyframe target after each step (within 0.1 mm);
- no whole-frame re-pairing removes the margin;
- the head is a plain sphere with no offset.

For our measurements to be the cause, every depth would have to be overstated by 2.7-5.3 cm.

## 3. Live: Havok's own contact data (3 runs, 24 grabs)

An outside reader logged, every frame, each attack hitbox's contact list (+0xD0: the other body at entry +0x30, the
contact distance at +0x2C) next to our geometry. Steam was offline, and the reader only read memory.

- **Settings, read live:**
  - allowed penetration (hkpRigidBody +0x88) exactly as in the table above;
  - Havok's TOI quality settings −0.5 / −0.25 / 0.0025 s, as the code says.
- **Same-frame contacts report −¼ P, not the depth:**
  - All 30 head first contacts above their cut-off (overlap 5.5-44.6 cm) got a point that same frame at about
    −1.25 cm, within Havok's TOI accuracy (about ±0.3 cm).
  - The other parts reported their own −¼ P: −0.70, −1.00, −2.00 cm.
  - A depth check in the game would have seen the full 5-45 cm.
- **Shallow contacts wait a frame:**
  - 7 of the 8 head first contacts below their cut-off (0.4-2.5 cm) had no contact point on that frame. Havok had
    noted the pair, but no point arrived.
  - The point arrived on the next frame as an ordinary contact, with a distance other than −1.25 cm.
  - The eighth is the suspect frame below.
- **Every first overlap tested:** 427 of 434 sphere-and-part first overlaps fit the rule, including 138 of 139 that
  were a hand's first contact.
  - **The one first-contact miss** (a 1.3 cm head overlap that got a point the same frame) is on a frame where the
    reader's two quick reads disagreed, so our depth there is suspect.
  - **The other 6 misses:**
    - 4 sit within 0.26 cm of the cut-off;
    - 1 is on another frame where the reads disagreed (a leg at 2.5 cm, 2 cm under its cut-off);
    - 1 is a deep leg overlap whose point came in at −0.04 cm.
    - All 6 are on parts touched after that hand's first contact, which Havok's handling of the earlier TOI can
      affect.
- **Whole grabs caught a frame late:** 3 seen live.
  - In each, the hands only grazed the head on the first frame (1.0-2.5 cm), with no contact point and no catch.
  - The catch came on the next frame.

## What it means in play

- **The basic lag:** attack hitboxes are moved to their drawn position over the next physics step. That puts a real
  touch one frame after the drawn one (see [phantom hitboxes](phantom-hitboxes-pc.md)).
- **One more frame for grazes:** a contact shallower than its cut-off lands one frame later still. Deep contacts land
  on the first frame of overlap.
- **The cut-off varies with:**
  - **the body part:** thin parts have smaller cut-offs;
  - **the attack's speed:** faster attacks have larger cut-offs in cm, but also travel further per frame.
- **Grabs:** which frame a grab catches on now follows exactly. Whether it catches at all still follows the i-frame
  rules on the [grab timing](grab-timing-pc.md) page.

## Not covered

- **Can a late contact catch on its own?** In all 18 live catches, at least one part also had a same-frame contact
  that frame. At the Ogre's speed, the frame after a graze always drives some part past its cut-off.
  - The code treats both kinds of contact the same, so a late one should count by itself.
  - Showing it live needs a slow hitbox that stops just after a shallow graze.
- **Normal hits:** only grabs were tested here. Normal attack hitboxes are the same kind of body, so the rule should
  apply. It may also explain the 6 Nov hits that landed a frame later than expected, but those logs used the wrong
  hurtbox (the ragdoll) and weren't re-checked.
- **Other enemies:** other sizes and speeds were not tested.
- **Offline only.**

## Method note

- **Havok class layouts** are Havok 2014.1, as published by LordRadai. The game-specific offsets above were found
  for this page and checked live.
- **Live logger:** a read-only extension of the grab-test logger. It adds the contact lists, the allowed penetration
  values and the TOI quality settings (local, not on this site).
- **The per-frame "two quick reads":** each frame is read twice, 1.5 ms apart. A frame where the two disagree is
  marked unstable and treated with care.
