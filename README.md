# tiny-chess-4k

A 4,096-byte chess engine for the [make.fun](https://make.fun) competition
**tiny-chess-01**: one Python file, standard library only, no clock and no
randomness, so a replay plays exactly the same moves.

[![tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun)
[![rank](https://make.fun/badge/tiny-chess-01/hermansto/tiny-chess-4k/rank.svg)](https://make.fun)
[![reward](https://make.fun/badge/tiny-chess-01/hermansto/tiny-chess-4k/reward.svg)](https://make.fun)

## The engine

`main.py` is the whole entry: about 13.7 KB of source shipped as an LZMA
payload that unpacks and execs itself in 4,027 bytes.

- **Board.** A 10x12 mailbox, border 7, rows 2..9 by files 1..8, pieces
  1..6 for P N B R Q K and negative for black. A move is one int:
  `from | to << 8 | promo << 16 | flag`.
- **Search.** Negamax alpha-beta with iterative deepening to a fixed
  budget of 60,000 nodes per move: transposition table on 64-bit Zobrist
  hashes (castling rights and en passant included), killer moves, history
  heuristic, null-move pruning when not in check, a one-ply late move
  reduction for late quiet moves with a full re-search on a fail high,
  and a capture quiescence search with delta pruning and a light
  static-exchange filter. Promotions are generated in quiescence too.
- **Legality.** Pieces move pseudo-legally; a move that leaves its own
  king under attack is refuted in the child node, where capturing a king
  returns `MATE - ply` at once. The move that is sent is filtered against
  the env's own `legal_moves`, so it is always legal.
- **Evaluation.** Material (P100 N320 B330 R500 Q900) plus piece-square
  tables kept incrementally in make/unmake, a castled king scored above a
  central one, and an endgame term that drives the losing king to the edge
  and walks the winning king in once the material swing passes 400.
- **No shuffling.** Every position of the game is counted by Zobrist
  hash; a root move that would repeat one is pushed down hard, and a
  small contempt bonus prefers a capture when the score is level.

## Running it

The engine talks JSON over HTTP to the env at `$MAKEFUN_ENV_URL`:
`POST /game {}` serves the game in play and `POST /move {"game": n, "move": "e2e4"}`
plays a move. Under the competition kit:

    MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py

To grade it locally, from inside the dev kit:

    makefun grade-local --kit . --bot <this repository> --driver docker

## Licence

MIT, see `LICENSE`.
