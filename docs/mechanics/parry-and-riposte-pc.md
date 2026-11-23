# Parries and ripostes: which hits can be parried, and why Sir Alonne can't be riposted (PC Scholar 1.0.3.0, live 23 Nov 2026)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

**Questions.**
1. How does the game decide which enemy attacks can be parried?
2. Players disagree on whether Sir Alonne can be parried at all. Can he?
3. He can be parried, but nobody manages to riposte him afterwards. Why not?

**Answers.**
1. **Per hitbox, not per enemy.** Each attack hitbox points at a DamageCtrlParam row, and one byte of that row
   (+0x3) says whether the hit can be parried and whether it can be blocked. Two hitboxes of the same swing can
   differ.
2. **Yes, through one hitbox of one attack.** His leaping slam (attack 12) has two hitboxes: his sword, which can't
   be parried, and his body as he lands, which can. Live, every body touch inside the parry window was parried
   (11 of 11) and no sword-only touch was (0 of 6).
3. **His riposte zone is empty.** The riposte check tests where the player stands against a zone set on the
   **enemy being riposted**, in its EnemyCommonParam row. Alonne's zone is all zeros, so no position passes.
   Everything else about him is ready for a riposte. With a hollow soldier's zone copied into his row (in memory
   only), he was riposted at once. **535 EnemyParam rows have a parryable hit; 137 of them have an empty zone**, many
   bosses among them ([the list](riposte-zones-pc.md)).

All runs were logged frame by frame from outside the game (hitboxes, the player's hurtbox, HP, the parry and guard
state, the enemy's stagger counters) and most were recorded at 60 fps on one clock with the log. Memory writes were
made offline only and undone after each test. The logs, videos and per-run notes are kept with the source notes,
not on this site.

## 1. Which hits can be parried (and blocked)

The path from an attack to the flag:

1. The enemy's attack picks a group in EnemyBehaviorParam / EnemyBehaviorSecondParam / EnemyBehaviorThirdParam (or
   EnemyComboBehaviorParam for combo attacks). TimeAct event 2200's first integer is the index into the group's
   damage ids.
2. That gives an EnemyDamageParam row. Its first field, `damage_ctrl_id`, names a DamageCtrlParam row. Follow-up
   hitboxes (`damage_id_followup`) carry their own ctrl id.
3. **DamageCtrlParam byte +0x3** (SmithBox calls it `disable_parry`) holds two flags:

| Value | Parry | Block (shield) | Typical rows |
|---|---|---|---|
| 0 | yes | yes | most humanoid weapon swings |
| 1 | **no** | yes | big enemies, most bosses, whips, shield bashes, jump and plunge attacks |
| 2 | yes | **no** | 2 rows: Sir Alonne's 768000090, and 865100000 (added in Scholar) |
| 3 | **no** | **no** | throws and grab damage, ripostes, roars, quakes, gusts, shockwaves |

Player attacks reach the same byte through PlayerDamageParam. The rows are identical from PS3 v1.00 to PC Scholar
(199 changed between the 2013 Network Test and v1.00).

**On the defending side**, the hand used must be able to parry that kind of hit: WeaponTypeParam +0x52 for a player
(0 none, as on greatshields; 1 melee and projectiles, small and medium shields with spell parry; 2 melee only, every
other type; 3 projectiles only, greatshield spell parry). Then a facing test against the defender's parry angle.

**The parry window** is TimeAct event 1220 on the player's parry animation. For a player it opens only for the last
`parry_frames_duration` share of the event (WeaponParam +0x74). A Buckler has 1.0, the whole event.

Attackers have no "parryable" window in their animations. The ctrl byte is the whole attacker side.

## 2. Sir Alonne's leaping slam

Attack 12 ("leaping cut then crouching slashes", combo id 53). Its first part has two hitboxes:

| Hitbox | Damage rows | Byte | Parry | Block | Active |
|---|---|---|---|---|---|
| Sword | 168000410 -> 414 -> 415 | 1 | no | yes | about frames +2 to +19 |
| Body spheres (two, on his body as he lands) | 168000411 -> 412 | **2** | **yes** | **no** | about frames +0 to +11 |

All his other attack hitboxes are byte 1 or 3, so this body hit is the only parryable hitbox in his attack data.
The landing also spawns a shockwave (168000413, a bullet, byte 1) at about frame +9.

**Live, 22 slams (Buckler):**

