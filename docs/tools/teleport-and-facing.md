# Teleport and facing: writing the player's position and yaw (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

This is the second half of the [Memory map for PC](memory-map-pc.md); read its "Notation" first. `GameManagerImp` is the object
`[exe + 0x16148F0]`, `[x + 0x70]` is the pointer stored at `x + 0x70`, and `PlayerCtrl = [GameManagerImp + 0xD0]`.
Tags in square brackets: `[live A7]` = seen in the game (ids point to the project's test log, see [Home](../how-to-read.md#the-modding-pages)), `[code]` =
read in the exe, `[inference]` = reasoned, `[untested]` = an idea only.

## Summary

- **The recipe.** Walk to the player's Havok body and write the same x, y, z to `+0x1A0`, `+0x1B0` and `+0x1C0`; write a
  velocity of (0, -2, 0) to `+0x230`; write the facing quaternion `(0, sin(yaw/2), 0, cos(yaw/2))` to `+0x30` and `+0x50` of the
  `MorphemeChrCtrl`. The player lands exactly on the spot, standing, and the map draws by itself. [live A7]
- **The kick is not optional.** A bare position write leaves the map undrawn until the player takes a step. [live A7]
- **Facing has one master.** `MorphemeChrCtrl +0x30`. The world matrix in `PlayerCtrl` and the motion controller's forward
  vector follow it within a frame. [live A7]
- **It is a teleport inside the current map.** The position is in that map's own frame. Every test warped to the map first.
  Landing inside a trigger region runs the region's script, and crossing the map by teleport **skips** the regions in between
  [live 30 Oct, 4 Nov].
- **The traps.** A guessed spot can be inside a wall or the exit fog and kill the player at once; raising the target by 0.5 m
  makes the body slide 3 m; raising it by 0.1 m leaves it floating and the map undrawn. [live A7, 28 Oct]

### Where the position lives

Three objects hold it. The first is the master; the others follow. [live A7]

| Where | Offsets | What it is |
|---|---|---|
| `[[[PlayerCtrl + 0x100] + 0x320] + 0x20]`, the `hkpRigidBody` | **+0x1A0, +0x1B0, +0x1C0**: three f32 each (x, y, z) | the physics position: the body's transform and the swept centre of mass, 0 and 1 |
| the same body | +0x220: (0, 0, 0, 0.01) | inverse inertia and inverse mass |
| the same body | **+0x230**: f32 x3 | the linear velocity |
| `[[PlayerCtrl + 0xF8] + 0xF0]`, the `MorphemeChrCtrl` | +0x20: (x, y, z); copy at +0x40 | the position the game shows; its y is 0.05 below the physics body's |
| the same `MorphemeChrCtrl` | **+0x30**: 4 x f32; copy at **+0x50** | the facing quaternion |
| `PlayerCtrl +0x60 .. +0x9F` | floats | the world matrix: x axis (cos yaw, 0, -sin yaw), y axis, z axis (sin yaw, 0, cos yaw) at +0x80, and the position |
| `ChrMotionCtrl` (`[PlayerCtrl + 0xF8]`) | +0x50 position, +0x60 forward (sin yaw, 0, cos yaw) | follows the quaternion |

The classes along the chain read `ChrPhysicsCtrl` (`[PlayerCtrl + 0x100]`), `hkpCharacterRigidBody` (`[+0x320]`) and
`hkpRigidBody` (`[+0x20]`). [live A7] The community cheat table's "Teleport" entries write the same three position copies.
The body's own rotation stays the identity; the yaw is not stored there.

**The three position copies are copies.** At every teleport the tools' write log shows the same old 12 bytes at `+0x1A0`,
`+0x1B0` and `+0x1C0`. We always wrote all three and never tried one alone [live A7 write log; untested]. The velocity's old value
was twelve zero bytes. The velocity was confirmed on another body that took over the sampled address after a warp: a settling
drop, then an idle wobble of about 0.6 m/s either way. [live A7]

### The recipe

Our tool does these writes in this order (`teleport.py go`, `arenalib.teleport`):

```
pc    = u64(GameManagerImp + 0xD0)                      # class PlayerCtrl, else stop
body  = u64(u64(u64(pc + 0x100) + 0x320) + 0x20)        # the hkpRigidBody; stop on a null link
morph = u64(u64(pc + 0xF8) + 0xF0)                      # the MorphemeChrCtrl
for off in (0x1A0, 0x1B0, 0x1C0):  write(body + off, pack("<3f", x, y, z))
write(body + 0x230, pack("<3f", 0, -2, 0))              # the kick: 2 m/s downwards
q = pack("<4f", 0, sin(yaw / 2), 0, cos(yaw / 2))       # yaw in radians
for off in (0x30, 0x50):           write(morph + off, q)
```

Yaw in degrees is `2 * atan2(q.y, q.w)`. A sample: the tester faced the fog, the tool read `(0, 0.09288, 0, 0.99568)`, which is
10.7 degrees. The warp had left the player at 90 degrees, `(0, 0.7071, 0, 0.7071)`. [live A7 write log] The same formula gives
the forward vector `(sin yaw, 0, cos yaw)` the motion controller keeps, so yaw 0 faces +z and yaw 90 faces +x. It matches the
`(sin yaw, cos yaw)` forward used for bonfire parts in [Moving a bonfire](../modding-world/moving-a-bonfire.md) [inference: same convention; not
cross-tested].

**What it looked like.** Landed exactly on the spot, steady, the map drew by itself (a brief transition, noticeable but fine for
a trainer) [live A7, 17:04]. Facing: the matrix and the motion forward vector followed the quaternion to 10.7 degrees at once and
held for the 1.8 s we read them; the tester saw no snap and the map drew as before [live A7, 17:10-17:15].

### Why each step is there

The tries, in order. Each is a separate teleport after a bonfire warp to the Forest, to a spot a few paces before the Last Giant's
fog. [live A7, 28 Oct]

| Try | Time | Result |
|---|---|---|
| Position only | 16:47 | arrived **standing**, height steady over 4.5 s, no fall, no void-out, no odd animation. **The map did not draw** until the tester took a step. The warp's facing was kept (the facing had not been written) |
| Position 0.5 m up | 16:59 | the map drew by itself, but the body landed **3.2 m east and 0.33 m higher**, facing a wall, and settled in 0.7 s |
| Position 0.1 m up | 17:02 | **no fall at all**: the body held 0.1 m above the floor for 4 s or more, inside the character controller's ground tolerance, and the map did not draw |
| Position, then velocity (0, -2, 0) | 17:04 | **exact landing, steady, the map drew** |
| Same, then the quaternion | 17:14 | the tester faced the fog, no snap |

Reading it [inference]: the drawn map pieces follow the collision the player stands on, and that only updates when the body moves;
an earlier project of ours found the same with a draw-whole-map change. A bare write puts the body on a spot where it already
stands, so nothing updates. A big lift makes it fall, but the fall slides it, and a small lift stays inside the ground tolerance and
does not fall. A downward velocity makes the body move without changing where it lands. 2 m/s was the first value we tried and the
only one [untested: other values].

### Facing: how the master copy was found

The facing was an open question after the position worked. `teleport.py face` stores the quaternion and then scans three objects
for floats that equal the yaw in radians, its sine, its cosine, `q.y` or `q.w` (within 0.002): `PlayerCtrl` (0x2000 bytes),
`ChrMotionCtrl` (0x400) and `MorphemeChrCtrl` (0x200). That listed the candidates: the matrix at `PlayerCtrl +0x60`, the forward
vector at `ChrMotionCtrl +0x60`, and the quaternion. Writing only the quaternion made the other two follow, so the quaternion is
the master and the others are derived. [live A7] The same trick finds the owner of any value that a game copies around: write one
suspect, read the others.

### Choosing a spot

- **A spot is saved, not computed, when it matters.** `teleport.py save <name>` stores the physics position (+0x1A0), `face <name>`
  stores the quaternion; both live in one JSON file. The trainer's Home key does the same for the boss you picked. [live A7]
- **From the game's data**, a live position is an event location's local position plus the map's offset (three floats in the map's
  MSB); see [Warping by script](../modding-world/warping-by-script.md).
