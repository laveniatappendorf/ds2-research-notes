# Diagnosing crashes from minidumps (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B27 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags are explained in the Evidence section.

## Summary

- **Three facts settle most crashes, and a minidump gives all three with no debugger:** the exception code, the place it
  happened (module and offset), and the chain of return addresses on the crashing thread's stack. Windows' Application
  log gives the first two even after the dump is gone. Reading a dump is a short job for a standard-library script: the
  method is described below, with no tool to install. [live B27, B33, B41]
- **The project met three kinds of crash in about two weeks** (25 Oct to 8 Nov, each one in the Windows log): a boot
  crash from an edited regulation file (B3), seven heap-corruption crashes while loading Lost Bastille and on a death
  reload (B27), and two crashes from a window text of 131 characters (B41). Each is worked through below.
  [data; live B3, B27, B41]
- **The heap-corruption crashes came from inside a community debug-menu add-on, DebugManager v2.0, loaded in our debug
  copy.** Five dumps show the same free-call chain, and no crash of that kind is logged since the add-on was moved
  aside on 6 Nov. The add-on is another modder's tool and is not part of the mod. We saw the crash in our own
  setups only (an unpacked copy with edited maps; once a packed install loading it through an ASI loader), so it says
  little about the add-on elsewhere. [live B27, B33; data]
- **The main trap: a heap-corruption stack shows where the damage was noticed, not who did it.** B27's first reading
  ("our files are innocent, so the game or the debug copy") was right about the files and then guessed at the game, the
  unpacked layout and a memory lead. Comparing five dumps of the same crash, and then removing the suspect, named the
  add-on. [live B27, B33]
- **The second trap: dumps disappear.** Windows kept the newest 10 and deleted the rest: the dumps of every crash on this
  page are gone, including the five from the heap-corruption crashes and the B41 dump. Copy a dump out the day it is written.
  The Application log kept every crash's code and offset. [data; inference for the 10-dump rule]
- **A dump narrows a crash; the exe decides it.** B41's dump named the exception type (`std::length_error`) and the
  thrower, and the project still blamed pixel widths until the exe was read ([worked there](../modding-menus/text-length-limits.md)).
  [live B41; code]

### The project's crash ledger

The crashes of the game exe that the project's tests met, and a few stray ones outside the arena's own copies (last
row), grouped by exception code, module and offset (the snippet is under How to do it). Offsets are RVAs in the Steam
1.0.3.0 exe, or in ntdll for the heap check. [data, read 17 Nov]

| Kind | Code | Where | Events | In which copy | What it was | Since |
|---|---|---|---|---|---|---|
| Boot crash, edited regulation (B3) | 0xc0000005 | exe+0x265b3e | 5, 28 Oct 19:15 to 19:25 | debug copy | the tag check rejected the edited file; a null pointer a moment later [live B3] | gone with the loose-params layout ([Editing params](editing-params.md)) |
| Heap corruption | 0xc0000374 | ntdll+0x118095 | 7: two on 25 Oct, then 28 Oct 20:58, 4 Nov 21:29, 5 Nov 11:48, 6 Nov 15:09, 6 Nov 18:35 | 25 Oct: the packed Steam install; the rest: the debug copy | DebugManager (the five in the debug copy were scanned) [live B27] | none after 6 Nov 18:35 |
| Window text too long (B41) | 0xc0000409 | exe+0xc39b1c | 2: 8 Nov 08:04 and 08:11 | Mod Engine test copy, then the debug copy | a 131-character window text [live B41; code] | none after the 8 Nov fix |
| Other | various | four different sites | 6 | the Steam install, a vanilla install | outside the arena's test copies: 24, 27 and 30 Oct, and three on 11 Nov from another tool's test | not followed up |

The "Mod Engine test copy" is a copy of the game on Steam's own packed files with the legacy Mod Engine's `dinput8.dll`
loaded. The "debug copy" is the project's unpacked copy, where DebugManager ran. See
[The legacy Mod Engine](mod-engine-ds2.md).

### Where the evidence comes from

