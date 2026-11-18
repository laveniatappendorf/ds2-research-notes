# The legacy DS2 Mod Engine (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- **What it is.** The 2019 community loader for the PC Scholar of the First Sin: a 40,960-byte x64 `dinput8.dll` that
  sits in the game's `Game` folder, reads `modengine.ini`, and makes the game open files from an **override folder**
  before its own copies [data]. It is not Mod Engine 2 or me3, which have no DS2 support [web, 5 Nov 2026].
- **The install is three items** in `Game`: the override folder (ours is `BossArena`), `modengine.ini` and `dinput8.dll`.
  No unpacking, no exe patch, no game file overwritten. Deleting the three items removes everything [live B49, B58].
- **It serves what the Boss Arena ships through it**: the trimmed regulation, the loose params, the event and talk
  scripts and the texts [live B41, B45, B49]. Maps and the two new bonfire pictures use the same folder, but no run
  looked at them on their own (see "What it serves").
- **Two builds of the dll are in circulation, and both worked.** The 12 August 2019 build that two large overhauls
  bundle carried B41 and B45. The 20 July 2019 build (0.2.0a2) that the mod's README links carried B49 and B58. Same
  size, same ini keys [data, live].
- **`blockNetworkAccess=1` is how the mod keeps players offline.** Every live run also used Steam's Offline mode, so the
  setting was never watched on its own [untested].
- **Two traps.** Its alternate-save setting does nothing in DS2: saves stay in `DS2SOFS0000.sl2` [live B41, B45]; see
  [One save file for every install](saves-and-installs.md). And it takes the game's `dinput8.dll` slot, so a second loader
  has to go through the ini's chain-load key [live L2].

### Getting loaded

The exe imports `DINPUT8.dll` and `DirectInput8Create` [data, exe import names, read 17 Nov 2026]. Windows looks in the
game's folder before the system folder, so a `dinput8.dll` there loads first [inference: the usual search order; every
run behaved that way]. Mod Engine's copy exports `DirectInput8Create` and hands the real work to the system's
`dinput8.dll` [data: its log line "Chain load failed. Falling back to system dinput8.dll"]. It reads `.\modengine.ini`,
a path relative to the working directory, so the game has to start from the `Game` folder, as Steam does it [inference
from the relative path and the `GetCurrentDirectoryW` import].

### What the dll does

Read from the dll's import names and log strings, 17 Nov 2026. We did not read its code, so the right-hand column is
[inference] unless a test is named.

| Log string or name in the dll | What it points to |
|---|---|
| "Hooking CreateFileW", "[FileHook] Intercepted game file path ...", "Overriding with ..." | files the game opens by path (the regulation, loose params, scripts) are redirected to the override folder when a copy exists there [live B45: the console printed "Overriding with" for each] |
| "AOB scan hooking archive function", "[ArchiveHook] Looking for override file ...", "Adding override for ..." | files inside the game's `.bdt` archives are redirected the same way. The dll finds the exe's archive function by a byte-pattern scan, and has a failure line for when it cannot |
| "Blocking winsocket startup"; `WSAStartup` named in both the dll and the exe | `blockNetworkAccess`: the network start-up call is blocked |
| `SteamAPI_Init` named in both | hooked, per a code reading on 10 Nov 2026 [code, not run]; what for is untested |
| "Patching save file location" | the alternate-save patch. It logs in DS2 but changes nothing there [live B45] |
| "Patching directional / dynamic shadow resolution ..." | the `[rendering]` keys. Not used by the mod |
| "Attempting to chain load DLL ...", "Chain load successful" | `chainDInput8DLLPath` [live L2] |
| `.\modengine.ini` read with `GetPrivateProfileInt` / `GetPrivateProfileString` | the ini |
| `AllocConsole` | `showDebugLog=1` opens a console window [live B45] |

### The ini

The ini the mod ships (comments left out) [data, v1.01 package]:

