# Live-testing tricks and pitfalls (PC Scholar 1.0.3.0, 2026-11-18)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 18 November 2026 from the Boss Arena project's tests.
> Test ids such as B19 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags are explained in the Evidence section.

## Summary

- **Fourteen habits, each paid for with a lost hour.** They come in the order you meet them: set the game up (1-4),
  finish fights quickly (5-8), watch without touching (9-10), deal with a crash (11-14). Our tools are named only as
  examples of the idea.
- **Test offline, in a copy, on a throwaway character.** The project's write tools refuse to run unless Steam's offline
  setting is on *and* the game has no outside connection. Every install on one Steam account shares one save file, so a
  character you care about is not a test character. [code; data]
- **Know what loads when.** Event scripts, talk scripts, maps and per-map params are read at each load of their map: no
  game restart. Bonfire names are read at game start. Game-wide ("rules") params were only ever seen after a restart.
  [live B13, B20d, B12, B20e]
- **Put a boss on 1 HP to finish a fight fast, but not at load.** The load-time write counts as a hit: it woke the Giant
  Lord and Vendrick before their fights. The Ancient Dragon died on the first hit while still friendly: kill count 1,
  but no fight flag, no kill flag, no souls. [live B19, B39]
- **The kill is credited about 6.5 to 7 s after HP reaches 0.** A warp or a death inside that window cancels it. Wait
  for the kill flag, never for HP 0. [live A6, B10]
- **Watchers should be read-only, log changes only and re-attach after a game restart.** A map-local flag or kill-count
  slot exists only while its map is loaded, so "not found" is not 0. [live A1, B38]
- **Crashes: read the Windows Application log first, and copy the dump out the day it is written.** Windows kept 10
  dumps, and every older one is gone. [data]
- **The main trap: your test tool is part of the experiment.** We mixed up the two sources of a failure three times.
  The Giant Lord's early wake and the Ancient Dragon's stuck rush looked like mod bugs and were the 1 HP tool (B19,
  B39 run 5). The Skeleton Lords' missing skeletons were read as a tool effect and were the game's spawn limit (B39
  runs 3-4). Switch the tool off and run it again before you blame either. [live B19, B39]

### Set the game up (habits 1-4)

**1. Steam in Offline Mode before anything writes to the game's memory.**

- Our write tools stop unless two checks pass. Steam's stored setting must read offline (`WantsOfflineMode` is `"1"` in
  `<Steam folder>\config\loginusers.vdf`), **and** the game process must have no TCP connection that is not loopback
  (read from Windows' extended TCP table, filtered on the game's process id). The setting alone is only a preference,
  so the second check looks at what the game is really doing. [code, `pcmem.py` `require_offline`]
- It is not a formality: at the 28 Oct check Steam was online (`WantsOfflineMode` 0), and the setup notes say to switch
  before any write. Every memory write described on [Memory map for PC](../tools/memory-map-pc.md) was made offline. [data; live]
- Why offline: the loader's own ini comment says the anti-cheat scans for the injection ([Mod Engine](mod-engine-ds2.md)).
  We never tried memory writes or modified files online, so we cannot say what would happen. [data; untested]

**2. Keep a debug copy beside a clean copy, and know what each one proves.**

| | Debug copy | Clean copy |
|---|---|---|
| What it is | an unpacked `Game` folder (UXM) with extra tools in it, such as a debug-menu add-on | the Steam install as Steam laid it down, plus `steam_appid.txt`; the mod is installed with the same steps a player gets |
| Use it for | fast edits, memory tools, new file kinds | the last check: does a player's install work |
| It cannot tell you | whether a player's install works. The unpacked exe is a changed exe (its signature reads HashMismatch, Steam's reads Valid) [data] | whether the mod is crash-free (see below) |
| What went wrong there | a plain-DCX regulation with params was ignored (B3 form 2; we never ruled out the copy as the cause), and a crash came from the add-on, not the mod (B27, B33) | a clean copy only tells you something if you read its crashes: check the Windows Application log after every session ([the ledger](crash-diagnosis.md)) |