| Source | What it gives | How long it lasts |
|---|---|---|
| Windows Application log, "Application Error", event 1000 | exception code, faulting module, fault offset, faulting exe path (which game copy), process id | the project's log still held the first game crash of 24 Oct, 24 days back |
| Windows Error Reporting local dump, `%LOCALAPPDATA%\CrashDumps\<exe>.<pid>.dmp` | the minidump. The event shows the process id in hex and the file name in decimal: 0x20C4 is 8388, the dump of the 6 Nov 18:35 crash | the newest 10 files only |
| The project's watcher, `tools/bastillewatch.py` | logs every Lost Bastille load, and counts a crash when a new dump appears after the game closes | `tools/bastille_loads.csv` |

Windows writes the dump a few seconds after the process ends, so a watcher has to wait. The project's watcher waits 8 s.
[code `tools/bastillewatch.py`]

The machine's LocalDumps key exists but holds no values, so it had no entry for Dark Souls II and the defaults applied: a minidump, newest 10 kept.
[data: registry, read 17 Nov; the defaults are Microsoft's documented ones, not looked up for this page] The dump folder
holds exactly 10 files now, and none is a dump of a crash on this page. [data] That is why those dumps are gone. [inference]

To make Windows write dumps for the game, and keep more of them: under
`HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps` add a subkey named `DarkSoulsII.exe` with
`DumpFolder` (a path), `DumpCount` (a DWORD, raise it) and `DumpType` (1 = mini, 2 = full). This is Microsoft's
documented setting. The LocalDumps key already existed on the project's machine, with no values in it, so we never created one. [untested]

### What a minidump holds

A minidump is a table of streams. These are the ones a reader needs:

| Stream type | Holds |
|---|---|
| 3 | thread list: each thread's id, its stack's address range, where its register context sits |
| 4 | module list: base address, size, timestamp and file name of every loaded module |
| 5 and 9 | the memory the dump kept (type 9 is the 64-bit form) |
| 6 | the exception: thread id, code, address, parameters, and the crashing thread's registers |
| 15 | misc info: process id and **process start time**, so start-to-dump is how long the game ran |

The x64 register context puts Rax at +0x78, Rcx +0x80, Rdx +0x88, Rbx +0x90, Rsp +0x98, Rbp +0xA0, Rsi +0xA8, Rdi
+0xB0, R8 to R15 from +0xB8 in steps of 8, and Rip at +0xF8. [data]

What the project's dumps contain, checked on game dumps still on disk on 17 Nov (about 14 MB each): thread stacks
(9 to 16 threads), the module list, the exe's writable data (two regions), and **128 bytes either side of the faulting
instruction**. They do **not** hold the exe's code or its read-only data. [data, 17 Nov] Two consequences:

- For an access violation, the bytes at the crash are inside the dump. You can read the failing instruction without
  the exe.
- Anything the exe keeps as constant data is not in the dump: the C++ throw info, class names, strings. Read those from
  the exe file, using the RVA (address minus module base).

### The exception code tells you where to look first

| Code | Name | What it means for us | First move |
|---|---|---|---|
| 0xc0000005 | access violation | parameter 0 is 0 for a read, 1 for a write, 8 for an execute; parameter 1 is the address | the registers, and the bytes at Rip: which register held the bad address, and where did it come from? |
| 0xc0000374 | heap corruption | ntdll's heap check found damaged heap data while freeing or allocating. The crash site is ntdll's fatal-failure report, the same offset every time (+0x118095 here) | the stack chain: the first frames that are not ntdll or the C runtime |
| 0xc0000409 | fast fail | the program asked Windows to kill it. `int 29h` with a code in `ecx`; code 7 is what the C runtime's `abort()` raises | the bytes at the crash, then look for a C++ exception on the stack |

The ntdll offset is only the report. It was the same in all seven heap crashes, and Windows also logged it for a
different game's exe on the same machine on 7 Nov. Never group heap crashes by that offset alone. [data]

### Reading a stack chain

Start at the crashing thread's stack pointer (Rsp) and walk up the stack in 8-byte slots. A slot whose value lies inside
a loaded module is a candidate return address. Print each one as `slot:module+offset`, where the slot number counts every
8-byte slot from Rsp. Three rules, all from the project's reads:

1. **Compare dumps of the same crash.** Frames that repeat at the same slots in every dump are the crash path. Frames
   that change from dump to dump are leftovers of calls that finished. The first reading of the Lost Bastille crashes
   took the nearest exe addresses as the path. They differed between the 4 Nov and 5 Nov dumps and were stale: one sat
   right after a `LeaveCriticalSection` call, one after a `RET`, one after a lock's unlock. [live B27; the findings]
2. **A return address follows a call.** An address that sits straight after a `RET` cannot be one.
3. **Name the owner of each frame by module.** One file can hide behind a common name: the add-on was loaded as
   `dinput8.dll`, and the legacy Mod Engine and Windows' own DirectInput also appear under that name. Tell them apart by
   size and timestamp (the add-on is 0x8f000 bytes, timestamp 0x68bc0ace; the Mod Engine builds are 0xf000 bytes, and
   Windows' own is a different size again). [data: the module lists, and the files, 17 Nov]

To name exe frames, open the RVAs in Ghidra: its address is the RVA plus the image base 0x140000000 (the project's names
are `FUN_14xxxxxxx`). The project used headless Ghidra to name the function at each stack address. [live B27]

### Naming a C++ exception

When the code is 0xc0000409, the crash address is an `int 29h` and `ecx` was loaded with 7 just before it (the exe has
that pair at +0xc39b17 and +0xc39b1c, read 17 Nov), the game's `abort()` ran. In B41 it ran because a C++ exception was
thrown and nothing caught it. The throw leaves a Windows exception record on the stack. Its code is 0xE06D7363, it has
four parameters, and **parameter 2 is the ThrowInfo address**, parameter 3 the image base, parameter 1 the exception
object. From the ThrowInfo you reach the class names:

1. ThrowInfo +12 holds an RVA to the catchable-type array.
2. The array is a count and then RVAs of catchable types.
3. Each catchable type has, at +4, an RVA of a type descriptor. The class name is the string at +16 in the descriptor,
   a mangled name such as `.?AVlength_error@std@@`.
4. For `std::exception` classes the object holds a pointer to the message at +8 (the `what()` text).

B41's ThrowInfo was at +0x15576f8. Read from the exe on 17 Nov, it lists `std::length_error`, `std::logic_error` and
`std::exception`. [live B41; code, checked 17 Nov] The walk-through from there to the 127-character limit is on
[Text that crashes the game](../modding-menus/text-length-limits.md), the worked example for this step.

### Who owns the memory

For a heap crash the useful question is who called `free`. The import tables help. The game exe imports no C runtime
DLL (its runtime is linked in). DebugManager imports the Universal C Runtime (`api-ms-win-crt-heap`) and the Visual C++
libraries (`MSVCP140`, `VCRUNTIME140`). [data, import tables read 17 Nov] So a `free` that reaches `ucrtbase` is not the
exe's own, and in B27 the frame above it sat in the add-on. [inference from the imports and the chain]

### The add-on, and what we are claiming

DebugManager v2.0 is a community debug-menu add-on for this game version: an F10 menu, hitbox display, enemy control and
a free camera. It loads as `dinput8.dll`, which the game imports, and its readme asks for an unpacked game. [data; the
notes, 25 Oct] The project ran it in its debug copy from 25 Oct to 6 Nov, and its menu and hitbox view worked there once
its font file was copied in. It is closed source, so we can say where the heap damage was found and not why. We saw
the crash only in our own setups, and the shipped mod does not include the add-on. [live; the notes]

### What happened

| When | What was happening | Files | Dump |
|---|---|---|---|
| 28 Oct 20:58 | the trainer's event warp, a third call (the Vammar start) | before any arena map edit | `.25804` |
| 4 Nov 21:29 | the first Lost Bastille load of that game session, 49 s after launch, no tools attached | arena | `.24208` |
| 5 Nov 11:48 | Lost Bastille load, travelling to the Saltfort bonfire | arena | `.14496` |
| 6 Nov 15:09 | Lost Bastille load, in the counted test | **original** m10_16 | `.30252` |
| 6 Nov 18:35 | a death at the Old Dragonslayer, before the warp to Majula | arena | `.8388` |

Each dump name is `DarkSoulsII.exe.<pid>.dmp`. All five are in the debug copy, and each one's module list shows the
add-on's file (0x8f000 bytes, timestamp 0x68bc0ace). The Windows log has the same five events, plus two from 25 Oct in
the packed Steam install, where the add-on was first tried through an ASI loader. That pair was never scanned. [live
B27; data, 17 Nov; the 25 Oct trial from the notes]

### The reading, step by step

1. **5 Nov: not the tools.** The write log had no write within a minute of either crash, and the 4 Nov crash had no
   tools attached at all. A fresh game with no tools, repeating the route, did not crash. [live B15; session notes]
2. **6 Nov 15:00 to 15:10, B27: not our files.** With the original m10_16 files (`tools/bastille_ab.py orig`: only that
   map's msb, event script, EventParam and GeneratorParam), the game crashed at the fifth load. The Windows log gave one
   signature every time: 0xc0000374, ntdll offset 0x118095, including 28 Oct, before any arena map edit. The counts, all
   taken with the add-on loaded: original files 6 loads and 1 crash (one start counted two arrivals); arena files 11
   loads and 2 crashes. [live B27; `tools/bastille_loads.csv`]
3. **What the test was designed to say.** The arena files crashed in 2 of 11 loads, so 12 loads with no crash at the
   original files would have been a 7 % chance (0.8 to the power 12) if our files were the cause. The test stopped at the
   first crash, which already cleared them. [live B27]
4. **The first conclusion stopped short.** B27 wrote "the debug copy or the game": the unpacked layout, or a memory-fix
   lead from a randomizer mod. "The debug copy" was right, and the add-on was in it. The game and the unpacked layout
   were not the cause. [live B27, then corrected]
5. **18:35, a fifth crash, and the chain.** Scanning the five dumps' crashing-thread stacks showed one chain, at the same
   slots in all five:

   | Slot | Frame |
   |---|---|
   | 0 to 51 | ntdll: the heap check that reports corruption (`RtlFreeHeap` into `RtlReportFatalFailure`, +0x118095) |
   | 73 and 81 | `ucrtbase+0x12d7b`, `free()` |
   | 87 | DebugManager +0x24765 (+0x24815 in two of the dumps): the code that called `free()` |
   | 95 | DebugManager +0x38e4 |
   | 129 | DebugManager +0x336c7 |
   | 131 to 159 | the game's per-frame update: `FUN_140b0bd10` from `FUN_140af0090` from `FUN_140af09b0` from `FUN_140aeeed0` from `FUN_1402eb630` (RVAs +0xb0bd10, +0xaf0090, +0xaf09b0, +0xaeeed0, +0x2eb630) |

   [live B27, from the notes; the dumps are no longer on disk. The four add-on offsets lie in the add-on's code section,
   checked on its file on 17 Nov.]
6. **What the chain says.** The block being freed when the damage was found was freed by the add-on's own code:
   `free()` through the Universal C Runtime, called from inside its hooks during the game's frame update. The damage may
   have been done earlier, by an overrun or a use after free in the add-on's data or by other code. The dumps show only
   where it was noticed. [inference; live B27]
7. **Why loads.** Bastille loads and death reloads are heavy frames: many files loaded, many objects made and freed.
   A hook bug in an overlay would show there. This is a guess. [inference]

### The test that settled it, and what the log adds

B33: rename the add-on's `dinput8.dll` to `dinput8.dll.off` in the debug copy (undo by renaming back; the F10 menu and
the scene-select boot screen go). The arena files do not need it. Done 6 Nov about 19:00. [live B33]

Tests from B34 on ran without it, and none reported a heap crash. [live B34 onwards] That is weak evidence, as nobody
counted the loads, so the Windows log is better: it holds **no event with code 0xc0000374 after 6 Nov 18:35:09**, in
the 11 days to 17 Nov, in any of the project's game copies. [data, 17 Nov] The counted A/B test that B27
ran with the add-on loaded was never repeated without it.

The 25 Oct pair adds a little. They happened in Steam's packed install, with a stock exe and no project file, with
the add-on loaded through an ASI loader. So the project's edits and exe patch were not needed for the crash. The add-on's
readme asks for an unpacked game, so that setup was outside what it documents. [data; notes, 25 Oct; inference]

## Worked example 2: the setup question that was too long (B41)

Run 1 (8 Nov, a Mod Engine test copy) and run 2 (the debug copy) ended the same way when the player said YES to the
setup's first question. The Windows log has both: 0xc0000409 at exe+0xc39b1c, at 08:04:23 and 08:11:36. [live B41; data]
Two different installs with one crash ruled out the install route and put the cause in the content. The dump gave the
exception type from its ThrowInfo, and the stack values that made sense were the string routine +0xefa30, then
+0x198326, +0x19882a, +0x463102 and +0xe23e64. The dump file itself is gone, so those are the notes' figures. [live
B41] A later read of the exe agrees: +0x198326 and +0x19882a are return addresses inside the window-text code. [code,
from the work on [Text that crashes the game](../modding-menus/text-length-limits.md)]

What the method got right: the type, the thrower, and "a string built with an impossible length while a window opened".
What it did not settle: why the length was impossible. The project measured line widths and blamed them. Reading the string
routine showed the real limit, 127 characters. That is the lesson of this example. [live B41; code] No 0xc0000409 event
has been logged since the 8 Nov fix, in any game copy. [data]

## Worked example 3: the boot crashes from an edited regulation (B3)

On 28 Oct the debug copy crashed on boot about 1 s after launch, five times in all (B3: three with a re-encrypted file, two
with a bare BND4). The Windows log has five events, all at exe+0x265b3e, between 19:15 and 19:25. [live B3; data] The
fault was an access violation on a null pointer. B3's entry records that the function at +0x275FC0 returns
`[GameManagerImp+0x22F0]` and that this value is null; the notes do not say how the trace was made. [live B3] Two of the
five left dumps, which the earlier notes filed under "the trainer days"; the shared address and the count tie all five to
B3. [inference] The cause and the cure are on [Editing params](editing-params.md).

## How to do it

A recipe for a new crash:

1. **Make the evidence last.** Confirm that Windows writes dumps for the game and raise the count (above). After a crash,
   copy the newest `DarkSoulsII.exe.<pid>.dmp` somewhere safe. Never share a dump: its module list holds full local
   folder paths, and its memory can hold anything. [data]
2. **Ask the Application log first.** Group the game's crashes by code, module and offset:

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

   One group is one crash. A new group after your change is a new crash, not the old one moved. [data: the ledger
   above came from this]
3. **Read the dump with a small script** (below): the exception record, the module list, the crashing thread's stack
   chain and, for a C++ exception, its ThrowInfo. Keep the game's exe beside it, because the dump lacks the exe's
   read-only data.
4. **Branch on the code** (table above):
   - 0xc0000005: print the registers. Which one holds the bad address? Decode the bytes before Rip (they are in the
     dump) to see where that register was loaded from, then find the owner of that pointer.
   - 0xc0000374: read the chain. Find the first frame that is neither ntdll nor the C runtime, and name its module.
   - 0xc0000409: check for `int 29h`, then look for the exception record and read its ThrowInfo.
5. **Get more than one dump.** Compare the chains. Keep only the frames that repeat.
6. **Name the module by size and timestamp**, not by its file name. Check its imports to learn which runtime it uses.
7. **Map exe frames in Ghidra** (RVA plus 0x140000000). Read the code at the frame the exception names.
8. **Test the suspect by removing it**, not by arguing. Change one thing (original files, the add-on renamed off,
   another copy of the game), and say before you start what each outcome would mean. For an intermittent crash, count
   loads. B27 logged each load automatically and wrote the odds down first. [live B27, B33]
9. **Write the evidence level down.** "Seen in five dumps", "the stack shows where it was noticed", "removed, no repeat in
   11 days": keep them apart, as above.

### Reading a dump with a script

The method is simple enough to re-implement with the standard library alone (Python's `struct` module is all it needs),
so no script is given here and nothing needs installing. The project did it with a throw-away reader of about 120
lines. Checked on 17 Nov against the dumps then on disk, and against the exe for the class-name walk. The C++-exception
step (3 below) has not run on a dump of a real C++ exception, because none survives. [data]

| Step | Where to read it (offsets in bytes) |
|---|---|
| Header | signature `MDMP` at 0; stream count at +8; RVA of the stream directory at +12; the dump's own time stamp at +20 |
| Stream directory | 12 bytes per entry: type, size, RVA |
| Exception (type 6) | thread id at +0; code at +8; address at +24; parameter count at +32; parameters (8 bytes each) from +40; RVA of the thread's context at +164 |
| Context | Rsp at +0x98 (the other registers are listed under "What a minidump holds") |
| Module list (type 4) | a count, then 108-byte records: base address (8 bytes), size, checksum, time stamp, RVA of the name (a length-prefixed UTF-16 string). Print file names only |
| Thread list (type 3) | a count, then 48-byte records: thread id at +0; stack start address (8 bytes) at +24, stack size at +32, RVA of the stack bytes at +36 |
| Memory (types 5 and 9) | address ranges and where their bytes sit in the file. Type 9 keeps its bytes packed one range after another from a base RVA; type 5 gives each range its own RVA |
| Misc info (type 15) | process id at +8; process start time (a Unix time) at +12 |

Then:

1. **Chain.** On the crashing thread, take the stack bytes from Rsp upward, read each 8-byte slot, and print the slots
   whose value lies inside a module as `slot:module+offset`.
2. **Session length.** The dump's time stamp minus the process start time is how long the game had run.
3. **C++ exception.** Scan the crashing thread's stack for the dword 0xE06D7363 with a parameter count of 4 at +0x18
   from it: that is the exception record. Its parameters sit from +0x20 (parameter 1 the object, 2 the ThrowInfo, 3 the
   image base). Walk the ThrowInfo as in "Naming a C++ exception". For `what()`, read the pointer at object +8.
4. **Read-only data.** When an address is in a section the dump did not keep, convert the RVA to a file offset with the
   exe's PE section table and read the exe file.
5. **Read the module list with care.** The list is long. Look for a second `dinput8.dll` or any module outside the game
   folder, and check its size and time stamp.

## Pitfalls and what failed

| Test | What happened | Result | Lesson |
|---|---|---|---|
| B27, 6 Nov 15:10 | The original files crashed too | "Not our files" was right. The next step, "the game or the debug copy: the unpack, a memory-fix lead", was only half right | Clearing your own files is not naming the culprit. Stop at "not ours" until something names the owner |
| 4 and 5 Nov dumps | Read one at a time, the frees and top frames differed from dump to dump | Taken to mean "the stack shows where the damage was noticed" (true), with no common cause found | The frames that differ between dumps are leftovers. Compare dumps before reasoning from one |
| B27, 6 Nov 18:35 | A fifth crash gave five dumps | One chain at the same slots, ending in the add-on's `free()` | The comparison found what the single reads could not |
| B33 | The add-on renamed off | No heap crash in the 11 days that followed | A removal test settles it, but only a counted rerun gives odds, and none was run |
| B41 runs 1 and 2 | The same crash in two installs, 7 minutes apart | The install route, Mod Engine and a fresh character were ruled out the same day | Change one variable, here the install, and the outcome is clear |
| B41 diagnosis | Width read as the cause | Wrong: the count was. The 8 Nov reword changed both | A fix proves a cure, not a cause. Read the exe ([Text that crashes the game](../modding-menus/text-length-limits.md)) |
| All dumps | Windows kept 10 and deleted the rest | The dumps of every crash on this page, B41's included, are gone | Copy a dump out at once. The Application log kept the codes and offsets |
| The ntdll offset | The same in every heap crash, and in a different game's crash | It is the report, not the cause | Never group by that offset alone |
| 28 Oct crash | The trainer's event warp was dropped as unsafe after it | The dump's chain, read on 6 Nov, showed the add-on, not the warp | A crash in the minute after your new feature is not the feature's until the stack says so |

Other things worth knowing:

- **Two things are called `dinput8.dll`.** The add-on, the legacy Mod Engine and an ASI loader each sit in that slot,
  because the exe imports DirectInput. The Mod Engine builds load Windows' own copy too, so a dump of those copies lists
  two modules with that name (0xf000 and 0x4b000 bytes). [data]
- **The scan names the code that noticed, not the code that did it.** In B27 that was the add-on's `free()`. The write
  that did the damage was never shown. [inference]

## Evidence

Tags: `[live Bxx]` seen in the game; `[code]` read in the exe; `[data]` read in the game files or the machine's logs;
`[inference]` reasoned; `[untested]` an idea only.

| Claim | How we know | Test or source |
|---|---|---|
| Windows' Application log (event 1000) keeps each game crash's code, faulting module and offset, back to the first crash on 24 Oct | data: the log | read 17 Nov |
| Five heap-corruption crashes in the debug copy, 28 Oct to 6 Nov, all ntdll+0x118095 | data (the log) and live | B27 |
| The five dumps share one chain ending in the add-on's `free()` | live (dump scan), from the notes; the dumps are gone | B27 |
| The add-on file is 0x8f000 bytes, timestamp 0x68bc0ace, and its four offsets are in its code section | data: the file, 17 Nov | B27 |
| The exe imports no C runtime DLL; the add-on imports the Universal C Runtime | data: import tables | read 17 Nov |
| Original m10_16 files crashed too: 6 loads and 1 crash against 11 and 2 with the arena files | live | B27; `tools/bastille_loads.csv` |
| No heap-corruption event in the log after 6 Nov 18:35:09 | data | read 17 Nov |
| The same code and ntdll offset were logged for a different game's exe on 7 Nov | data | read 17 Nov |
| The add-on renamed off from 6 Nov about 19:00; no heap crash reported since | live | B33, B34 on |
| Two heap crashes on 25 Oct in the packed Steam install, add-on loaded through an ASI loader | data (log) and the notes | 25 Oct notes; unscanned |
| 0xc0000409 at +0xc39b1c: the bytes are `mov ecx,7` then `int 29h` | code | exe, 17 Nov |
| B41's ThrowInfo (+0x15576f8) lists `length_error`, `logic_error`, `exception` | code, and the notes' figure | exe, 17 Nov; B41 |
| B41's two crashes: 8 Nov 08:04:23 and 08:11:36; no 0xc0000409 since | data | log, 17 Nov |
| Five B3 boot crashes at +0x265b3e on 28 Oct 19:15 to 19:25 | data | log; B3 |
| A minidump of this game holds the stacks, module list, exe data and 128 bytes around the fault; not the exe's read-only data | data | the project's game dumps on disk, 17 Nov |
| The dump's streams (header, exception, modules, threads, memory, misc info) can be read with the standard library alone | data | a throw-away reader run on the dumps on disk, 17 Nov |
| Windows keeps only 10 dumps, which is why older ones are gone | inference | folder has exactly 10; the setting's defaults |

## Tools

| Tool | What it does for this topic |
|---|---|
| `tools/bastillewatch.py` | Read-only. Watches the game process, logs each Lost Bastille load as `ok` (the map's flag block loaded and a character in the world for 15 s) or `crash` (the process ended and a new `DarkSoulsII*.dmp` appeared, after an 8 s wait) |
| `tools/bastille_ab.py` | Switches the four Lost Bastille map files between the original and the mod's (`orig`, `mod`, `status`), logs a load with `log`, and `tally` counts per variant. A model for an A/B crash count |
| `tools/bastille_loads.csv` | The log those two write: 17 rows from 4 to 6 Nov |
| A minidump reader (not a project tool) | The method under "Reading a dump with a script": a short standard-library script, re-implemented as needed. The project's own was a throw-away kept outside `tools/` |
| Headless Ghidra (not a project tool) | Names the function at each exe stack address; image base 0x140000000 |

## Open questions

- **Which write corrupted the heap?** The dumps never showed it. The standard tool is Windows' page heap (the `gflags`
  setting) or Application Verifier on the debug copy with the add-on loaded. We did not try it. [untested]
- **Is the add-on's crash tied to our arrangement?** Test the add-on in an unpacked copy with no project file and no exe
  patch. Its author is the right person to ask. [untested]
- **Do the four add-on offsets name functions?** They are in its code section; the add-on is closed source and we did
  not analyse it. [untested]
- **The C++-exception step of a dump reader** (step 3 under "Reading a dump with a script") has not been run on a dump
  of a real C++ exception, only checked in parts: the ThrowInfo walk against the exe's own ThrowInfo, and the stream
  reading on the dumps on disk. [untested]

## Related pages

- [Text that crashes the game, and missing glyphs](../modding-menus/text-length-limits.md): the worked example for the ThrowInfo step
- [Editing params: the encrypted regulation vs loose params](editing-params.md): the boot crash at exe+0x265b3e
- [Moving a bonfire](../modding-world/moving-a-bonfire.md): the Lost Bastille crash that looked like a bad map edit
- [Warping the player by script](../modding-world/warping-by-script.md): the 28 Oct crash that followed a trainer warp
- [The legacy Mod Engine](mod-engine-ds2.md): the Mod Engine test copy that B41's first run used
- [Live-testing tricks and pitfalls](live-testing-notes.md): counted A/B tests, watchers and logs