- **Computed fog spots were wrong for 5 of 7 bosses.** The fog nearest a boss is often the exit, and one fog object sat 4 m
  under the floor. The community table's boss teleports were on the entrance side 5 times out of 5 [live 28 Oct]. A guess of
  55 m for Aava put the player in a wall and killed them; the trainer now limits a computed guess to 35 m and skips an automatic
  teleport within 8 s of a death [live 28 Oct]. A later rule calls a spot bad after a death within 8 s **and** within 3 m of it, or
  more than 5 m below it [4 Nov; not triggered live].

### Teleports and trigger regions

A teleport does not walk. It crosses no regions, and it can land inside one.

- **Skipping:** the Giant Lord's memory starts his fight when the player is inside a region just past the front fog. The trainer's
  teleport from the memory's start to the saved spot, 89 m past that region, never crossed it, and the Giant Lord stood still. The
  trainer now teleports into the region first, waits 1.5 s, then goes on [live 4 Nov, 13:43 and 13:53].
- **Landing in one:** the Dark Chasm path ends in a cylinder region. The trainer teleports the player into it and the game's own
  warp to the Darklurker's arena follows about 3 s later, in the same map [live 30 Oct].
- **Across a loading screen in one map:** the Frigid Outskirts are part of m50_37, so a teleport from the Eleum Loyce side landed on
  solid ground with the map drawn, no fall [live A8].

### An enemy's position

An enemy is not on this chain. A `CharacterCtrl` holds its world position as three f32 at `+0x90`. We found it by matching the
spawn positions of the Freja spiders, 0.0 m off at rest [live B13]. The player's chains do not apply to enemies. We never wrote
an enemy's position.

## How to do it

