# Writing DS2 ESD files (PC Scholar 1.0.3.0, 2026-11-17)

!!! success "Status: Confirmed live"
    Seen working in the game or emulator, not only read from files. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.
> Test ids such as B7 point to the project's test log (see [how to read this site](../how-to-read.md#the-modding-pages)). Evidence tags: `[live B7]` seen in the game,
> `[code]` read in the exe, `[data]` read in the game files, `[inference]` reasoned, not proven, `[untested]` an idea only.

## Summary

- **Event, talk and AI scripts are `.esd` files (ezstate), and you can write them.** The project's `tools/esdwrite.py` reads a
  PC Scholar script, lets you edit its states, conditions and commands as Python objects, and writes the file back. Unchanged,
  all 538 scripts of the game (28 event, 25 talk, 485 AI) come back **byte for byte**. [data]
- **A loose script is read, and a script whose layout changed loads.** A one-literal swap in a talk script (B4), two commands
  added to an event state so that everything after them moved (B7), a whole new state group (B9) and a talk script with
  new states (B20d) all loaded and ran. [live B4, B7, B9, B20d] The shipped mod's 24 scripts (23 event, 1 talk) are all written
  this way. [data]
- **Scripts are re-read at the next load of their map, with no game restart.** The `.edd` debug file beside a script does not
  have to match it: ours never did. [live B7, B9, B20d]
- **The recipe** is: read, edit through the model, write, then check three things: the new file writes out again byte for
  byte, `tools/esd.py`'s semantic diff shows only your change, and the record counts moved by what you expect.
- **The main traps:** conditions are *shared* between states (edit a new one, never in place); a command object may be used
  once only; a new state runs only if a condition targets it; and a new group also needs an `EventParam` row (see
  [Event script groups](event-script-groups.md)). A test that passes the writer is not a test of the feature: our first menu
  edit (E2) proved the writer and nothing else.

### 1. What the files are

| Magic | Platform | Byte order | Offset / count width | Seen |
|---|---|---|---|---|
| `fsSL` | PC Scholar of the First Sin (1.0.3.0) | little-endian | 64-bit | all 538 scripts of this build [data] |
| `fSSL` | PC, original 2014 game | little-endian | 32-bit | one stray file in the PS3 network-test builds [data] |
| `FSSL` | PS3 builds | big-endian | 32-bit | PS3 builds [data] |