- The clean copy is built from files no newer than the game's install date, with the large files (the archives and
  the exe) hard-linked, so it costs almost no space. The recipe is step 8 of [Mod Engine for DS2](mod-engine-ds2.md).
  [code `tools/cleantest.py`; live B49]
- **Close every other copy before you start one.** All copies share one save folder ([One save file for every
  install](saves-and-installs.md)), and without `steam_appid.txt` Steam relaunches the main install instead of the copy.
  [code; live B49 procedure]
- A second copy is also a diagnostic. The B41 crash happened in the Mod Engine copy and again in the debug copy with the
  same character seven minutes later. Two installs and one crash put the cause in the content, not the install route.
  [live B41; see [Text that crashes the game](../modding-menus/text-length-limits.md)]

**3. Test on a throwaway character, in its own slot.**

- A mod's load-time scripts write into any character that loads while it is installed, and some changes are permanent
  for that character. B56 asked for "a disposable arena character" because a Bonfire Ascetic burned in the test cannot be
  undone. [live B56; see [One save file for every install](saves-and-installs.md)]
- A level 1 character cannot kill a boss, and it does not need to. The first tests used one (all nine stats at 1, no
  gear) and put the boss on 1 HP instead (habit 5). From B12 the same character had all stats at 99 and a +10 weapon,
  still with the 1 HP tool, because the point is speed. Keep that character in its own slot and use a new slot for a
  fresh-player test. [live B8, B12, B23]
- **Leftover state in a save can hide a bug or fake one.** A kill flag left on by an earlier trainer run made the first
  Nashandra trip stand down (B16). A race that the game's own code won on one save (the ship at No-man's Wharf, B39) had
  passed in B13, probably because that save already had the flag on. [live B16, B39; inference for B13]
  Test a flag-dependent feature on a save in the state a player has, as well as on the one you have.

**4. Know what loads when, and prove the new file was read.**