```
[misc]
skipLogos=1
chainDInput8DLLPath=""

[online]
blockNetworkAccess=1

[files]
loadUXMFiles=0
useModOverrideDirectory=1
modOverrideDirectory="\BossArena"
cacheFilePaths=1

[debug]
showDebugLog=0
```

All the keys either build reads, with what the ini's own comments say (paraphrased) [data, the two overhauls' inis and
the 0.2.0a2 zip's sample ini, read 17 Nov 2026]:

| Key | Ours | What it does | Tested |
|---|---|---|---|
| `[misc] skipLogos` | 1 | Skips the boot logos. B41 listed "no logos" as the sign that the loader was running | Effect not recorded [untested] |
| `[misc] chainDInput8DLLPath` | `""` | Loads a second dinput8-style DLL. The path is the game folder plus this string, so it needs the leading backslash, and the DLL must export `DirectInput8Create`. The ini's own example renames another `dinput8.dll` and points here | Works [live L2, 11 Nov 2026, 12 August build] |
| `[online] blockNetworkAccess` | 1 | Blocks all network access. The comment says the injection is scanned for by the anti-cheat, so turn it off only at your own risk | Never seen alone [untested] |
| `[savefile] useAlternateSaveFile` | not set | Should move the save to another file | Does nothing in DS2 [live B41, B45] |
| `[files] loadUXMFiles` | 0 | Reads a complete UXM extraction instead of the archives | Not used [untested] |
| `[files] loadLooseParams` | not set | Loads loose param data instead of from the encrypted archive. Present in both builds, and in the 0.2.0a2 sample ini only | Effect in DS2 unknown [untested]. Our loose params load without it [live B49] |
| `[files] useModOverrideDirectory` | 1 | Turns the override folder on. At 0 the loader still runs but serves nothing, which the project used to get a vanilla copy [inference, 14 Nov 2026 note] | On [live B45, B49] |
| `[files] modOverrideDirectory` | `"\BossArena"` | The override folder, relative to the game folder, with a leading backslash. The sample ini's default is `\mods` | [live B45, B49] |
| `[files] cacheFilePaths` | 1 | Caches the file lookups. Adding or removing a file in the folder may need a game restart | See "How to do it", step 6 |
| `[debug] showDebugLog` | 0 | 1 opens a console that lists every served file | [live B45] |
| `[debug] breakOnStart`, `[gameplay] restoreBonfireSacrifice`, `[rendering]` four shadow-map sizes | not set | In the 0.2.0a2 sample ini only | Not used [untested] |

The sample ini in the 0.2.0a2 zip has no `[misc]` section, and its override folder is `\mods`. Both builds still read
`skipLogos` and `chainDInput8DLLPath`, because the strings are in both dlls. Copying the sample ini unchanged would
leave the loader looking for a folder the mod does not ship [inference]. That is why the mod's README says to use the
ini from the mod's zip.

### What it serves

Override-folder paths mirror the game folder. The v1.01 package holds 379 files [data, read 17 Nov 2026]. The first test
package (B41, 8 Nov) held 356.

| Path in the override folder | What | v1.01 files | Seen working through Mod Engine |
|---|---|---|---|
| `enc_regulation.bnd.dcx` | the regulation without params, a plain DCX | 1 | Yes: console line [live B45]; vendors stocked and free on a clean install [live B49] |
| `Param\*.param` | every other regulation param loose, plus the per-map params the mod edits | 273 | Yes: every loose param logged [live B45]; shop, cost and fight-list rows [live B49] |
| `ezstate\*.esd` | event scripts of 22 boss maps, Majula's event and talk script | 24 | Yes: the Herald's setup asked [live B41 run 1]; logged [live B45]; fights, rush and retry [live B49, B50c] |
| `menu\Text\<language>\*.fmg` | `bonfirename`, `mapevent`, `npcmenu`, `pluralselect`, `titlemenu` | 59 | English only: four files logged [live B45]; the title line "Arena 1.01" shown [live B58] |
| `map\*.msb` | arena maps with moved or new bonfires | 20 | **Not seen on its own.** B41 step 4 planned the check; no result is recorded [inference: same hook] |
| `menu\tex\icon\bonfire_list\*.tpf` | the two new travel-menu pictures (v1.01) | 2 | **Not seen.** Nobody has looked at the pictures in any copy [untested] |

