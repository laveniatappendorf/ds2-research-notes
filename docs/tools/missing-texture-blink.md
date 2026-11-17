# The missing-texture blink, and switching it off (2026-10-22)

!!! warning "Status: Not yet rated"
    Not yet given a status. Treat as a research note and check anything you plan to rely on. See [status labels](../status-labels.md).

Retail v1.00 draws any texture it can't find (a map piece's diffuse or lightmap, see `unused-map-pieces.md`) as flat
white for 0.5 s, then black for 0.5 s. That's the engine's own placeholder, a two-frame flip-book, and one instruction
makes both frames white.

## How it works

- At boot the texture manager makes 10 built-in 1x1 textures from a colour table (`0xb38a68` loop, `0xb3d300` creates,
  `0xd3ca90` fills). Array at `[manager+0x260]`, 0x14 bytes an entry, handle at +0xc; `0xb32e50(manager, kind)` returns
  one. Kind 0 = `0xFFFFFFFF` (white), kind 1 = `0xFF000000` (opaque black); 3 and 5 are the specular and normal-map
  defaults.
- `0x75bbe8` registers six of them by name: `dummy128` and `t_dummy_d` = kind 2, `spe_040_s` and `t_dummy_s` = kind 3,
  `t_dummy_n` = kind 5, `t_dummy_0` = kind 1 (names at `0x17ff308`-`0x17ff380`, reached through the unit base in TOC
  slot `0x1d17db4`, offsets -0x7fe8 to -0x7fd4; the PS3 compiler addresses each unit's data as base - 0x8000 + k).
- Straight after the built-in loop, `0xb38ae8`-`0xb38b28` builds an object from kind 0 and kind 1 and stores it at
  `manager+0x264`. Its constructor `0xb9e818` keeps frame A at +0x28, frame B at +0x2c, a counter at +0x30 and a float
  timer at +0x34 that it starts at **0.5**. Live it reads anywhere from 0 to 0.5 (0.03-0.45 over several reads), so it's
  a running half-second timer, not a constant. That's the blink: white, black, every 0.5 s.

## Switching it off

**From boot (RPCS3 patch):** `0xb38b04` `lwz r28, 0xc(r11)` (`0x838B000C`, frame B = black) becomes `lwz r28, 0xc(r9)`
(`0x8389000C`, frame B = white). Patch file: `DS2 Vanilla\rpcs3-patches\imported_patch.yml`, keyed to this EBOOT's PPU
hash `PPU-efd8e00902c586ebf88f5c97dcbdfe27bdce3bcc` (from RPCS3.log; the official patch.yml lists the same hash for
BLUS41045 1.00). To use it: copy it into RPCS3's `patches` folder as `imported_patch.yml` (there's none there yet; if
one appears later, merge the entry into it), then tick "Missing textures stay white (no blink)" under Manage Game
Patches and boot. RPCS3 rewrites `patch.yml` when it updates, so the entry must not go there.

**In a running game:** `DS2 Vanilla\tools\vwhite.py on` writes the white handle over frame B (`off` restores black,
`status` shows the chain). Chain: owner = `[0x1d99650]` (the address in `[[TOC-0x72b4]-0x7fec]`), manager =
`[owner+0x24]`, flip-book = `[manager+0x264]`. It checks frame A is white, frame B is white or black and the timer is
between 0 and 1 before writing. Lasts until the game restarts.

Either way, a missing texture then looks the way the white phase did: flat white where the diffuse is missing,
unshaded where the lightmap is missing. Anything else drawn with the flip-book (for example a texture that is still
streaming in, if the engine uses it for that) would also show white instead of blinking.

**Status:** `vwhite.py` run live 2026-10-22 (manager 0x30517070, flip-book 0x37a98ab0, white 0x37a98690, black
0x37a986f0): the chain read exactly as worked out (frame A white, frame B black), the first version refused because it
expected +0x34 to hold a constant 0.5, and after the fix `on` made frame B white, read back. **the author confirmed in game:
the untextured assets are steady white.** The RPCS3 boot patch was copied into RPCS3's `patches` folder as
`imported_patch.yml` the same day (no file of that name was there before); ticking it and checking after a reboot is
The author's next step.