| What you changed | When the game reads it | Test |
|---|---|---|
| Event script (`event_<map>.esd`) and its `EventParam_<map>` row | each load of that map | B13, B54 |
| Talk script (`talk_<map>.esd`) | each load of its map. The test plan said "needs a restart"; the same game process (same pid) read the new script at the next Majula load | B20d |
| Map (`.msb`) and per-map params such as `GeneratorParam_<map>` | each load of that map. Freja's row (a moved bonfire and removed enemies) was rebuilt three times with the game running and the player out of her map, and the last build passed | B13, B8 |
| Bonfire names (`bonfirename.fmg`) | game start only | B12 |
| Other texts (`mapevent.fmg` and so on) | not tested alone: every text change was installed closed and tested after a restart | [inference] |
| Game-wide params (the regulation, or loose `Param\` files) | every change was installed with the game closed and seen after a restart; a change with the game running was never tried | B20d, B20e, B28 |
| Files added to or removed from Mod Engine's override folder | the loader caches file lookups, so a restart may be needed | [data] |

- **Install with the game closed.** The exception is files read per map: they went in while the game ran as long as
  the player stood outside the maps being rebuilt. A scratch helper waited for the map to unload, then built. [live
  B13, B54; the 5 Nov notes]
- **Build before you launch.** B12's build ran while the game was up, and the names only changed after a restart.
  One restart on 5 Nov still showed the old names with the right file in place, and the next restarts showed the new
  ones. We never found why. [live B12, B13, B15]
- **Make every test say how you will know the new file was read.** Read the edited field in memory ([Editing
  params](editing-params.md)), give the mod a mark players can see (the title screen's version line), or write down the
  game's process id before and after the install. B53's fail sign was "fireballs still flying: the old param file
  loaded, restart the game". [live B20d, B53; B48]
- **A bad text sits quietly until its window opens.** A 131-character question crashed the game only when the player
  said YES to the question before it. Open every window you changed, in every language you fill. [live B41; see [Text
  that crashes the game](../modding-menus/text-length-limits.md)]

### Finish fights quickly (habits 5-8)

**5. The 1 HP sweep: put the boss on 1 HP so any hit wins by the game's own death.**

1. **Find the live boss.** Take his `EnemyParam` row id, find the row's copies in memory, then every live character whose
   `+0x38` points at a copy (HP is at `+0x168`). Human NPC-type bosses are `PlayerCtrl` objects, so accept both classes
   and skip your own player. [live A10, B13, B17; the method is on [Memory map for PC](../tools/memory-map-pc.md)]
2. **Write 1 only when his HP is above 1,** and check the class and row pointer again right before each write, because a
   reload frees the object. [live B8; the code]
3. **Trigger it two ways.** At the moment his fight flag comes on, and with a sweep every 3 s for any boss whose map is
   loaded and whose kill is not credited yet. The sweep is there because some fights never set a usable flag: the
   Huntsman's Copse passes its fight flags only as arguments to a shared boss routine, so they never came on. [live B12]
4. **Re-arm after every load.** A death in the same map reloads it in about 1 s. The 3 s sweep missed the gap, and
   Nashandra came back at full HP. The fast loop now counts loads (the flag blocks leave and return, or the player's HP
   cannot be read), and the sweep forgets what it has done on every load. [live B16; code `tools/batchwatch.py`]
5. **Hit every enemy of a group fight.** The Skeleton Lords are three generators; the Ruin Sentinels, the Belfry
   Gargoyles and the Twin Dragonriders are several enemies each. Every one needs 1 HP and a hit. [live B12, B39]
6. **Scan in the background, with a gap.** The first scan for a row's copies takes 5 to 11 s, once per game session. B39's
   run 1 scanned 59 rows (about 8.5 s each), one after another, with the lock re-taken at once, so the fight-start write
   and the sweep waited about 8 minutes: the Last Giant was not on 1 HP. A 0.5 s pause after each scan fixed it. [live
   B39]

**6. The load-time 1 HP counts as a hit, and wakes any boss that wakes on damage.**

- B19: the Giant Lord woke as soon as the player crossed the fog and walked 100 m to the player. It looked like a mod
  bug. It was not: the sweep had put him on 1 HP at load (16:23:19), and his own routine wakes him within 30 m **or
  when he is hurt**. No AI data had been changed. [live B19]
- Vendrick did the same in B39 (the sweep woke him, the fog was already up on arrival; the fight and the kill were fine).
  [live B39]
- The fix is a short list of bosses the sweep must skip, so their 1 HP waits for the fight flag. Ours is the set called
  `WAKE_ON_HURT`: the Giant Lord, Vendrick and the Ancient Dragon. Of 44 bosses, three needed it. [live B19, B39; code
  `tools/batchwatch.py`]
- We found the first two by watching them wake. We have not read the other routines for a wake-on-damage condition.
  [untested]

**7. A boss killed while friendly gets no kill flag (the Ancient Dragon).**

- Her boss routine (the boss bar, the fight flag, the kill flag) starts only when talk flag 103970 is on. Her talk
  script sets it after four counted hits. On 1 HP, the first hit killed her while she was still friendly. The kill count
  went to 1, but her fight flag (127020080) and kill flag (127000081) never came on, so there were no souls, and the
  mod's warp on the kill flag never fired. A live read showed 103970 at 0. [live B39 run 5]
- At full HP four hits cannot kill her, so a player is not affected. Her routine is the only one of the 44 gated on a
  talk flag; the others start on "player in region" or at once. [live B39; data]
- What to take from it: **HP 0 is not kill credit.** The game grants the kill flag inside the boss routine, and if that
  routine never started there is none. A kill *count* is not a kill *flag* either
  ([Bringing a boss back](../modding-world/respawning-bosses.md)). A quick-kill tool must wait for the flag that starts the routine.
- To carry on past the stuck fight, the test set her kill flag by hand with a logged write. The mod's win flow cleared it
  1.3 s later and warped on, so the part after the kill was tested without redoing the fight. A step you cannot reach
  naturally can be tested by setting its input flag. [live B39]

**8. Kill-credit timing: a warp or a death before the credit cancels the kill.**

| When | What happens |
|---|---|
| HP reaches 0 | a fight-state flag turns on (110020084 for the Last Giant); we read it as "HP 0" [inference] |
| +6.5 to 7 s | the death animation ends: kill flag, map-clear flag, souls and kill count, all in one tick |
| a warp at +6.1 s | the Forest unloaded and the kill was lost: no souls, no flag, no count. The boss was back at the next visit. No crash |
| a death at +4 s and +6 s | the kill is cancelled the same way |
| a death at +5 s after the credit | the win is kept, and the player comes back at the bonfire |

[live A2, A6, B10, B12]

- Anything that fires on a win (a warp home, a script step) must wait for the **kill flag**. Never fire on HP 0.
  [live A6]
- Put the 1 HP tool in your test, but test the window on purpose. A tool shortens the fight, not the 6.5 s. Try a warp
  or death just before the credit (cancelled) and just after (kept). [live A6, B10]
- The whole timeline, and how the arena group uses it, is on [Bringing a boss back](../modding-world/respawning-bosses.md).

### Watch without touching (habits 9-10)

**9. Read-only watchers that log changes and survive a game restart.**

- A watcher polls flags, map blocks, HP and the respawn record every 0.1 to 0.5 s and logs **only what changed**, with a
  time, to the screen and to a dated file. A 3-hour log stays readable, and the timeline is the evidence: a 50 ms flag
  watcher showed the 6.5 s between HP 0 and the kill (A2). [live A2, B19]
- Keep watching and writing apart. The tools that only read said so in their first line, and every write went through one
  library that checks Steam's offline state and logs the old and new bytes. [code `pcmem.py`; `tools/testwatch.py`]
- **Re-attach.** Start the watcher before the game: it waits for the process and for a loaded character. Treat failed
  reads as normal while a map loads, and call the game gone only after about 30 s of failures (loading screens are
  shorter), or when the process has exited or its id has changed. [code `tools/testwatch.py`]
- What went wrong without it: the first `testwatch` went silent after a restart (6 Nov); after a restart in B19 the
  watcher had lost the game, so the Darklurker's chasm test has no log; `fltrace` stalled on the dead process and
  missed the bonfire rests (B38): reads of a dead process fail with a read error, not with "game not found", so compare
  the process id; and the 1 HP watcher was tied to the previous process and had to be restarted (B39). [live B19, B38,
  B39]
- **Walk the pointers again after every load.** The player object and the characters are rebuilt at each load, and any
  scan result you cached belongs to one process id. [live A7, B57; see [Memory map for PC](../tools/memory-map-pc.md)]
- **Poll faster than the thing you want to see.** A load drops the flag blocks for about 0.3 s, so a 3 s poll cannot
  see a same-map reload. [live 28 Oct logs, B16]
- **Keep slow work off the fast loop.** The heap scans held up the main loop in B12, and a 1 s kill flag went unlogged.
  Run scans in a thread. [live B12]
- A quiet log is evidence too. After a game close that left no crash event and no dump in the Windows log, the notes say
  so (B38, the 14:51 close). [live B38]

**10. Read flags and memory only inside the map that holds them.**

- A map-local flag (the Last Giant's kill flag 110000081 is in block 11000) is in memory only while that map is loaded.
  From another map a reader finds no block and the setter returns 0 and does nothing. [live A1]
- Arena flags were sometimes readable from Majula by luck. Do not build on that: B38 found the arena blocks usually
  absent. So a test that must set a flag has to do it from inside the map. `tools/stalekill.py --rearm` waits for a real
  kill and sets the flag back at once, in the map, for that reason. [live B38]
- A map's kill-count slot exists only after the map has loaded in this game session. After a restart the first lookup
  found nothing from Majula and 0 inside the map (4 Nov). Treat "not found" as "unknown, look again after the load", never
  as 0. [live 4 Nov; see [Memory map for PC](../tools/memory-map-pc.md)]
- A boss does not exist at map load either. He appears as the player nears his arena, so search after arrival. [live A10]
- **A loaded flag block does not mean the player is there.** A neighbouring map's block loads while it streams in, and
  its scripts run. In B39 the Royal Rat Authority's map streamed in while the player was still in Najka's map next door,
  and its load group acted as if the player had arrived. A watcher keyed on "this map's block is loaded" logs the
  neighbour as if the player were in it. In a script, wait for the player to be in the map. [live B39; see [Event
  script groups](event-script-groups.md)]

### When it crashes (habits 11-14)

**11. Use the Windows Application log as a crash ledger.**

- Each crash of the game exe is an "Application Error" event (id 1000) with the exception code, the faulting module, the
  fault offset and the exe's path, which says which copy crashed. It outlives the dump: on 17 Nov our log still held the
  first game crash, of 24 Oct. [data]
- Group by code, module and offset. One group is one crash; a new group after your change is a new crash, not the old one
  moved. (Do not group heap crashes by the ntdll offset alone: it is only the report.) [data; the snippet and the
  ledger are on [Diagnosing crashes](crash-diagnosis.md)]

  ```powershell
  Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='Application Error'} |
    Where-Object { $_.Message -match 'Faulting application name: DarkSoulsII\.exe' } |
    ForEach-Object { $m = $_.Message
      [pscustomobject]@{ Time = $_.TimeCreated
        Code   = [regex]::Match($m, 'Exception code: (\S+)').Groups[1].Value
        Module = [regex]::Match($m, 'Faulting module name: ([^,\r\n]+)').Groups[1].Value
        Offset = [regex]::Match($m, 'Fault offset: (\S+)').Groups[1].Value } } |
    Group-Object Code, Module, Offset | Sort-Object Count -Descending | Format-Table Count, Name
  ```

- **Read it after every session, even when nothing seemed to crash.** A crash on quit, or one that happens while you
  are looking elsewhere, never reaches your test notes, but it does reach the Application log. See [Diagnosing
  crashes](crash-diagnosis.md).

**12. Copy the crash dump out before Windows deletes it.**

- With the default settings Windows writes a minidump to `%LOCALAPPDATA%\CrashDumps\<exe>.<pid>.dmp` and keeps only the
  newest 10. By the time we went back for them, the five dumps that named the heap-corruption culprit and the one from
  the B41 text crash were gone. The folder held exactly 10 files, and each new crash pushed an older one out. [data;
  inference for the rule, which we did not look up]
- Windows writes the dump a few seconds after the process ends. Wait before you look (our watcher waits 8 s).
  [code `tools/bastillewatch.py`]
- The event shows the process id in hex and the file name in decimal (0x20C4 is 8388). Other programs' dumps share the
  folder and count towards the 10: two of the ten were not the game's. [data]
- Copy it the day it is written. **Do not share a dump:** its module list holds full local folder paths and its memory
  can hold anything. To keep more, Microsoft documents a `LocalDumps` key with a `DumpCount`; we never set it. [data;
  untested]

**13. Count the loads when a crash is intermittent, and say what each result would mean first.**

- B27 logged every Lost Bastille load by itself (a watcher: "ok" when the map's block was loaded and a character had
  been in the world 15 s, "crash" when the process ended and a new dump appeared). It worked out the odds before the
  run (12 loads without a crash at the mod's rate would have been 7 %) and stopped at the first crash with the original
  files: 6 loads and 1 crash against 11 loads and 2 for the mod's. [live B27]
- That cleared the mod's files, and it did not name the culprit. Removing the suspect did (B33). [live B27, B33; the
  chain and the add-on are on [Diagnosing crashes](crash-diagnosis.md)]
- **Change one thing at a time** (the files, the add-on, the copy of the game), and write the result as a count. [live
  B27, B33, B41]
- **The quit-and-load loop** that B27 used: (1) travel into the map and quit to the desktop, so the save sits in that map;
  (2) repeat about 12 times: start the game, load the save straight into the map, wait until you can move (about
  15 s), quit to the desktop; (3) tally. Run it with the watcher on, so no load goes uncounted. [live B27]
- Quitting and loading again is also how you test what a save kept: B20d's last step quit and reloaded, and the setup
  items were still there. [live B20d]

**14. A crash in the minute after your new feature is not the feature's until the stack says so.**

- The trainer's event warp was dropped as unsafe after a 28 Oct crash. The dump's chain, read on 6 Nov, named a
  debug-menu add-on and not the warp. [live B27; see [Warping the player by script](../modding-world/warping-by-script.md)]
- A fix proves a cure, not a cause. B41's reword cured the crash and the cause we named at the time (line width) was
  wrong; reading the exe gave the real one (127 characters). [live B41; code]

## How to do it

A test session, start to finish.

1. **Close every other copy of the game.** Set Steam to Offline Mode. (Habits 1, 2.)
2. **Install with the game closed** unless every changed file is read per map and the player is outside those maps.
   Note what you changed and which of those files need a restart (the table under habit 4). (Habit 4.)
3. **Start your watchers first.** They wait for the game and re-attach after a restart. Check the watcher's log for a
   "character loaded" line after each launch. Restart any tool tied to the old process id. (Habit 9.)
4. **Launch, load the throwaway character,** and check that the new file was read: an edited field in memory, a visible
   mark, a changed process id. (Habits 3, 4.)
5. **For a fight, use the 1 HP tool,** and keep the three bosses that wake on damage, or any talk-gated routine, out of
   the load-time sweep. (Habits 5-7.)
6. **If the step fires on a win, test the credit window:** a warp or a death just before the credit, and one just
   after. (Habit 8.)
7. **Open every window and screen you changed,** in every language you fill. (Habit 4.)
8. **After the session read the Application log,** copy any new dump out of `CrashDumps`, and read the watcher log for
   "game lost" gaps. (Habits 9, 11, 12.)
9. **Write the result with times from the watcher log** and one evidence tag per claim: `[live Bxx]` only for what you
   saw. If a test fails, switch the tool off and run it again before you blame the mod or the tool. (Summary.)

## Pitfalls and what failed

| Test | What happened | Result | Lesson |
|---|---|---|---|
| B19 (5 Nov) | The Giant Lord woke when the player crossed the fog and walked 100 m | Not the mod: the load-time 1 HP counted as a hit | Ask what your tool touches before you ask what your mod changed |
| B39 run 1 (7 Nov) | The Last Giant was not on 1 HP | A first reading blamed a nested list in the boss table; the real cause was a lock starved behind 59 row scans | Time-stamp the tool's own actions as well as the game's |
| B39 run 5 | The Ancient Dragon died on the first hit, still friendly | No fight flag, no kill flag, the rush stuck | HP 0 is not kill credit (habit 7) |
| B39 runs 3-4 | The Skeleton Lords' skeletons never came, so the fight could not end | Run 3 was read as "a test-tool effect". Run 4 at full HP did the same: every skeleton spawner has `SpawnLimit` 1 ([Bringing a boss back](../modding-world/respawning-bosses.md)) | Rerun without the tool before you blame it |
| B16 (5 Nov) | After a death in the same map Nashandra was back at full HP | The 3 s sweep missed the 1 s reload | Count loads in the fast loop (habit 5) |
| A6 (28 Oct) | A warp as the boss's bar emptied | No souls, no flag, no count | The credit comes 6.5 s after HP 0 (habit 8) |
| B20d (6 Nov) | The test text said "needs a game restart" | The same process read the new talk script at the next Majula load | A planning note is not a result. Log the process id and check |
| B12, B13 (4 and 5 Nov) | Names unchanged after the build | The build ran with the game up. Later, one restart still showed old names | Build first, then launch (habit 4) |
| B13, B39 | Flexile Sentry's arena passed in B13, then the player arrived in the sea in B39 | The game's single look at a flag at load won a race against our group. The B13 save probably had the flag on already | State left in a save can hide a race (habit 3) |
| B38 (7 Nov) | The trace stalled on the old process after a restart and missed the bonfire rests | A read of a dead process is a read error, not "game gone" | Compare the process id (habit 9) |
| B12 | A 1 s kill flag went unlogged | Heap scans held up the watcher's main loop | Slow work in a thread |
| B27, B33 | "Not our files" was right; "the game or the debug copy" was half right | The add-on in the debug copy | Clearing your files does not name the culprit (habit 13) |
| All dumps | The folder holds 10 | Dumps from 28 Oct to 13 Nov are gone | Copy a dump out the same day (habit 12) |

## Evidence

Tags: `[live Bxx]` seen in the game; `[code]` read in the exe or a tool; `[data]` read in the game files or the machine's
logs; `[inference]` reasoned; `[untested]` an idea only.

| Claim | How we know | Test or source |
|---|---|---|
| The write tools need Steam's offline setting and no outside TCP connection | code | `pcmem.py` `require_offline` |
| Steam was online at the 28 Oct check | read from `loginusers.vdf` | the setup notes, 28 Oct |
| Per-map event scripts and params load with the map, no restart | built with the game running, seen at the next load | B13, B54 |
| A talk script loaded at the next map load in the same process | process id unchanged since launch | B20d |
| Bonfire names need a game restart | built with the game up, names unchanged until a restart | B12 |
| One restart on 5 Nov still showed old names | live; cause unknown | B13, B15 |
| Game-wide params were seen only after a restart | all installs were made closed | B20d, B20e, B28 |
| A plain-DCX regulation was ignored in the debug copy | memory read | B3 form 2 |
| The load-time 1 HP woke the Giant Lord and Vendrick | live; the watcher log | B19, B39 |
| The Ancient Dragon died friendly: count 1, no fight or kill flag, 103970 at 0 | live read | B39 run 5 |
| Hers is the only boss routine of the 44 gated on a talk flag | scan | B39 |
| Kill credit about 6.5 to 7 s after HP 0; a warp at +6.1 s lost it | live: flag timelines | A2, A6, B8, B10 |
| A death before the credit cancels, after it keeps the souls | live: six fights | B10 |
| A map-local flag cannot be read or set from another map | live | A1 |
| Kill-count slots exist only after the map has loaded this session | live | 4 Nov |
| A same-map reload drops the flag blocks for about 0.3 s | live: 50 ms watcher | 28 Oct logs |
| A streamed-in neighbour map runs its scripts | live | B39 |
| The Windows log outlives dumps; the dump folder keeps 10 | data; the 10-file rule is inferred | read 17 and 18 Nov |
| 12 loads at the mod's rate would have shown a crash 93 % of the time | arithmetic written down before the run | B27 |

## Tools

All in `tools/`, named as examples of an idea. Nothing here is needed to use the habits.

| Tool | The idea it shows |
|---|---|
| `testwatch.py` | a read-only logger of changes that waits for the game, then re-attaches after a restart |
| `batchwatch.py` | the 1 HP sweep with its `WAKE_ON_HURT` skip list, a load counter and a background warm-up |
| `fltrace.py` | detecting a restart by the process id when reads fail |
| `stalekill.py` | setting a map-local flag from inside its map, at the moment you need it |
| `bastillewatch.py`, `bastille_ab.py` | an automatic load count for an A/B crash test |
| `cleantest.py` | building a clean copy of the game as Steam installed it |
| `pcmem.py` (the memory library, kept outside `tools/`) | every write logged with old and new bytes, refused unless Steam is offline |

## Open questions

- **Are game-wide ("rules") params re-read at a map load, or only at boot?** Only restarts were tried. [untested]
- **Do `mapevent.fmg` and the other texts need a restart,** as `bonfirename.fmg` did, or do they reload at a map load? Only
  the names were tested alone. [untested]
- **Which other bosses wake on damage?** Three needed the skip list; the other routines were not read for that
  condition. [untested]
- **Why did one restart on 5 Nov still show old bonfire names?** [untested]
- **Would a larger `DumpCount` keep the dumps?** We never set it. [untested]
- **Is a write or a modified file safe online?** Every test was offline, and our tools refuse to try. [untested]

## Related pages

- [Bringing a boss back](../modding-world/respawning-bosses.md): the kill timeline, the kill count and the stale-flag cases
- [Memory map for PC 1.0.3.0](../tools/memory-map-pc.md): the pointers behind the watchers and the 1 HP tool
- [Diagnosing crashes from minidumps](crash-diagnosis.md): the ledger, reading a dump, and the worked examples
- [Text that crashes the game, and missing glyphs](../modding-menus/text-length-limits.md): the crash that only came when a window opened
- [Event script groups](event-script-groups.md): when a group runs, and why a streamed-in map runs its groups
- [The legacy DS2 Mod Engine](mod-engine-ds2.md): the clean-copy recipe and the loader's settings
- [One save file for every install](saves-and-installs.md): why a test character is not a normal one
- [Editing params](editing-params.md): what a restart is needed for, and how to check a param was read
- [Writing ESD files](writing-esd-files.md): the talk-script reload result (B20d)
- [Warping the player by script](../modding-world/warping-by-script.md): the warp that was wrongly blamed for a crash