| What touched you | Parry window | Slams | Parried |
|---|---|---|---|
| Body sphere | open | 11 | **11** |
| Body sphere, a 5.6 cm graze, same frame as the sword | open | 1 | 0 |
| Sword only | open | 6 | **0** |
| Body sphere | shut | 3 | 0 (hit) |

A parry cuts his whole attack short: his sword hitbox lived 7-9 frames instead of 18.

**Clean parry or parry plus shockwave damage.** Some parries still cost 351 HP. That came from the shockwave, and
it depended on how deep the body overlapped the player, not on button timing:

| Group | Body overlap at frame +7 | Attack stopped after | Shockwave |
|---|---|---|---|
| Clean (4) | 4 to 12 cm | frame +8 | never spawned |
| With damage (7) | -10 to +1 cm, a graze | frame +9 or +10 | spawned, 351 each |

So the parry lands on the first frame the body really overlaps you (see [contact margin](contact-margin-pc.md)).
Standing where his body comes down a few cm deeper gets the parry in before the shockwave spawns.

## 3. After a parry: the stagger and the riposte window

A parried enemy hits itself with damage 2400 (the same row for every character), which plays its parried stagger.
The stagger animations carry two Morpheme events, counted in the enemy's TimeAct block (T = [chr + 0xB8]):

- event **262** raises T+0x500 (roughly the stagger's start);
- event **261** raises **T+0x4FC** (its held loop). **This is the riposte window.**

Hollow soldier (EnemyParam 102010), 7 parries, times from the end of the parry motion:

| | Every parry |
|---|---|
| T+0x500 | from the parry to about +1.2 s |
| T+0x4FC | from about +1.2 s to about +3.5 s, or until the riposte |
| Ripostes | 6 of 6 started with T+0x4FC up (and T+0x500 already down) |
| One late press, after T+0x4FC closed | a plain swing, no riposte |
| Earliest riposte | +1.94 s in this run (+1.56 s later, on Alonne with a zone added) |

A riposte runs as a grab: the press starts the grab, and the player is locked into its animation.

## 4. The riposte check

Found by watching Sir Alonne's T+0x4FC with Cheat Engine ("find out what accesses this address") while parrying
and pressing attack. Besides the event handler that raises and lowers it, one instruction read it once per press,
at **0x141CF7A94** (image addresses, base 0x140000000), inside the exe's obfuscated code. Followed through the jump
stubs, the check works on the **victim** and fails (returns false) unless all of these hold:

1. T+0x4FC is not 0 (the riposte window);
2. T+0xFC is 9 or 12 (9 during the parried stagger; 12 not seen);
3. the victim's size class, EnemyCommonParam `chr_size` (+0x0), is 1-3;
4. a distance is no more than the victim's **`riposte_hitbox_radius`** (EnemyCommonParam +0x11C);
5. a height difference is no more than the victim's **`riposte_hitbox_height`** (+0x118);
6. an angle test built from the victim's **`riposte_hitbox_angle_xy`** / **`_angle_z`** (+0x110 / +0x114).

The values compared in 4-6 are filled in by obfuscated code that wasn't traced, so whether "distance" is centre to
centre or a gap between bodies isn't known. The fields' SmithBox names suggest an attacker's hitbox. **They are the
victim's zone.** A player's own row (ChrCommonParam 100: 45 deg / 45 deg / 1.0 / 1.5) plays no part when the player
ripostes: raising it to a 5 m height and an 86 deg angle changed nothing.

## 5. Why Sir Alonne can't be riposted

His EnemyCommonParam row 680000 has `riposte_hitbox_angle_xy`, `_angle_z`, `_height` and `_radius` all **0.0**. The
hollow soldier's row (102000) has 30 deg / 45 deg / 0.5 / 2.5.

What was ruled out first, live (dagger, presses inside his riposte window, 1.07-1.10 m straight in front of him, the
closest the two bodies allow):

| Tested | Result |
|---|---|
| His riposte window | **opens** like the hollow's (T+0x4FC from about +1.2 s to +3.6 s, 18 parries) |
| 18 presses inside it | plain swings, no grab |
| His stagger state T+0xFC | 9 throughout: passes check 2 |
| His size class | 3: passes check 3 (shared with the Old Dragonslayer, the Lost Sinner, the Throne Watcher) |
| EnemyCommonParam +0xEC set from 0 to 1 (1 on every other enemy compared) | no riposte in 20 presses, before and after a fresh spawn |
| EnemyParam +0x130 set 0 -> 1 and +0x16 set 3 -> 0 (his values that differ from the controls) | no riposte in 9 presses |
| The player's own riposte reach raised (above) | no riposte in 10 presses |

**Then the zone test:** with the hollow's four zone values written into his row in memory, two parries gave two
ripostes (the grab started +1.56 s and +1.66 s after the parry, from 1.08 m; 257 x 3 damage each). Both were
recorded. The row was set back to the file values after.

So in the shipped game he passes every check except the zone. The parry works, the window opens, and the riposte
is refused at check 4.

## 6. The Old Dragonslayer: blocked twice over

His quick thrust (move 15, byte 0) was parried on all 5 touches inside the parry window; his leaping slam (move 19,
byte 1) on none of 2. But after a parry he doesn't go into his parry-break stagger: his T+0x4FC never rose, in 6
parries. Only T+0x500 did, for 1.02 s from about +0.5 s after the parry. That matches the 1.0 s event-262 stretch of
his heavy-hit staggers; the parry-break loop, where his 261 event sits, never played. So check 1 fails whatever the
timing, and his riposte zone is empty as well. Attacks pressed after his parries (mace, dagger, fists) were all
plain swings.

## 7. Who else

By the rule above, an enemy with an empty zone can be parried but never riposted. Counted over EnemyParam rows in
the PC Scholar regulation: **535 rows have at least one parryable hitbox. 398 have a riposte zone, 137 have an empty
one.** The 137 include the Pursuer, the Throne Defender and the Throne Watcher, Velstadt, the Looking Glass Knight,
the Ruin Sentinels, the Belfry Gargoyles, the Skeleton Lords, the Old Dragonslayer, the Lost Sinner, Mytha, the
Dragonriders, Sir Alonne, Elana and the Fume Knight's rows, plus Kobolds, Ogres, Mimics, Basilisks, boars, Rupturing
Hollows, Bell Keepers and others. The full list is on [Parryable but never riposted](riposte-zones-pc.md).

Two limits on that list:
- A zone is needed but not enough. The enemy's parried stagger must also raise the riposte window (the Old
  Dragonslayer's doesn't).