The mod's regulation is a plain DCX of 21,675 bytes with only 12 `.fmg` and 11 `SpEffect*.emevd` inside. The two large
overhauls ship the same form: a DCX of 22,588 bytes (Bearer of the Curse Edition, Nexus DS2 1328, v1.5.1) and 23,312
bytes (Seeker of Fire, Nexus DS2 978, 2.1.5 zip), beside 574 and 575 loose params [data, zip listings read 17 Nov 2026,
nothing extracted]. See [Editing params](editing-params.md) for why that form works.

Those two packages also list models, textures, maps, filters, event scripts and sound in the same override folder (27
and 28 `.msb`, 7 and 124 `.tpf`, 3 and 30 `.esd`, and 243 / 343 model files) [data]. That shows what the community puts
there. It is not a test of ours.

### The two builds of the dll

Both are 40,960-byte x64 DLLs [data, read 17 Nov 2026].

| | 12 August 2019 build | 20 July 2019 build (0.2.0a2) |
|---|---|---|
| Where from | bundled by Bearer of the Curse Edition (Nexus DS2 1328) and Seeker of Fire (Nexus DS2 978): the same bytes in both zips | the `ModEngine-DS2-0.2.0a2.zip` release (23 KB) of a community-kept GitHub repository, "ModEngine-DS2-Compatible", which the Souls Modding Wiki's tools page points to [web, 8 Nov 2026] |
| PE timestamp (UTC) | 2019-08-12 01:22:23 | 2019-07-20 05:42:40 |
| `.text` / `.rdata` size | 23,356 / 12,548 bytes | 23,148 / 12,356 bytes |
| Ini keys read | the same set | the same set |
| Log strings | two extra "[Allocator] ..." lines | none |
| Sample ini in its zip | Bearer's: `[misc]`, `[online]`, `[files]`, `[debug]` | its own: adds `[savefile]`, `loadLooseParams`, `breakOnStart`, `[gameplay]`, `[rendering]`; no `[misc]` |
| Our runs | B41, B45, L2 (the test copy holds it) | B49, B47, B50c, B58 (the clean-install copy) |
| Result | served every file we looked at | the same |

- SHA-256 of the 12 August dll: `707539a9a11fcdc1b95483a63582b1e46827b9d66550a3fdb2580e54b439a8b5`.
- SHA-256 of the 20 July dll: `54d63471b97061809666fe3817932a743a698034302d0d99c6563e3a2c9d096a`. Its zip:
  `92db38d78fa8d76ca7ad80a64a96681537dabe2c5594999e616efbe02aab54d7`.
- A byte-for-byte compare of the two files differs in 26,626 of 40,960 bytes. That overstates the change: a rebuild
  moves code, and the section sizes differ by about 200 bytes [inference]. We did not find what the extra allocator
  lines do.

The clean-install test was first run with the 12 August build, because that was the dll in the overhauls' zips. The
README links the 20 July build, so the compare on 8 Nov made B49 use it instead: the test has to use the file a player
gets [live B49].

### Licence and where to get it

- The ini header of every copy we saw carries a 2019 copyright line marked all rights reserved. Neither overhaul zip
  holds a licence or readme file, and the 0.2.0a2 zip holds only the dll and the ini [data, read 17 Nov 2026].