The writer handles all three; only `fsSL` has ever been loaded in a game. The files are `event_<map>.esd` (one per map
that has events, 28 in this build), `talk_<map>.esd` (25) and `ai<id>.esd` (485, one per enemy AI). Each event and talk script
has a `.edd` beside it (53 `.edd` files in Steam's archives; the AI scripts have none). [data]
The `name` field inside an event or talk script is its own file name; inside an AI script it is the file name plus `_` and the
developer's label. [data]

Every record is a row of fixed-size words, and every offset is **relative to the data start, file offset 0x6C**. `-1` means
"none". On PC a word is a signed 64-bit value, except the two 32-bit numbers (bank and id) at the start of a command call.

**File header (0x6C bytes):** the magic, then 26 signed 32-bit words (byte order of the file). [data]

| Word (file offset) | Value |
|---|---|
| 0 (+0x04) | 1 |
| 1, 2 (+0x08, +0x0C) | 2, 2 (`dscount` in the writer; 2 in every DS2 file) |
| 3 (+0x10) | 0x54 (= 0x6C - 0x18 [inference]) |
| 4 (+0x14) | size of the data part |
| 5 (+0x18) | 6 |
| 6 (+0x1C) | size of the data header: 0x48 (PC) or 0x2C |
| 7 (+0x20) | 1 |
| 8, 9 (+0x24, +0x28) | state-group record size, **number of state groups** |
| 10, 11 (+0x2C, +0x30) | state record size, **number of state records** (the dummy copies count, see below) |
| 12, 13 (+0x34, +0x38) | condition record size, **number of conditions** |
| 14, 15 (+0x3C, +0x40) | command-call record size, **number of command calls** |
| 16, 17 (+0x44, +0x48) | command-argument record size, **number of arguments** |
| 18, 19 (+0x4C, +0x50) | offset of the condition-offset lists, number of words in them |
| 20, 21 (+0x54, +0x58) | offset of the name block, name length in UTF-16 units including the terminator |
| 22 (+0x5C) | size of the data part again |
| 23 (+0x60) | 0 |
| 24 (+0x64) | size of the data part again |
| 25 (+0x68) | 0 |

`esd.py` documents the count words by file offset (+0x28 groups, +0x30 states, +0x38 conditions, +0x40 commands, +0x48
arguments); they are the same words. The parser in `tools/esdwrite.py` refuses a file whose fixed words differ from these.

**Record sizes** (data header, state group, state, condition, command call, argument): **PC `fsSL` 0x48, 0x20, 0x48, 0x38,
0x18, 0x10**; 32-bit files 0x2C, 0x10, 0x24, 0x1C, 0x10, 0x08. [data]

**Data header.** A 32-bit word (1 on PC, 0 on PS3), then 16 bytes of unknown purpose (below); on PC a zero 32-bit pad at
+0x14; then six words at +0x18 (PC) or +0x14 (32-bit): state-group table offset, group count, name offset, name length, -1, -1.

**Records** (each field one word unless noted): [data]

| Record | Fields, in order |
|---|---|
| State group | id; states offset; state count; states offset again |
| State | id; condition-offset-list offset, count; entry commands offset, count; exit commands offset, count; while (`during`) commands offset, count |
| Condition | target state offset (-1 = no transition); pass-commands offset, count; sub-condition-offset-list offset, count; evaluator offset, evaluator length |
| Command call | int32 bank; int32 id; arguments offset; argument count |
| Argument | bytecode offset; bytecode length |

A **state group** is what the other pages call a group (`g7000`), a **state** is `s5`, and a command is written `bank:id(args)`:
`1:130101(flag, 1)` is the game's "set flag" in bank 1. Banks are in section 3.

### 2. The layout the game's compiler uses

The format does not say where the tables go. The compiler always lays them out in the same order, and writing them in that
order is what makes an unchanged script come back identical. All offsets follow from it. [data: 538 of 538 files rewrite
identically]

1. The file header, the data header, then the **state-group records**, in file order (the game does **not** sort them: 51 of
   the 53 PC event and talk scripts are out of id order; so are 475 of the 485 AI ones). [data]
2. The **state records**: for each group, its states in order, then, **if the group has more than one state, a dummy copy of
   the first state's record**. All 6,093 event groups, all 1,105 talk groups and 13,843 of the 13,844 AI groups have more than
   one state. [data]
3. The **conditions, breadth first over the whole file**: every state's conditions in group, state and list order; then all
   their sub-conditions; and so on. A condition reached from two places is stored once. [data]
4. The **command calls**: for each state in order its entry list, its exit list, its while list; then the pass commands of the
   conditions (in the condition table's order).
5. The **argument records**, in the same order as the commands.
6. The **condition-offset lists**: for each group, each state's list in state order, **plus one more copy of the first state's
   list for the dummy state** (nothing points at it); then the sub-condition lists in condition-table order.
7. The **bytecode pool**: first the evaluators of the top-level conditions; then, state by state, the evaluators of the
   sub-conditions first reached through that state, then that state's command arguments; last the arguments of the pass
   commands.
8. The **name**, UTF-16, aligned to 2 bytes, then padding to a multiple of 16 on PC (not on PS3). [data]

Four facts that matter when you edit:

- **Conditions are merged per group.** No group in any of the 538 files holds two identical conditions (same target,
  evaluator, pass commands and sub-conditions). `write()` does the same, so an edit that adds a condition identical to an
  existing one in its group does not add a record: in E2 below the new "go to state 4 if 1" merged with one already there.
  Across groups the compiler does not merge: 167 identical target-less conditions stay separate. [data]
- **Conditions are shared objects.** Where a group would hold identical conditions in two states, the file stores one and both
  states point at it: 1,160 shared conditions in the event scripts, 1,429 in the talk scripts. Change one in place and you
  change every state that points at it. [data]
- **There is at most one level of sub-conditions** in any PC file. A sub-condition is tried only when its parent passed. [data]
- **The first state record of a group is where the group starts.** Its id is 0 in every group of every file. A state you add
  runs only when a condition targets it (see [Event script groups](event-script-groups.md) for the first-state rule). [data]

**The 16 unexplained bytes.** After the data header's first word come 16 bytes that differ in all 538 PC files and match no
MD5, SHA-1 or SHA-256 of the file, its data or its name (re-checked on 60 files). The game accepted an edited file with them
unchanged (B4), and every file the writer produces keeps the bytes of the file it was read from. We never tried other values
[untested]. The padding at the end of a PC file is compiler junk (467 of 538 files have non-zero padding); the writer keeps
the source file's padding bytes and uses zeros when there are none. [data]

### 3. The commands and the bytecode

Each command call has a **bank**: [data]

| Bank | Meaning | Seen |
|---|---|---|
| 1 | a game command, `1:130101(flag, value)` | state lists in event, talk and AI scripts |
| 5 | conditional command: the first argument is a condition, the rest as the bank-1 command | state lists; 20 command records in the event scripts, 287 in the talk scripts |
| 6 | call a library state machine: the id is the group to run, and the calling state reads its result as `B9` | state lists |
| 7 | return a value from a state machine, `7:-1(14)` | **only as pass commands of a condition** |

What each command does is on [Script command reference](script-command-reference.md). Commands and functions have separate id
spaces: command `1:130452` and function `f130452()` are different things.

**Arguments and evaluators are bytecode**: a postfix stack machine, each blob ending in `0xa1`. The opcodes (names from the
community's ESDLang, checked against every blob of the game by the writer's round trip): [data]

| Byte | Meaning |
|---|---|
| `0x00`-`0x7f` | integer `byte - 64`, so -64 to 63 in one byte |
| `0x82` + int32 | any other integer (5 bytes) |
| `0x80` + float32, `0x81` + float64 | floats |
| `0x84`-`0x8a` | call with 0 to 6 arguments: pops the arguments, then the function id, pushes the result |
| `0x8c` `0x8e` `0x8f` `0x90` | `+` `-` `*` `/` (`0x8d` negate) |
| `0x91` `0x92` `0x93` `0x94` `0x95` `0x96` | `<=` `>=` `<` `>` `==` `!=` |
| `0x98` `0x99` `0x9a` | `&&` `\|\|` `!` |
| `0xa5` | a zero-terminated UTF-16 string |
| `0xa6`, `0xb7` | markers, no stack effect (below) |
| `0xa7`-`0xae` | store the top of the stack in register 0-7 (the value stays on the stack) |
| `0xaf`-`0xb6` | push register 0-7 |
| `0xb8` | push argument *n* of the current state machine (a library group's parameter) |
| `0xb9`, `0xba` | `B9` = the result of the sub-machine the state called, `BA` = the value it holds while still running; `B9 != BA` means "it has returned" |
| `0xa1` | end |

Example: `f6(100971) != 0` ("flag 100971 is on") is `46 82 6b 8a 01 00 85 a6 40 96 a1`: push 6, push 100971, call with one
argument, marker, push 0, `!=`, end. [data] **A literal above 63 takes 5 bytes, one from -64 to 63 takes 1**, so changing a
number across that line moves everything after it. E1 below does exactly that.

**Markers.** `0xa6` (written `~`) and `0xb7` (`.`) are what ESDLang calls "if false: continue / abort" markers; they do
nothing in the evaluator. The compiler puts them in by rule, and the rules reproduce all 111,124 blobs of the PC event and talk
scripts: [data]

- In a top-level condition's evaluator, every function call gets `~` (not directly inside a register store).
- The left operand of `&&` or `||` gets `.` if it is not itself an `&&`/`||`, no register is set in it or after it, and its
  `&&`/`||` chain reaches the top; otherwise `~`. So a call on the left of a top-level `&&` ends in `~.`.
- In a sub-condition's evaluator and in command arguments, only the left operand of `&&` gets a marker, always `~`.

`f6(100971) != 0` above carries one `~`; `f6(1) != 0 && f130452() == 22` carries `~.` on the left call and `~` on the right.
**AI scripts follow other habits**: the same rules recompile only 627,639 of 720,468 AI blobs (87 %: top-level evaluators
153,985 of 243,748, sub-conditions 18,550 of 18,872, arguments 455,104 of 457,848). For an AI edit, copy a neighbour's
markers or write them out. [data] Whether the game needs markers at all is not known (see Open questions).

**Registers.** The compiler caches a function's value: a state's first condition stores it (`SetREG0(f130452()) == 19`) and
the later conditions read it back (`GetREG0() == 22`). Calling the function again should also work, but only the register
form is what the compiler writes. [data; the register form is what E2 used, not a live test]

### 4. Reading scripts: `esd.py` and `edd.py`

`tools/esd.py` (shared with the project's earlier network-test work) turns a script into one canonical text form. A line is
`g<group>.s<state> entry|exit|during: bank:id(args)` or `g<group>.s<state> cond -> <target or None> if <expression> [do
<pass commands>]`; each dot after `cond` is one sub-condition level (`cond.` is a sub-condition). Its `diff()` is the tool
that proves an edit touched only what you meant: two compilations of one script differ in meaningless ways (register caching,
the numbers the compiler gives library groups, the numbers of states after an inserted one), so `diff()` substitutes
registers, pairs library groups by content and aligns states by content. A group that only one side has is listed whole, its
states named `n0`, `n1`... (the parse shows `s0`, `s1`). [data]

- **Hidden library ids (`ANON`).** The compiler numbers its library state machines counting down from 0x7fffffff (the lowest
  in the event scripts is 2147483256, in the talk scripts 2147483516). `esd.py` prints a call to any group with id at or
  above `esd.ANON` (0x7fff0000) as `6:sub(...)`, because two compilations number them differently. **To see the ids, set
  `esd.ANON` to something huge before you dump** (`esd.ANON = 1 << 62`): the line `6:sub(0, 220, 70000000, 0)` then reads
  `6:2147483634(0, 220, 70000000, 0)` (a talk-script menu library). Library ids are **per script**: 2147483634 is a menu
  library in Majula's talk script and an appearance check for a shop in Majula's event script. [data]
- **Developer names (`edd.py`).** The `.edd` beside an event or talk script names every command, function and state group in
  Japanese: commands and functions as `【category】name`, groups by the event they implement. In Majula's event script it
  names all 195 groups, the 99 library ones included. The event `.edd` files together name 325 commands and the talk ones 324
  (command 140741 is the only difference); both name the same 131 functions. [data]
  `tools/edd.py` reads the PC layout only. The file is a header of table offsets (from base 0x94) plus a string table: [data]

  | Header word | Table | Entry |
  |---|---|---|
  | +0x9c (u64), +0xa4 (s32 count) | commands | 24 bytes: s64 command id, s64 -1, s32 unknown, s32 string index |
  | +0xac, +0xb4 | functions | 8 bytes: s32 function id, s32 string index |
  | +0xbc, +0xc4 | state groups | 72 bytes: s32 group id, s32 string index, then more |
  | +0xdc, +0xe4 | strings | 16 bytes: u64 offset (from base), u64 length in UTF-16 units |

  The `.edd` is **debug data**: a group the mod adds (990000 and the rest) is in no `.edd`, and the game ran those scripts
  with the original `.edd` beside them [live B7, B9, B20d]. The release ships `.esd` files only and Steam's archives supply
  the `.edd`: the v1.0 package passed a clean install that way. Whether the game reads `.edd` files at all is not known.
  [data; live B49]
- **A Windows console trap.** Printing the Japanese names raises `UnicodeEncodeError` under the default code page. Start
  Python with `-X utf8` (with `-I`, `PYTHONIOENCODING` is ignored). [seen while checking this page]

### 5. Why not ESDLang

ESDLang (v0.5.1, August 2025) is the community's ESD tool and lists DS2 support since v0.2; its ESD class reads and writes
the little-endian formats, so it can write PC Scholar files, and its decompiled language is a good way to *read* scripts. We
wanted something different: a writer whose output is **identical to the game's own for an unchanged script**, so that
"it writes back the same bytes" is a check on every edit, and edits driven from a build script with no .NET step. From reading its
source (not run here), its layout choices differ from the compiler's in ways that do not change meaning but do change bytes:
state groups sorted by id, conditions laid out depth first per group, all evaluators before all arguments, no dummy state's
extra condition list, zero padding. It reads no big-endian files. Its bundled DS2 name lists cover the AI scripts; for event and
talk commands we read the names from the game's own `.edd` files. [inference: from its source, not run]

### A. Set up and check the writer

1. **Work from the unmodified script.** Keep the original as `<name>.esd.orig` the first time you change a file, and start every
   build from it. The project's `buildmod.py` `fresh(path)` copies the `.orig` back over the script before each build, so a
   build never edits its own earlier output.
2. **Prove the parser on your file before editing it.** `ew.write(ew.read(b)) == b` must hold. If it does not, stop: the
   edit will not be trustworthy. `esdwrite.py roundtrip <file|folder|glob>` counts the identical files.
3. **Know the shape you are changing.** Dump it (`esd.py dump file.esd`), name the groups with `edd.py`, set `esd.ANON` huge to
   see library ids.

### B. The model

`Script.groups` is a list of `Group` (id, `states`); a `Group` holds `State` (id, `entry`, `exit`, `during` lists of `Cmd`,
and `conds`); a `Cond` has `target` (a `State` of the same group, or `None`), `evaluator` (bytecode), `pass_cmds` and `subs`.
Helpers: `read(bytes)`, `write(script)`, `s.group(id)`, `g.state(id)`, `g.add_state()` (next free id), `s.add_group(id)`,
`cmd(bank, id, *args)`, `cond(target, evaluator, pass_cmds, subs)`, `sub(...)` (a sub-condition), `clone(x)`, `users(s, cond)`,
`dis(bytes)`, `asm(text)` and `compile_expr(x, kind)`.

**Expression text** is `esd.py`'s syntax: `f6(100971)`, `(a == b)`, `&&`, `B9`, `arg0`, plus `GetREG0()` and `SetREG0(x)`,
`neg()` and `not()`, `0.5` for a float32 and `0.5d` for a float64, `'text'` for a string, `int32(5)` to force the wide form,
`hex:...` for raw bytes, and a `~` or `.` suffix for a marker. Plain input with normal precedence is accepted
(`!x`, `-x`, no parentheses needed). `cmd()` and `cond()` add the compiler's markers for you; `asm()` writes exactly what you
type. `esdwrite.py asm "f6(100971) != 0 && f130452() == 22"` prints the hex.

`write()` refuses: duplicate group or state ids, a condition that targets a state outside its own group, bytecode that does not
end in `0xa1`, and **one `Cmd` object used twice** (build a new one, or `clone()` it). The expression compiler refuses a call
with more than 6 arguments. [code: `tools/esdwrite.py`]

### C. Three edits, in the shape the project used

**Change a literal** (E1 below; the same shape as every flag or text-id change):

```python
import esdwrite as ew, esd
s = ew.read(old)
st = s.group(7000).state(5)
st.entry[0].args[2] = ew.compile_expr(200, kind="arg")      # was 20
new = ew.write(s)
```

**Add a group** (the shape of `choices.spec_group()` and `arenamod.arena_group()`; the first state only moves on):

```python
s = ew.read(old)
g = s.add_group(990100)                                      # appended to the file's groups
s0, s1, s2 = g.add_state(), g.add_state(), g.add_state()     # ids 0, 1, 2; s0 is where the group starts
s0.conds.append(ew.cond(s1, 1))                              # always true
s1.entry.append(ew.cmd(1, 130101, FLAG, 1))                  # 1:130101 set flag; FLAG is your own free flag
s1.conds.append(ew.cond(s2, "f3() > 2"))                     # f3() = seconds in this state
new = ew.write(s)
```

**Patch a state of an existing group without touching the old condition:**

```python
g = s.group(42000); s0, s2 = g.state(0), g.state(2)
pre = g.add_state(); pre.entry.append(ew.cmd(1, 130101, FLAG, 1)); pre.conds.append(ew.cond(s2, 1))
s0.conds = [ew.cond(pre, 1)]            # a NEW Cond: the old one may be shared with another state
```

Then, for every edit:

4. **Write and re-read.** `ew.write(ew.read(new)) == new` must hold.
5. **Diff.** `esd.diff(old, new)` must list only the lines you meant. The project's patch functions stop with "nothing
   written" if the diff reaches another group or the re-write differs (`datapatch.py addcmd`, `choices.spec_group`,
   `modes.mytha_drain_patch`). `esdwrite.py check old.esd new.esd` prints the diff and the re-write result.
6. **Check the record counts.** The header's counts should move by what you added: a new group of N states adds 1 group and
   N + 1 state records (the dummy copy), as in E3.
7. **Install loose** where your install reads loose files ([Editing params](editing-params.md), [The legacy Mod Engine](mod-engine-ds2.md))
   and **load the map again**. Event and talk scripts are re-read at the next load of their map. [live B7, B9, B20d]
8. **Test the effect, not only the load.** A group also needs its `EventParam` row to run ([Event script groups](event-script-groups.md)).

**The writer's own checks** (`tools/esdwrite_tests.py <ezstate folder> <scratch folder>`, about 12 s, writes only into the
scratch folder, which must not sit inside the game folder): [data]

| Check | Result |
|---|---|
| Round trip of the 538 PC scripts | **538 / 538 byte-identical** (event 28, talk 25, AI 485). Also the 538 scripts of the Steam archives, read in place |
| Round trip of the mod's own 24 shipped scripts | 24 / 24 |
| Bytecode: text and back (`asm(dis(b))`) | 111,124 / 111,124 event and talk blobs, none needing raw hex; AI scripts 720,468 / 720,468 |
| Bytecode: text without markers, rules added back | 111,124 / 111,124 event and talk (top-level evaluators 37,695, sub-condition evaluators 7,215, arguments 66,214) |
| PS3 builds, offline only (from the 4 Nov run, not repeated) | v1.00 472 / 472, a network-test build 633 / 633 (with its stray `fSSL` file), PS3 Scholar 581 / 581 |

**The three test edits**, each made on a fresh read, written, read back with `esd.py`, and required to give exactly the
intended lines, exactly the intended count changes, and the same bytes when written again: [data]

| Edit | Script | Change | File size | Record changes |
|---|---|---|---|---|
| E1 | `talk_m10_04` (Majula) | one argument 20 -> 200 (1 byte -> 5; everything after it moves) | 298,000 -> 298,000 (the 4 bytes fitted in the padding) | none |
| E2 | `talk_m10_04` | a new state in the menu library group 2147483634 that returns 14 when the window's result is 22, and a condition in group 7860's menu state that goes back to its state 4 on 14 | +320 | +1 state, +3 conditions, +1 command, +1 argument |
| E3 | `event_m10_04` (Majula) | a new group 4999: state 0 waits for flag 100971, state 1 runs the event warp `1:132125(0, 0, 40030000, 400010, 0)` | 239,936 -> 240,400 | +1 group, +3 states, +1 condition, +1 command, +5 arguments |

E3's three new state records are its two states and the dummy copy; its one condition is state 0's; its one command is the warp,
with its 5 arguments. The tests still pass when pointed at the debug copy's folder after the mod edited its scripts (that folder
then holds our written versions: the sizes differ, the diffs are the same). E2 and E3 were checked offline only; they never ran
in the game. [data]

### D. No writer needed: swap two literals (`esdswap.py`)

When two numbers have the same size, you can swap them in place. A literal above 63 is `0x82` + int32, 5 bytes on PC; swapping
two keeps every size and offset. `tools/esdswap.py swap <file.esd> <a> <b>` requires each value to occur **exactly once** as
`0x82` + int32 in the file, re-reads the result with `esd.py` and refuses unless only the lines holding the two values
changed. The first copy is kept as `<file>.orig`; `restore` puts it back. This was **B4**: it swapped two greeting text ids
(70000300 and 70000400) of two states in the Emerald Herald's talk group, in `talk_m10_04`. [live B4]

## Pitfalls and what failed

| Test or source | What happened | Lesson |
|---|---|---|
| **E2** (4 Nov, offline) | The first note called group 7860 the Emerald Herald's menu and the new return value "a menu entry". Group 7860 is the **Victor's Stone** (hers is talk group 7000), and the edited library is the menu library **shared by every Majula NPC menu**. The window returns fixed codes from `NpcEventParam`, so "22" can never come back | E2 proved the writer, not a usable menu entry. Read what the host really is before you build on it. See [NPC menus](../modding-menus/npc-menus.md) [data, from the 5 Nov re-read] |
| **B9, first build** (4 Nov) | A new group ran but the commands in its first state did nothing | Nothing in `s0`; see [Event script groups](event-script-groups.md). The writer cannot know this: it wrote exactly what it was given [live B9] |
| **Editing a shared condition** | A condition object can belong to several states. Changing its target in place changes them all | Build a new condition (`s0.conds = [ew.cond(...)]`). `ew.users(s, c)` lists who holds one; `clone()` makes an unshared copy [data] |
| **A command used twice** | The writer refuses a `Cmd` object that appears in two lists | New command objects each time; `tools/fightlist.py` has a comment for this "esdwrite rule" [code] |
| **A state nothing targets** | `add_state()` gives a state the next free id. The game starts a group at its first state, so any other state runs only if a condition points at it | Add the condition that reaches it, and test that it is reached [code: `add_state()`; data: every group starts at id 0] |
| **Merged conditions** | `write()` merges a condition identical to one already in its group, so your "added" condition may be the old one | Expected: it is what the compiler does. The `esd.py` diff shows one `+` line, not two (E2) [data] |
| **B20d header** (6 Nov) | The test text said a talk script needs a game restart | The result: the same game process, the next Majula load read the new talk script. Trust the result [live B20d] |
| **Chasing the 16 bytes** | No hash of the file, data or name matches them; they differ in every file | Leave them alone: keep the source file's [data; live B4] |
| **Matching ESDLang's output** | Its files are valid but laid out differently from the game's | Not a fault: the goal differed. For byte-identical checks use the compiler's order (section 2) |
| **AI scripts and the marker rules** | The rules match only 87 % of AI blobs | Copy markers from a neighbour for AI edits. We never edited an AI script [data; untested in game] |
| **Asking the original tools** | `esd.py`'s old big-endian-only reading failed on the one stray little-endian 32-bit file in the PS3 builds | It now reads all three magics [data] |

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| A loose talk script is read; `esd.py`'s state-to-line reading matched what the NPC said | live | B4 (28 Oct) |
| An event script whose layout changed (two commands added to a group's state) loads and runs | live | B7 (4 Nov) |
| A new state group with its own `EventParam` row runs; the original `.edd` knows nothing of it | live | B9 (4 Nov) |
| A talk script with many new states loads and runs; no game restart | live | B20d (6 Nov) |
| The release folder holds 24 `.esd` files (23 event, 1 talk), all written by the writer; the v1.0 package works on a clean install with Steam's original `.edd` files | data; live | release folder, 17 Nov; B49 (8 Nov) |
| 538 of 538 PC scripts round-trip byte for byte; the 24 shipped ones too | data | `tools/esdwrite_tests.py`, re-run 17 Nov |
| The table order, the dummy state and its offset list, per-group merging, one sub-condition level | data | 538 files, recounted 17 Nov |
| 1,160 shared conditions (event), 1,429 (talk); 51 of 53 event / talk scripts unsorted | data | recounted 17 Nov |
| The marker rules reproduce 111,124 of 111,124 event and talk blobs, 87 % of AI blobs | data | recounted 17 Nov |
| The 16 bytes after the data header's first word differ in all 538 files and match no hash tried | data | recounted 17 Nov |
| `esd.py` shows library ids with `esd.ANON` raised | data | run 17 Nov |
| `.edd` names: 325 event commands, 324 talk, 131 functions; all 195 groups of Majula's event script | data | `tools/edd.py`, run 17 Nov |
| The record layouts and the header words | data | `tools/esdwrite.py` `read()`, which checks them in every file |
| The compiler's layout is a design choice we copied; the game's own reading of it | not known | no exe work on the loader |
| ESDLang writes differently from the compiler | inference | from its source, not run |
| PS3 and 2014-PC files round-trip | data, offline only | 4 Nov run; never loaded in a game |

## Tools

All in the project's `tools/` folder, except `esd.py` and `edd.py`, which come from the earlier network-test project and are
shared.

| Tool | Use for this topic |
|---|---|
| `tools/esdwrite.py` | Read, edit and **write** scripts; its command line: `roundtrip`, `bytecode`, `dis` (lossless text of every state), `asm` (expression to hex), `check old new` (diff plus re-write check) |
| `tools/esdwrite_tests.py` | The writer's checks (round trip, bytecode, E1-E3); writes only to the scratch folder you give |
| `tools/esdswap.py` | Swap two same-size literals in place (B4); keeps `.orig`; `restore` |
| `tools/esd.py` | Read: `dump <file>`; importable `parse`, `lines` and `diff(old bytes, new bytes)`; the `ANON` switch. Shared |
| `tools/edd.py` | The developer names in `.edd` files (`commands`, `functions`, `machines`, `strings`). Shared; PC layout only |
| `tools/choices.py` `spec_group()` | A new group from a list of states and conditions; refuses commands in the first state and a file that does not re-write identically |
| `tools/datapatch.py` `addcmd` | Append commands to a state's entry list (B7); refuses if the diff leaves that group |
| `tools/arenamod.py` `arena_group()` | The standard per-map group: the worked example of a large group |
| `tools/ids.py` | Fails a build whose new group id is outside the mod's block |

## Open questions

- **Does the game need the markers?** We always write the compiler's. Write one condition without them. [untested]
- **Does the game read the dummy state, or its offset list?** We write both as the compiler does. [untested]
- **Is any of the 16 header bytes checked?** Write a file with them zeroed. [untested]
- **Does the game read `.edd` files?** An edited script that disagrees with its `.edd` ran every time; a missing `.edd` was never
  tried. [untested]
- **Does a new group run with no row?** [Event script groups](event-script-groups.md) says no (B53 deleted rows and the groups stopped);
  a bank-6 call or `1:130003` as a starter was never tried. [untested]
- **Does the order of groups in the file matter?** New groups go last, and that worked. [untested]
- **Can you add a *usable* menu entry?** Not by script alone: see [NPC menus](../modding-menus/npc-menus.md).
- **AI scripts.** The writer round-trips them; we never put an edited one in the game. [untested]
- **Command meanings** are on [Script command reference](script-command-reference.md); the loader and the command handlers in the
  exe were not read for this page.

## Related pages

- [Event script groups](event-script-groups.md): what goes in a group, the `EventParam` row, the first-state rule
- [Script command reference](script-command-reference.md): the commands and functions you put in states
- [NPC menus](../modding-menus/npc-menus.md): why E2's "choice 22" was not a menu entry
- [Adding dialogue options](../modding-menus/adding-dialogue-options.md), [Asking the player a question](../modding-menus/yes-no-windows.md), [Vendor stock](../modding-menus/vendor-shops.md), [Giving the player items](../modding-menus/giving-items.md): talk-script edits built with the writer
- [Editing params](editing-params.md): the `EventParam` row and loose files
- [The legacy Mod Engine](mod-engine-ds2.md): how loose files reach the game on a Steam install
- [Live-testing tricks and pitfalls](live-testing-notes.md): reloading scripts and watching their effects
- [Event flags and free ids](event-flags-and-free-ids.md): free flags and the group-id block
