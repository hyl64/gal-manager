# gal — one command for the galgame runtimes in this tree

Each engine in `~/Documents/gal-engines` ships its own binary and its own idea
of how to be started: OnscripterYuri wants `--root`, OpenBGI wants the game
folder as its working directory.  `gal` looks at a game folder, works out which
engine it belongs to, remembers it, and starts the right binary the right way.

```bash
./gal engines             # what is built, where, and how far along
./gal add <folder>        # detect the engine and add the game to the library
./gal list                # the library
./gal info <game>         # why the engine was recognised
./gal launch <game>       # start it      (--dry-run prints the command)
./gal verify <game>       # start it headless for a few seconds and report
./gal remove <game>       # drop it
./gal doctor              # check the binaries and the folders
```

The library is JSON under `~/Library/Application Support/gal/library.json`;
the game folders stay where they are.

## Where the engines stand, verified by `gal verify`

Measured, not claimed - the output of `gal verify` on every data set in the
library at the end of round 37:

| engine | data set | verdict |
|---|---|---|
| NScripter | `testdata/lifegame` (the official OnscripterYuri demo) | **runs** - enters its main loop |
| NScripter | `testdata/nscripter-sdk/.../サンプル1` | **runs** |
| NScripter | `.../サンプル2-1`, `2-2`, `3` | **runs** |
| Ethornell/BGI | the official Lump of Sugar trial (1.669) | **not yet** - the engine identifies it, opens a window and decodes its data, then stops on opcodes that upstream has not implemented for 1.669 (200 of them) |
| KiriKiri | - | TJS2, XP3, storage, the software renderer, images and text all work; no game data to run, and the layer system still stops on its first fill |
| RealLive | - | ported in another session, no binary yet |
| Artemis | - | written from scratch: `.pfs` and `.ast` read and write, no game data to run |

Two engines run real data; the others are honest about where they stop.

## How detection works

Each engine declares signature files with weights; the engine with the most
evidence wins, so one generic name does not mis-file a game.

| engine | signature files |
|---|---|
| NScripter | `nscript.dat`, `nscr_sec.dat`, `00.txt`, `0.txt`, `*.nsa`, `*.ns2`, `*.nt2`, `*.nt3` |
| KiriKiri | `data.xp3`, `patch.xp3`, `startup.tjs`, `*.xp3`, `*.tjs` |
| RealLive | `Seen.txt`, `Gameexe.ini`, `*.g00`, `*.nwk`, `*.koe`, `*.ovk` |
| Ethornell/BGI | `system.arc`, `sysprg.arc`, `comap.dat`, `BGIError.txt`, `BGI.dat`, `*.arc` |
| Artemis | `*.pfs`, `*.ast`, `*.iet` |

## Engine status

| engine | binary | state |
|---|---|---|
| NScripter → OnscripterYuri | `OnscripterYuri/build/onsyuri` | **runnable** |
| Ethornell/BGI → OpenBGI | `openbgi/openbgi-src/bin/bgi` | **runnable** |
| KiriKiri → Kirikiroid2 | `kirikiroid2-mac/build/tjsc` | partial: TJS2 + XP3 work, no window yet |
| RealLive → RLVM | `rlvm/build/rlvm` | owned by the other session |
| Artemis | — | not implemented, the from-scratch target |

`launch` refuses the engines that cannot run a game yet and says why, instead
of failing halfway.

## Real data it runs today

Five data sets are registered and pass `gal verify` (headless, no window):

| data set | source |
|---|---|
| `testdata/lifegame` | the official OnscripterYuri web demo (`onscript.nt2`, an encrypted script) |
| `testdata/nscripter-sdk/n18/.../サンプル1` | NScripter final SDK, Lua value passing |
| `.../サンプル2-1` | NScripter final SDK, a square bouncing around |
| `.../サンプル2-2` | NScripter final SDK, text turned into sprites |
| `.../サンプル3` | NScripter final SDK, the input functions |

The NScripter2 SDK samples are *not* in the list: they are written in
NScripter 2's Lua dialect, which `verify` correctly reports as
`Script error, Label "define" is not found`.

## How verify judges

`verify` starts the engine headless and reads both the output and the clock:

| what happened | verdict |
|---|---|
| still running when the window closed | ✓ entered its main loop |
| exited on its own after 2 s or more | ✓ the script ended normally |
| exited on its own in under 2 s | ⚠ gave up: missing data or a suppressed error |

The third row is the important one: the engines report errors through a
message box, and a headless run has none, so a quiet early exit is the only
evidence that something was missing.  The BGI trial below is exactly that case
and `verify` refuses to call it a pass.

## Verified

```
$ ./gal add ../testdata/lifegame        # the real NScripter data set
已加入: lifegame
  引擎: NScripter → OnscripterYuri
  依据: onscript.nt2

$ ./gal launch lifegame --dry-run
cwd:  .../testdata/lifegame
argv: .../OnscripterYuri/build/onsyuri --root .../testdata/lifegame --enc:sjis

$ ./gal launch lifegame                 # really starts the engine:
ONScripter Yuri 0.7.7 (Jh0.8.0,Ons20181218,NSC2.96)
Display: 800 x 600 (32 bpp)
Audio: 44100 Hz 16 bit stereo
```
