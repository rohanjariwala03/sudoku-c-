# sudoku

A terminal Sudoku game in C++. It generates a random puzzle, lets you fill it in from the command line, and can solve it for you. It also has a second solver based on Douglas-Rachford splitting and a mode that times the two solvers against each other.

The source file headers credit Matt Olson and Alex Izydorczyk. I wrote about it here: https://medium.com/@rohanjariwala03/sudoku-in-c-d058859abbe4

## How it works

- Puzzle generation (`solver.cpp`): fill the diagonal with a random permutation of 1..N, solve the board with backtracking, then blank out all but `nobs` cells. The remaining cells are locked.
- Backtracking solver (`solver.cpp`): fills cells left to right, top to bottom, trying 1..N and backing up when nothing fits.
- Douglas-Rachford solver (`altproj.cpp`): represents the board as an N x N x N cube with Armadillo and alternates reflections onto the row, column, block, one-value-per-cell and given-clue constraints. It stops when the error drops below `1e-7` or after 10,000 iterations.
- Board size can be any perfect square (4, 9, 16, ...).

## Building

Needs a C++11 compiler and [Armadillo](https://arma.sourceforge.net/) installed where the compiler and linker can find it. The Makefile passes `-stdlib=libc++`, so it is set up for clang on macOS.

```
make          # builds ./sudoku from main.cpp
make debug    # builds ./debug, which runs the DR solver on a random 16x16 board
make clean
```

The `sudoku` binary in the repo is an old x86_64 macOS build linked against `libarmadillo.5.dylib`. Rebuild it rather than relying on it.

## Playing

```
./sudoku -s 42 -g 9 -n 30
```

| Flag | Meaning | Default |
| --- | --- | --- |
| `-s`, `--seed` | random seed | none, pass one |
| `-g`, `--gamesize` | board size N | 9 |
| `-n`, `--nobs` | number of prefilled cells | 10 |
| `-u`, `--Unittest` | run the timing comparison this many times instead of playing | 0 |
| `-v`, `--Verbose` | print each timing run | off |

During the game, type a move as `row column value` (all 1-based), for example `3 5 7`. Moves that break a row, column or block rule are rejected, and the clashing cells are printed in red. Prefilled cells can't be changed.

Commands:

- `solve` runs the backtracking solver on the current board.
- `clear` removes all of your entries.

If your entries make the board unsolvable, it offers to clear them and solve from the original clues.

## Timing mode

```
./sudoku -s 1 -g 9 -n 20 -u 100 -v
```

Generates a puzzle per run, times the backtracking solver and the DR solver in nanoseconds, and prints the averages.

## Files

- `main.cpp` argument parsing and entry point
- `game.cpp` / `game.hpp` the interactive game loop
- `solver.cpp` / `solver.hpp` `Board` class, backtracking solver, puzzle generator
- `altproj.cpp` / `altproj.hpp` Douglas-Rachford solver
- `tests.cpp` / `tests.hpp` timing comparison
- `debug.cpp` standalone DR test program

## Known issues

- There is no default seed. If you leave out `-s` the seed is uninitialized.
- `-rp` / `--RP` is parsed but doesn't change anything. The game always uses backtracking.
- In timing mode the backtracking solver runs first and fills in the board, so the DR solver is timed on an already solved board.
- If you finish the puzzle by hand, the game still prints "Could not solve puzzle based on pre-filled values", because the success flag is only set by the `solve` command.