1. Check the build (image size `0x1D76000`) and that you are offline, as on the main page.
2. Warp to a bonfire in the target map and wait for the load (the map's flag blocks leave and return, HP above 0, then 2 s).
3. Walk the chain again; stop if a link is null or a class differs.
4. Write the position three times, the kick, and the quaternion, in that order, in one pass.
5. Read the position back after about 1 s. The body should be where you wrote it, to the centimetre.
6. If you wrote a spot for a trigger region, wait for the region's script before you move on.
7. Save spots you trust, with their facing. Do not compute a spot you cannot check by standing on it.

To look before you write: `teleport.py here` prints the class names along the chain, the shown position, the three physics
copies and the yaw.

## Pitfalls and what failed

1. **A bare position write (A7, 16:47).** The player stands, but the map does not draw until a step. Add the kick.
2. **Lifting the target (A7, 16:59 and 17:02).** 0.5 m up slid the body 3.2 m and left it 0.33 m high; 0.1 m up floated and did not
   draw. Neither lands on the spot. Do not use a lift instead of the kick.
3. **A spot inside a wall or the exit fog (28 Oct).** Computed spots, a 55 m guess for Aava, killed the player at once. Save spots
   by standing there.
4. **A fog object under the floor (28 Oct).** One fog part sat 4 m below its floor, so a spot taken from it was under the map.
5. **Skipping a trigger region (4 Nov).** The Giant Lord stood still after a teleport past his start region. Go through the region.
6. **Facing left at the warp's value (A7).** Without the quaternion the early arrivals kept the warp's facing, 90 degrees; the
   second one faced a wall.
7. **Reading position through a stale chain.** The body's address changed at every warp, and one was taken over by an enemy body.
   Walk the chain every time [live A7].
8. **The map origin is not a spot.** A respawn at (0, 0.1, 0) in the void is the respawn record's kind going wrong, not a teleport;
   see [The respawn point](../modding-world/respawn-point.md).

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Three position copies at +0x1A0, +0x1B0, +0x1C0 | the write log shows the same old 12 bytes at each of the three | live A7, write log 28 Oct 16:47 |
| A bare write leaves the map undrawn | the tester took a step to make it draw | live A7, 16:47 |
| A 0.5 m lift slides the body 3.2 m east and 0.33 m up | read after settling | live A7, 16:59 |
| A 0.1 m lift floats and does not draw | held 0.1 m up for 4 s or more | live A7, 17:02 |
| Velocity at +0x230; (0, -2, 0) lands exactly and draws the map | the old value was zero; the landing; the same offset on another body | live A7, 17:04, write log |
| Facing master is `MorphemeChrCtrl +0x30`, copy at +0x50 | the matrix and the motion forward followed at once | live A7, 17:10 to 17:15 |
| The quaternion is `(0, sin(yaw/2), 0, cos(yaw/2))` | 10.7 degrees stored as (0, 0.09288, 0, 0.99568); 90 degrees as (0, 0.7071, 0, 0.7071) | live A7, write log |
| The shown position is 0.05 below the physics one | the two read side by side | live A7 |
| A teleport works across the Frigid Outskirts' loading screen | solid ground, map drawn, no fall | live A8 |
| A teleport skips regions and can trigger one | the Giant Lord stood still, then woke; the Dark Chasm warp followed | live 4 Nov, 30 Oct |
| An enemy's position is `CharacterCtrl +0x90` | matched the spider spawns, 0.0 m off | live B13 |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/teleport.py` | `here`, `save`, `face`, `go`, `vel`: read, store and write the position and facing; `go <spot> [dy] [vx,vy,vz]` is the recipe; `face` lists other fields holding the yaw; `vel` prints the body's velocity whenever it is not zero. |
| `tools/arenalib.py` | `where()` reads the position and quaternion; `teleport(spot)` is the recipe; `player()` is the safe chain walk. |
| `tools/arenactl.py` | The trainer: lands at a saved spot after each warp or respawn (F7 lands again, Home saves a spot), with the region and end-region tables. |
| `pcmem.py` | Reads the shown position (`player` command) and writes with logging; offline only. |

## Open questions

- **Do we need all three position copies?** We always wrote all three [untested].
- **Do other kick values work?** (0, -2, 0) was the first value tried and the only one used [untested].
- **Does the order of the kick and the facing matter?** We always wrote position, kick, facing [untested].
- **A teleport between maps.** Every sample warped to the map first. Whether the position can be written across a map change is
  not known [untested].
- **A teleport during a fight, or while dead.** The trainer refuses to teleport automatically in the second case; nobody tried
  either by hand [untested].
- **Is `CharacterCtrl +0x90` the enemy's physics position or a copy?** It matched at rest; its behaviour in motion was not logged.

## Related pages

- [Memory map for PC](memory-map-pc.md): the pointer tree and the other structures.
- [Warping by script](../modding-world/warping-by-script.md): script warps, event locations and the map offset.
- [Moving a bonfire](../modding-world/moving-a-bonfire.md): landing spots and yaw for placed bonfires.
- [The respawn point](../modding-world/respawn-point.md): the record that decides where a death sends the player.
- [Live-testing tricks and pitfalls](../modding-scripts/live-testing-notes.md): watchers and write logs.
