# SC-3000 Verilator simulator

Headless simulator of the machine in `../rtl/` (the same files the MiSTer
build uses). Needs Verilator 5 and zlib. No ROMs or software are included:
point the options at your own files.

```
make                      # obj_dir/Vsc3000_sim
make full                 # the same at the real clock ratios (slower)
make test                 # keyboard unit test, then tests/regress.tsv
./obj_dir/Vsc3000_sim --help
```

`tests/regress.tsv` lists cartridges by path relative to `verilator/`
(`../software/...`); rows whose file is missing are skipped.

Examples:

```
# boot a cartridge, picture at frame 600 (mapper detected automatically)
./obj_dir/Vsc3000_sim --cart game.sg --frames 600 --png out/game.png

# BASIC Level III: type a program, run it, print the screen as text
./obj_dir/Vsc3000_sim --cart basic3.bin --frames 800 \
    --type '10 FOR I=1 TO 5:PRINT I*I;:NEXT\nRUN\n@400' --text-end

# paste through the PS/2 keyboard path, at a PC paste's speed
./obj_dir/Vsc3000_sim --cart basic3.bin --frames 1500 --paste '10 PRINT "HI"\nRUN\n@400' --text-end

# CPU trace in MAME tracelog format (frames 0-2)
./obj_dir/Vsc3000_sim --cart game.sg --frames 3 --trace out/game.tr:0:2
```

About 13 frames/s on an M-series Mac (0.22x real time).

`--type` drives the key matrix directly (MAME `sk1100.cpp` layout): 6 frames
per key, 30 after RETURN; lower case types as capitals, `\` is the yen key.
`--paste` sends PS/2 key events through `rtl/sc3000_keyboard.sv`, as the
MiSTer does.

## Tape and SF-7000

```
# BASIC Level III, LOAD from a WAV (the deck plays while the program polls PB7)
./obj_dir/Vsc3000_sim --cart BASIC3.sg --tape game.wav --tape-log --type 'LOAD\n@400' --frames 3300 --text-end

# SF-7000: IPL ROM + Disk BASIC disk; --disk-out writes the image back
./obj_dir/Vsc3000_sim --sf7000 ipl.rom --disk basic.sf7 --type 'FILES\n@1000' --frames 1300 --text-end
```

Othello Multivision: `--omv omvbios.bin` (BIOS, no cartridge), `--pad KEYS@FRAME[:N]`
presses keypad keys (1-9, 0, A-H):

```
# title, S2 = two players, then D 3 S1 plays D3
./obj_dir/Vsc3000_sim --omv omvbios.bin --pad 1@320:5 --joy b@500:5 --pad D@700:5 --pad 3@760:5 --joy a@820:5 --frames 1000 --png omv.png
```

`make fdclog` builds `obj_log/` with every FDC command and result printed.

## Known gaps

- The first instruction after reset (0000) is missing from `--trace`.
- No GUI build yet (ImGui: copy ColecoAdam `verilator/sim/`).
- No tv80 build.