- Its original author, Katalash, published the DS2 source on GitHub but never released a DS2 build, and has no Nexus
  page for it.
  Nexus DS2 1512 ("Dark Souls 2 mod tools") is a third party's re-upload [web, 8 Nov 2026]. The 0.2.0a2 release is
  described as unofficial, and its file was unchanged when checked on 16 Nov 2026 [web].
- The overhauls bundle the dll and credit its author. The Boss Arena **links** to it instead: updates stay with the
  source, and no file of unstated licence is passed on [design choice, 8 Nov 2026].
- **Credit.** The DS2 version of Mod Engine is Katalash's work. DebugManager, the debug-menu add-on that several pages
  mention, is LordRadai's.

### Who else ships through it

Seeker of Fire (Nexus DS2 978), Bearer of the Curse Edition (Nexus DS2 1328), Vestiges, SOTFS+, InfiniDetail (its "Easy"
install) and the Enemy + Item Randomizer (Nexus DS2 1317: drop `dinput8.dll`, `modengine.ini`, `ds2s_memfix.dll` and its
folder into `Game`; delete them to uninstall) [web, 5 Nov 2026]. Reports for big overhauls are mixed: one overhaul's
install guide recommends UXM unpacking instead, and warns that Mod Engine can leave files missing [web, 5 Nov 2026]. We
saw no missing file in a 379-file package, which is far smaller than an overhaul.

### `blockNetworkAccess`

- **What the author's comment says:** the setting hooks networking so the game never connects, and as long as it is on
  you will not be banned for playing mods. The injection itself is scanned for by the anti-cheat, so leave it on.
- **What the dll shows:** the log line "Blocking winsocket startup" and the `WSAStartup` name, which the exe also holds
  [data]. The network start-up call is the likely target [inference].
- **What we tested:** nothing on its own. Every live run used Steam's Offline mode as well, and B41 expected a "cannot
  connect" notice but none was recorded [untested]. The mod's ini says "offline" in a comment, and its README says in
  capitals not to play online.
- **Other mods:** both overhauls' current zips ship `blockNetworkAccess=1` [data, 17 Nov 2026]. Their own pages show it
  was not always so: Bearer's 1.5.1 changelog fixes the default to offline, and Seeker of Fire's FAQ tells players to set
  it to 1 [web, 8 Nov 2026]. One user report says the setting keeps a CPU core busy, and Seamless Co-op needs it at 0
  [web, 5 Nov 2026].

### Saves, in brief

The ini has `[savefile] useAlternateSaveFile`. Set to 1 with the debug console on, the console printed "Patching save file
location" and DS2 still wrote `DS2SOFS0000.sl2`, with no `.sl3` [live B45, 8 Nov 2026, 10:56]. With the key absent the
save was also still the `.sl2` [live B41 run 1]. The ini's chain-load example puts the other DLL in a Sekiro game
folder, so the code comes from a sibling game's loader [data]. The patch probably swaps `sl2` for `sl3` in memory and
finds nothing to swap in DS2 [inference]. So a Boss Arena install shares the vanilla save file. What that does to a
character, and the ways round it, are on [One save file for every install](saves-and-installs.md).

## How to do it

How the Boss Arena ships through it, as a recipe for another mod:

1. **Choose the dll build** (either worked) and decide whether to link or bundle it. Link it if you cannot show a licence.
2. **Lay out the override folder** to mirror the game folder: `enc_regulation.bnd.dcx`, `Param\`, `ezstate\`, `map\`,
   `menu\Text\<language>\`. Ship only files you changed, except that the loose-params layout needs the whole `Param`
   set ([Editing params](editing-params.md)).
3. **Write `modengine.ini`** as in the block above, with your folder name and a leading backslash. Ship your own ini
   and tell players to use it, not the sample that comes with the dll.
4. **Install by hand, as a player would:** the folder, the ini and `dinput8.dll` next to `DarkSoulsII.exe`. Put no other
   `dinput8.dll` there. A second loader goes through `chainDInput8DLLPath` [live L2].
5. **Check it loaded.** Two ways:
   - Give the mod a mark players can see. The Boss Arena relabels the title menu's version line to "Arena 1.01 Ver. 1.03"
     [live B48, B58]. The README tells players to look for it.
   - Set `showDebugLog=1` and watch the console: each served file prints "Overriding with <path>" [live B45]. Set it back
     to 0 before shipping.
6. **Update with the game closed.** Delete the old override folder, put the new one and the new ini in, and leave the
   dll [live B58, following the README]. Replacing the whole folder avoids a stale `cacheFilePaths` list, which the ini
   warns about. The project's refresh command does exactly this [code].
7. **Uninstall** by deleting the three items. A file compare then found nothing left over and no game file changed
   [live B58]. Saves are not touched either way.
8. **Test as a player has it, in a separate copy of the game folder.** The project's two copy tools:
   - Hard-link every file over 10 MB (the archives and the exe: read-only, no extra space) and copy the rest.
   - Leave out anything later added to the Steam folder: another `dinput8.dll`, DebugManager and its font, `.asi`
     plug-ins. The clean copy keeps only files dated no later than the game's install, plus the game's own
     `userconfig.properties` [code].
   - Add `steam_appid.txt`, so the copy starts without Steam relaunching the main install. It is the one thing a
     player's folder lacks [code].
   - Run Steam in Offline mode, and close any other Dark Souls II first (all copies share one save folder) [live B49
     procedure].

   The debug copy (a UXM unpack with a debug plug-in) is not a safe stand-in. A plain-DCX regulation with params was
   ignored there and was never retried on a clean copy [live B3, untested]; see [Editing params](editing-params.md).

## Pitfalls and what failed

| Test | What happened | Why / lesson |
|---|---|---|
| B41 run 1 (8 Nov) | A new character was made and the setup asked. Afterwards `DS2SOFS0000.sl2` had changed and no `.sl3` existed | The 8 Nov reading of the dll ("default 1, so the save moves to `.sl3`") was wrong. A setting that exists in a loader's code does not prove it works in another game |
| B45 (8 Nov, 10:56) | `useAlternateSaveFile=1` and the console on: "Patching save file location" appeared, still no `.sl3` | The patch comes from a sibling game's loader. Mod Engine cannot separate DS2 saves |
| B41 runs 1 and 2 (8 Nov) | The game crashed with no error window after YES to the setup's first question | **Not the loader.** The same crash on the same character in the debug copy cleared Mod Engine the same day. The cause was a text, found in a minidump: [Text length limits](../modding-menus/text-length-limits.md), [Crash diagnosis](crash-diagnosis.md) |
| B41 step 4 | The plan expected a moved bonfire in an arena as the sign that map files are served | No result recorded, so maps are tagged [inference] above |
| B49 set-up (8 Nov) | The dll tested so far was not the build the README links | Compared: same size and keys, different build. B49 ran with the linked one and passed |
| 14 Nov | The loader copy was reused as a vanilla game by setting `useModOverrideDirectory=0` | Handy, but that copy is no longer a route test as it stands [inference from the ini] |
| Any | Two `dinput8.dll` users cannot share the slot | The Mod Engine copies leave out DebugManager's. Whether DebugManager chain-loads is untested |
| Any | Copying the sample ini unchanged | Its override folder is `\mods`, so nothing of ours would be served [inference] |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| The override folder serves the trimmed regulation and every loose param | console lines "Overriding with ...\param\..." | [live B45] |
| It serves Majula's event, talk and `EventParam` and four English text files | the same console | [live B45] |
| The setup question is asked, so the talk script and `mapevent` texts are served | the Herald asked on the first run | [live B41 run 1] |
| A clean Steam copy plus the zip by the README gives vendors, fights and the rush | played by the README with the 20 July build | [live B49] |
| Install and uninstall leave a clean folder | file compare: all files equal the zip, no extras, no game file changed | [live B58] |
| The alternate-save key does nothing in DS2 | no `.sl3`, `.sl2` written, with the key absent and set | [live B41, B45] |
| A second DLL loads through `chainDInput8DLLPath="\name.dll"` beside the Boss Arena | a probe DLL's start-up log; arena unchanged | [live L2, 11 Nov 2026] |
| The 12 August and 20 July builds read the same keys and differ in size by ~200 bytes of code | PE headers and strings compared | [data, 17 Nov 2026] |
| Both overhauls ship the 12 August dll, the ini with `blockNetworkAccess=1`, and a plain-DCX regulation | zip listings read in memory | [data, 17 Nov 2026] |
| The exe holds `DINPUT8.dll`, `DirectInput8Create`, `WSAStartup` and `SteamAPI_Init` | import names in the exe | [data, 17 Nov 2026] |
| `blockNetworkAccess=1` keeps the game offline | the log line and the author's comment only | [inference] |
| Maps and the new pictures are served | same hook as the rest, never watched | [inference] |
| The original author never released a DS2 build; Nexus 1512 is a re-upload | project notes from the public pages | [web, 8 Nov 2026] |

## Tools

All in `tools/`. Both hold this project's folder paths in constants, so copy the commands, not the paths.

| Tool | For this topic |
|---|---|
| `modengine_test.py` | The route-B test copy: `package` (collect the changed files into the override-folder layout, with a SHA-1 manifest), `setup` (hard-linked copy of the game folder plus the dll and ini), `refresh` (replace the override folder, game closed), `ini save` / `ini plain` (the alternate-save test ini and the normal one), `snapshot` / `compare` (the save folder before and after), `status`. Its `INI` constant is the tested ini |
| `cleantest.py` | The clean-install copy: `setup` (the game folder as installed, plus the release zip and the linked Mod Engine zip), `check` (the install equals the zip, the dll is the linked build, nothing extra, no game file changed size), `update` (the README's updating steps, then `check`), `status` |
| `release.py` | Builds the zip. It writes `modengine.ini` from the tested constant under the mod's own header, and does not bundle the dll |

## Open questions

- **Does `blockNetworkAccess=1` block the network?** Never tested on its own. A test needs Steam online, so it risks an
  account; use a throw-away one, with a connection monitor.
- **Are map files (`.msb`) and the new bonfire pictures served?** One arena visit in a clean copy, looking at a moved
  bonfire and the travel-menu picture, would settle both.
- **Do the other ten languages' texts load through it?** Only English was logged.
- **What does `loadLooseParams` do in DS2?** If it makes loose params load without the plain-regulation trick, it
  bears on [Editing params](editing-params.md).
- **Does `skipLogos` work in the 20 July build?** B41 expected "no logos" and recorded nothing.
- **Can DebugManager load through `chainDInput8DLLPath`?** The ini's own example suggests it should.
- **What do the extra allocator lines in the 12 August build do, and does either build behave differently under load?**
- **Will a game update break the byte-pattern scans?** The game's Steam build has not changed since October 2022 [web,
  16 Nov 2026], so this was never exercised.
- **Why does the save patch find nothing in DS2?** The exe builds the save name from one string (`DS2SOFS`) [code, B35,
  7 Nov 2026]. Nobody has looked at what the patch searches for.

## Related pages

- [Editing params](editing-params.md): the loose-params layout this loader serves, and why a plain regulation works here.
- [One save file for every install](saves-and-installs.md): what the shared save means, and the options.
- [Diagnosing crashes from minidumps](crash-diagnosis.md): how the B41 crash was traced.
- [Text length limits](../modding-menus/text-length-limits.md): the real cause of the B41 crash.
- [Adding a new bonfire](../modding-world/adding-a-bonfire.md) and [Moving a bonfire](../modding-world/moving-a-bonfire.md): the map files and pictures this
  loader serves.
- [Live-testing tricks](live-testing-notes.md): running the game copies.