- "Parryable" means the data has such a hitbox for the row. Some of those hits may be rare or unused in play.

It matches the author's experience that the Lost Sinner and the Throne Watcher can't be riposted. Only Sir Alonne
was tested live with a zone added.

## Not checked

- What fills the distance, height and angle values the check compares (obfuscated). With an empty zone no position
  passed, from 1.07 m and up, straight in front.
- What T+0xFC = 12 is.
- Other builds. The ctrl byte rows are the same from PS3 v1.00 to PC Scholar. The riposte zones and the check were
  only read in PC Scholar 1.0.3.0.

## Code reference (PC Scholar 1.0.3.0, image base 0x140000000)

| What | Where |
|---|---|
| Hit outcome (1 hit, 2 guarded, 5 parried) | `ChrDamageActionCtrl` vtable 0x1410bf178, slot 0x98 `FUN_140138510` |
| Parry test on the ctrl byte: `((b - 1) & 0xFD) != 0` refuses 1 and 3 | slot 0xb0 `FUN_140139f40` |
| Guard test: `(byte)(b - 2) > 1` refuses 2 and 3 | slot 0xb8 `FUN_140139eb0` |
| Projectile deflect (same test as the parry) | slot 0xa8 `FUN_14013a050` |
| Parry window (TimeAct 1220, [chr + 0xB8] + 0x550) | `ChrMorphemeTimeActTrackParryCtrl` `FUN_14016cd70`; read by `FUN_14030e1d0` |
| Parried attacker's self-hit, damage 2400 | `FUN_14030e630` case 5 |
| Events 261 / 262 -> T+0x4FC / T+0x500 | `FUN_140369890` (0x1403699F7 raises, 0x140369A03 lowers) |
| Stamina refill while T+0x500 is up | `FUN_140333900` (reads at 0x1403339AC) |
| **Riposte check** (window, state, size, zone) | from 0x141CF7A8A (obfuscated); fail path 0x140390A23 |
| Character's rows | [chr + 0x38] EnemyParam, [chr + 0x40] EnemyCommonParam (ChrCommonParam for the player) |
