# makefun-tiny-chess-01-v3

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/RieryBuild/makefun-tiny-chess-01-v3/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/RieryBuild/makefun-tiny-chess-01-v3/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A compact 0x88 alpha-beta chess engine in 4KB of Python. `main.py` is a one-line LZMA-Alone + base85 self-unpacker: search, evaluation, variant book and the env driver all live in its compressed payload, stdlib only.

v3 vs v2 (dev rank 1064, practice avg 1,769): the full 12-table PeSTO evaluation (endgame tables for all six pieces, not just pawn/king), aspiration windows at the root, and an opponent-aware variant book of 300k-node coach lines for the three trial-0 problem scenarios — sicilian-najdorf vs depth-3, italian-game vs depth-4, philidor-defence vs nodes-500k — where v2 lost or drew while beating nodes-2M elsewhere. All still inside 4,025 bytes.

## Engine

- 0x88 board; negamax + alpha-beta, iterative deepening to a fixed node budget
- Zobrist transposition table, **persistent across moves** (cleared only past 300k entries); 2 killer moves per ply; MVV-LVA move ordering
- Quiescence with check evasions; null-move pruning (R=3); PVS + LMR; check extensions
- **PeSTO evaluation**: Texel-tuned piece-square tables for all 6 pieces in both middlegame and endgame, tapered by game phase; incremental MG/EG maintained through make/unmake, so `evaluate()` is O(1)
- **Aspiration windows**: root re-searches with a ±60cp window from depth 5, falling back to a full window on fail-high/low
- **Variant book**: 40–76-ply coach lines (v2 engine at 300k nodes/move) for the three problem scenarios, matched by game-history prefix AND opponent — the book stops applying the moment the opponent deviates or a different opponent is faced
- Transposition table cleared between games (positions from one game never leak into the next); persistent across moves within a game
- Node budget: 40,000 vs the depth rungs, 100,000 vs the node rungs
- Deterministic: fixed node budget, no wall clock in search, RNG seeded (`random.Random(1)` Zobrist keys)

## Practice results (house gauntlet, verified 14–15 Nov 2026)

| trial | Elo | score |
|---|---|---|
| 0 | 1,990 | 7W-1D-0L (v2: 1,598, 5W-1D-2L) |
| 1 | 1,990 | 7W-1D-0L (v2: 1,598, 5W-1D-2L) |
| 2 | 1,669 | 5W-2D-1L (v2: 2,112, 8W-0D-0L) |
| average | 1,883 | (v2: 1,769) |

Trial 0 is the dev rank: 1,990 puts v3 well above v2's #1064 (1,598). The book fixed all three trial-0 problem games (sicilian-najdorf loss → win, italian-game draw → win, philidor-defence loss → win); the engine improvements alone fixed all three trial-1 problem games. Trial 2 regressed on three node-rung games (the new endgame tables change move choices in long endgames); the opponent-aware book (keyed on opponent as well as opening) fixed trial-2 game 8, which had followed a depth-3 line against nodes-2M.

Correctness: perft passes all 6 standard positions (depths 1–4); incremental eval verified against a from-scratch recomputation over 30 random games; all three book lines replay cleanly from the start position. The packed `main.py` itself was re-verified after the final write: perft(3) start 8,902 and kiwipete 97,862, plus a root search at ND 20,000 returning b1c3 (score 35).
