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
./gal remove <game>       # drop it
./gal doctor              # check the binaries and the folders
```

The library is JSON under `~/Library/Application Support/gal/library.json`;
the game folders stay where they are.

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
