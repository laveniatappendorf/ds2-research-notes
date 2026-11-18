# One save file for every install (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed in code"
    Read from the game's code or logic (disassembly or data), but not run live. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests; trimmed to the general lessons on 18 November 2026.

## Summary

- **Every install of Scholar on one Steam account reads and writes the same save file**, `DS2SOFS0000.sl2` in
  `%APPDATA%\DarkSoulsII\<Steam id>\`. It holds all ten character slots. A second copy of the game, a debug copy and a
  modded copy all share it. Two installs separate the game files, not the saves [data; the project's debug copy and
  its Steam install shared it from the start].
- **A mod's load-time scripts write into every character that loads while the mod is installed**, with no player
  choice. A group that clears a boss's kill flag, or sets a world flag, at load does it to a normal character too, and
  the change stays in that character's save. A normal character opened in a modded install can come out partly
  changed [code]. Nobody has yet loaded a normal character and compared before and after [untested].
- **Mod Engine's "alternate save file" option does nothing in DS2.** With the setting at its default and with
  `useAlternateSaveFile=1` and the debug console on, the game wrote `DS2SOFS0000.sl2` both times. No `.sl3` appeared.
  The dll even logged "Patching save file location" [live B41 run 1, B45].
- **The exe builds the file name from separate strings**: a prefix `DS2SOFS` (UTF-16, found once), a `%s%04d` format and
  `.sl2`. Giving an install its own file by changing the prefix in the exe was built and checked offline, then put on
  hold. It was never run in the game [code; B35, on hold]. So a separate save needs more than a Mod Engine setting: the
  name is built inside the exe.
- **Advice that holds for any mod.** Players: back up the save file, keep a modded copy and a normal copy, and swap the
  copies when you install or remove a mod. Modders shipping through a loader: say so plainly, because nothing in the
  game warns.
- **Main trap:** the mistake is silent and lasting. A normal save loaded with the mod keeps the changes. A save made
  with the mod and loaded without it can name rows the game does not have, for example a respawn record at a mod-only
  bonfire. What the game then does is untested [inference; see [Adding a new bonfire](../modding-world/adding-a-bonfire.md)].

### One file, ten slots, shared by every install

The save folder is `%APPDATA%\DarkSoulsII\<Steam id>\`. On 17 November it held two save files (and a zero-byte file
of unknown origin that is not part of any save) [data]:

| Entry | Size seen | What it is |
|---|---|---|
| `DS2SOFS0000.sl2` | 8,251,680 bytes | Scholar's save file. Ten character slots, per the project's 7 Nov notes (not re-read for this page). |
| `DARKSII0000.sl2` | 7,203,104 bytes | Presumably the original (non-Scholar) DS2's file: the same folder, a different prefix [inference]. The Scholar exe holds no `DARKSII` string [data]. |

Sizes are of the project's own files, given only for scale. A `DS2SOFS0000.sl2.bak` copy is written at times. The folder
had none on 17 November [data], so a backup step must say "if it is there".

The original game and Scholar already live side by side in one folder, kept apart only by the file prefix. That is the
only separation the game itself offers, and every Scholar install on the account uses the Scholar prefix.

### How the exe names the file [code]

Read in the stock 1.0.3.0 exe (28,200,992 bytes) on 17 November. Offsets are RVAs (add 0x140000000 for a Ghidra
address) and file offsets. Each string is read from one place in the first `.text` section (a scan for RIP-relative
`lea` references; a pointer held in a data table would not show up in that scan).

| String | UTF-16 file offset | RVA | Read at (RVA) | How it is used |
|---|---|---|---|---|
| `SaveData` | 0x10D92C0 | +0x10DA4C0 | +0x2E6FBE | stored, length 8, into a wide-string member of an object. Its role was not traced. |
| `DS2SOFS` | 0x10D92D8 | +0x10DA4D8 | +0x2E8249 | handed to a wide-string helper (+0xA88130) as a NUL-terminated string, with no length argument |
| `%s%04d` | 0x11B4EF8 | +0x11B60F8 | +0xA8D340 | a format: a string plus a number read from a field of the object (+0x38). The number would be the `0000` [inference]. |
| `.sl2` | 0x11B4438 | +0x11B5638 | +0xA8739E | appended with an explicit length of 4 to another wide-string member |

What this tells a modder:

1. **The name is a prefix, a four-digit number and an extension.** `DS2SOFS` + `0000` + `.sl2`. The prefix appears once
   in the exe, in UTF-16. It is the only thing that separates Scholar's file from the original game's.
2. **Prefix and extension are separate strings.** The extension is one that a loader might try to swap (see Mod
   Engine, below). The prefix is the one that makes a different file.
3. **The prefix is passed with no length argument; `SaveData` and `.sl2` are passed with one.** So the prefix may not be
   tied to seven characters in the game. The project's tool only ever writes a same-length string, so an in-place edit
   keeps the exe's size [inference; the helper was not read].
4. **A Mod Engine install does not change the exe.** Mod Engine loads through `dinput8.dll` and overrides files at run
   time. A file-by-file check of a clean Mod Engine install found no game file changed [live B49, B58]. The Steam exe's
   signature reads Valid on 17 November. The debug copy's exe, which was unpacked with a modding tool, reads
   HashMismatch [data]. Anyone running a UXM-unpacked copy already runs a changed exe.

This page gives no patch recipe for this string: the experiment (B35) is on hold and was never run. The tool,
`tools/savefile.py`, is described under Tools.

### What a modded install writes into any character that loads

Loading a map is enough. A mod's script groups run at every map load, whatever the player chose, and they can write any
flag or record [code]. See [Event script groups](event-script-groups.md) for how a group runs at load. Two small examples
from the Boss Arena's build code, neither seen on a normal save:

- A group that sets a boss's kill flag to 0 at load brings that boss back, and it stays back in the normal game too
  [inference]. See [Bringing a boss back](../modding-world/respawning-bosses.md).
- A group that sets a flag to 1 before a script runs leaves the flag on. At Earthen Peak (m10_17) the windmill flags
  117000055 and 117000056 are set to 1 before Mytha's pools drain. The game's own group only ever sets them, so the
  pools stay drained for good [inference]. See [Map object states](map-object-states.md).

Changed rules are different. Edited params and shop or level-up cost files write nothing to the save, but whatever the
player buys, levels or upgrades while the new rules apply stays: levels, items, upgrades [code].

Two more kinds of data live in the save and belong to other pages:

- **The save's bonfire list.** It is rebuilt from `MapObjectBonfireParam` at every load, so a save written with the mod
  holds entries for the two mod-only bonfires, 3650 and 10685 (77 -> 79 entries) [live B2, B55]. See
  [Bonfire lit state](../modding-world/bonfire-lit-state.md).
- **The respawn record** `{map, kind, id}`. After a script warp the guard can set it to one of a mod's own arrival rows
  (kind 1). Resting at a mod-only bonfire sets kind 0 with the bonfire id [live B44, B2]. See
  [The respawn point](../modding-world/respawn-point.md) and [Warping by script](../modding-world/warping-by-script.md).

None of this corrupts a save. The game reads every value as a normal flag or record. The risk is a changed story, not a
broken file.

### What stays when the mod goes

Taking a mod out leaves the save as it was. Whatever its load-time groups wrote stays: flags, items, levels. A save
loaded without the mod can also name rows the game lacks: a respawn record can name a row that only the mod has. The
nearest case we saw: a record of kind 1 ("event location") that held a bonfire id, a row the map does not have, put every
respawn at the map's origin, in the void [live, 4 Nov death loop]. Our arrival rows (EventLocation 9907xx) do not exist
in the unmodded game, so a kind-1 record naming one would most likely behave the same [inference]. A kind-0 record
naming a mod-only bonfire was never tried [untested]. See [The respawn point](../modding-world/respawn-point.md) and
[Adding a new bonfire](../modding-world/adding-a-bonfire.md#the-saves-and-uninstall-risk).

### Keep a modded and a normal save apart by swapping files (any mod)

This needs only file moves, and it works for any mod that shares the game's save.

1. **Close the game.** Find the folder `%APPDATA%\DarkSoulsII\<Steam id>\`. Use the folder with the long number.
2. **Back up.** Copy `DS2SOFS0000.sl2` (and `DS2SOFS0000.sl2.bak`, if it is there) to a safe place. Label it "normal".
3. **Move those files out of the folder** before the first modded session. The game starts a fresh `DS2SOFS0000.sl2`
   with the mod installed [live B58, 16 Nov: the save moved out, the game started with a fresh file, a new character
   was made]. Do not load a character you want to keep unchanged while the mod is installed.
4. **Back to normal play:** close the game, remove the mod's files, move the modded save pair out and put the normal
   pair back. Keep the modded pair only if you may reinstall, and do not load it without the mod. [Removing the mod was
   run live (B58: the check found no leftover file). Swapping the saves back was not exercised end to end
   [untested].]

A save-manager tool that copies the whole file at the main menu does the same job. Two cautions from reading such a
tool's description, not from running one: it swaps all ten slots at once, and a normal character loaded while a mod is
installed still picks up the mod's writes [inference].

### Advice for a modder shipping through a loader

- **Say it plainly.** The game gives no warning, and the load-time groups run on any character [code]. Tell players that
  the mod shares the normal save, and how to keep a copy.
- **Do not lean on Mod Engine's alternate-save setting.** It does nothing in DS2 (below).
- **Treat every flag you write as permanent.** Once a release is in someone's save, its flag ids and arrival rows can
  never move [inference]. See [Event flags and choosing safe ids](event-flags-and-free-ids.md).
- **Test on a throwaway character** in a save file you can lose.

### Check which save file a build writes

This is how B41 and B45 were run, and it works for any install, mod or not.

1. **Snapshot.** With the game closed, record the names, sizes, times and a short hash of every file in the save
   folder. `tools/savefile.py snapshot` and `tools/modengine_test.py snapshot` do this and write names only, with no Steam
   id.
2. **Play** as far as the first autosave (a minute past the opening of a new game), then quit.
3. **Compare.** `compare` lists each file as new, changed or unchanged. A build that moved its save shows a new file
   and an unchanged `DS2SOFS0000.sl2`. A build that did not shows `DS2SOFS0000.sl2` changed and nothing new.

### Try Mod Engine's alternate-save option (what B45 did)

1. Take a copy of the game with Mod Engine (see [The legacy DS2 Mod Engine](mod-engine-ds2.md)).
2. In `modengine.ini`, add a `[savefile]` section with `useAlternateSaveFile=1`, and set `showDebugLog=1` so the dll
   opens a console.
3. Start the game and read the console. The dll prints "[ModEngine] Patching save file location" when it runs the
   patch.
4. Load Game: are your characters listed? Quit. Compare the save folder.

On our builds the line printed and the save stayed on the `.sl2` [live B45].

## Ways to separate saves

| Option | How | Cost to players | State |
|---|---|---|---|
| a. One file, keep modded characters apart by discipline | back up, move the file, swap back | a few file moves; a mistake is silent and lasting | works with no tool (recipe above) |
| b. A separate save file for the modded install | change the exe's save prefix to another 7-character string | a one-time edit per install | **built, checked offline, never run (B35, on hold)** |
| c. Mod Engine's alternate save | a `[savefile]` line | none | **tested, does nothing in DS2 (B41, B45)** |
| d. A launcher script that swaps the files around a session | rename the two `.sl2` files in and out | launch through the script | not built. A crash leaves the files swapped [inference] |
| e. Hex-edit instructions for players | the player edits the exe | fiddly | not chosen: no tool needed, but it asks the player to change the exe |
| f. A shared pre-lit save | copy a ready file | copy a file | rejected: one file holds all ten slots and could carry its author's account id |

The Mod Engine option would have cost players nothing, and it failed on the first live test.

### Before editing the exe for a separate save

- **It changes FromSoftware's exe.** A Mod Engine install changes no game file, and a clean install leaves the Steam
  folder's files equal to Steam's [live B49, B58]. A changed exe is a new class of risk for a player.
- **How it is delivered matters.** A readable script is rarely flagged but needs Python. A compiled program that edits
  another program's exe matches "patcher" heuristics for antivirus. Shipping a ready-patched exe is out [inference].
- **Steam would undo it.** "Verify files" or a game update puts the original exe back, and the modded install would
  then silently open the normal save [inference].
- **It would be the only exe change.** With Mod Engine as the loader the exe stays Steam's own. A save-name edit would
  be the one change to it.

## Pitfalls and what failed

1. **"Mod Engine moves the save by default" was wrong.** The 8 Nov reading of the dll's code said the option defaults to
   on and renames `.sl2` to `.sl3`. B41 run 1, a fresh character through Mod Engine, wrote `DS2SOFS0000.sl2` (08:04) and
   no `.sl3`. The code reading was wrong, or the patch finds nothing in DS2 [live B41].
2. **Switching the option on changed nothing.** `useAlternateSaveFile=1` with the debug console on: the console showed
   "[ModEngine] Patching save file location", yet no `.sl3` appeared and `DS2SOFS0000.sl2` was written (10:56:25)
   [live B45, 8 Nov]. The project's reading is that this is the patch Mod Engine has for DS3 and Sekiro, written for
   another game's code, and that DS2 builds its name another way [inference]. Of the dll's strings, the shadow and
   archive patches each have an "AOB scan failed" message and the save patch has none, so it may fail silently
   [data, inference]. We found no `sl2` or `sl3` string in the dll, so the "swap .sl2 for .sl3" reading comes from the
   other games, not from the dll's text [data].
3. **Only one dll build was tested.** The build tested (compiled 2019-08-12) is the one shipped inside a big
   overhaul's zip. The other build, 0.2.0a2 (compiled 2019-07-20), holds the same option and the same log line (see
   [The legacy DS2 Mod Engine](mod-engine-ds2.md)). It was never run with the option on. After the clean-install sessions,
   which used 0.2.0a2 with the default settings (B49, B57, B58), no `.sl3` is in the save folder [data, 17 Nov; weak
   evidence, as files might have been cleaned up].
4. **A modded install on a shared file is a silent mistake.** No warning appears, and the load-time groups run on any
   character [code].
5. **The exe route was built but never run.** B35 would have started a fresh file in the debug copy, checked that the
   original stayed untouched, and then checked that a copied slot loads from a renamed file (so the game does not check
   the name inside the save). None of that happened. State on 17 Nov: the debug copy's exe still holds the original
   prefix, no backup of the exe was made and no snapshot was taken [data].
6. **Restoring a backup exe undoes the edit.** The project's other exe tool, `tools/exepatch.py restore`, copies its
   whole backup exe over the game's, so it would put the prefix back [code]. That holds for any exe edit: re-apply it
   after every restore.
7. **Steam Cloud was never checked.** It is unknown whether Scholar syncs `DS2SOFS0000.sl2`. If it does, moving the
   file out and starting a fresh one could be undone by a sync, and backups could be overwritten [untested].

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| Every install on the account shares one `DS2SOFS0000.sl2` (ten slots) | the debug and Steam copies shared it from the start; the folder listing | project notes 7 Nov; data 17 Nov |
| The original game's file sits beside it under another prefix | folder listing; no `DARKSII` string in the Scholar exe | data 17 Nov [inference on origin] |
| The exe builds the name from `DS2SOFS`, `%s%04d` and `.sl2`, each read once | string scan and reference scan of the stock exe | code, 17 Nov |
| The debug copy's exe was never changed by B35 | prefix still `DS2SOFS`; no backup file; no snapshot file | data 17 Nov |
| Mod Engine's default does not move the save | a new character wrote `DS2SOFS0000.sl2`; no `.sl3` | live B41 run 1 (8 Nov) |
| `useAlternateSaveFile=1` also does nothing | console showed the patch line; the `.sl2` was written, no `.sl3` | live B45 (8 Nov) |
| Both dll builds carry the option and the log line | string scan of both dlls | data 17 Nov |
| The 0.2.0a2 dll build was never run with the option on | no test | untested |
| A Mod Engine install changes no game file | file-by-file check of a clean install: 0 problems, no game file changed | live B49, B58 |
| Steam's exe signature is Valid; the unpacked debug exe fails | `Get-AuthenticodeSignature` on Steam's exe (as hard-linked into the test copy) and on the debug copy's | data 17 Nov |
| A fresh file starts when the save is moved out | new character made on the fresh file | live B58 step 3 (16 Nov) |
| A mod's load-time groups write flags into the character | read from the build modules and the scripts; flags a load-time group wrote were read back from the running game | code, 17 Nov; live B42 read-back |
| A normal character loaded with a mod picks these up | not observed | untested |
| Kind-1 record at a missing event location: respawn goes to the map origin (void) | the 4 Nov death loop | live, 4 Nov session |
| Kind-0 record at a missing bonfire id | never tried | untested |
| Steam Cloud and `DS2SOFS0000.sl2` | never checked | untested |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/savefile.py` | A development tool. It finds the save-name prefix in an exe (refusing unless there is exactly one candidate after `SaveData`), reports it, and can rewrite it in place. The exe keeps its size, so the Cheat Engine table's and `pcmem`'s version checks still match. It keeps a backup once and checks its own result by reading back. `snapshot` and `compare` record the `.sl2` files (names, sizes, times, a short hash) and report what changed. Checked offline only on a copy: four bytes differ, a second apply changes nothing, and restore is byte-identical. **Nothing was applied to a game folder.** |
| `tools/modengine_test.py` | Builds the Mod Engine test copy; `ini save` and `ini plain` switch the `[savefile]` setting; `snapshot` and `compare` do the same file check for the save folder |

