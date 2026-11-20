# tiny-chess

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/iosai2035/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/iosai2035/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A clean-room 4 KB chess engine in one Python file. 0x88 board, alpha-beta negamax with iterative deepening under a fixed node cap (never the wall clock), Principal Variation Search, null-move pruning, capture-only quiescence with check evasions, MVV-LVA plus killer ordering, a small Zobrist transposition table, and material plus piece-square evaluation with an endgame king table. No randomness, no clock: the same position always gives the same move, so replays match. Standard library only, CPython 3.13.

## Run it

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:8080 python3 main.py
```

The engine talks to the make.fun environment server: it reads the position (FEN plus legal moves) from `POST /game`, answers on `POST /move`, and resigns only if the server stops answering.

## Design

- **Board:** 0x88 mailbox, pseudo-legal generation with make/unmake legality test, all four promotions, castling only when legal, en passant including the en-passant pin.
- **Search:** alpha-beta, iterative deepening to depth 12, stopped by a 50,000-node cap so the slowest move fits the 10 s limit on hardware ~4x slower than the build machine. First move on a full window, later moves on a null window with re-search; null-move pruning (R=2, depth ≥ 3, never in check) prunes quiet positions. Mate scores prefer faster mates (`MATE − ply`); insufficient material is a draw.
- **Quiescence:** capture-only with stand-pat and delta pruning; in check, all evasions are searched and checkmate scores as mate, so mate threats are visible at the horizon.
- **Evaluation:** material (100/320/330/500/900) plus my own piece-square tables (mirrored, 32 values per piece): centralised knights and bishops, rooks on the seventh rank, advanced central pawns, a king that stays home in the middlegame and centralises in the endgame.

`main.py` (3,869 bytes) is the readable engine (`engine.py`, kept beside the build) packed with the make.fun kit's `compress` (LZMA). Perft: startpos(5) = 4,865,609; Kiwipete(3) = 97,862.

---

<sub>Entered on [make.fun](https://make.fun/?ref=57z043hx&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