## Open questions

- **Does Steam Cloud sync `DS2SOFS0000.sl2`?** Needs a look at the game's Steam Cloud setting, then a fresh-file test
  [untested].
- **Does the exe route work?** B35's questions are all open: does the game start a fresh file under a different
  prefix, does it load a slot copied into the renamed file (is the name inside the save checked), and is the
  original left untouched? It is on hold, not shown to fail [untested].
- **Why does Mod Engine's patch do nothing in DS2?** The extension is a separate string appended with a fixed length.
  Whether the patch looks for something that is not there, or writes to a copy the game never reads, is unknown. The
  0.2.0a2 build was not tested with the option on [untested].
- **What does a normal character look like after one load of a modded map?** Snapshot the event flags of a copy of a
  normal save before and after, to turn the code-read examples into data [untested].
- **A save that last rested at a mod-only bonfire, loaded without the mod (kind 0).** Never tried [untested].
- **Does the game write `DS2SOFS0000.sl2.bak` on a schedule?** The folder had none on 17 November.
- **A launcher script that swaps files (option d)** would need a safe way to recover from a crash [untested idea].
- **Does swapping the files back after removing a mod work end to end?** Not run [untested].

## Related pages

- [Adding a new bonfire](../modding-world/adding-a-bonfire.md): mod-only bonfires in a save, and the uninstall risk.
- [Bonfire lit state, the travel menu and the save's bonfire list](../modding-world/bonfire-lit-state.md): the list the save holds, and why
  a pre-lit save and a save patcher were rejected.
- [The respawn point](../modding-world/respawn-point.md): the record a save carries, and the void loop at a missing row.
- [Warping by script](../modding-world/warping-by-script.md): the guard that sets a respawn record after a warp.
- [The legacy DS2 Mod Engine](mod-engine-ds2.md): the install route and its settings; this page covers only what it means
  for saves.
- [Bringing a boss back](../modding-world/respawning-bosses.md): the kill flags a load group can clear.
- [Map object states](map-object-states.md): the drained-pools example.
- [Event script groups](event-script-groups.md): how a group runs at every load.
- [Event flags and choosing safe ids](event-flags-and-free-ids.md): which flags are free, and why a released id cannot move.
