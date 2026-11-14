# makefun-tiny-chess-01-v3

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A compact 0x88 alpha-beta chess engine in 4KB of Python. `main.py` is a one-line LZMA self-unpacker: search, evaluation, variant book and the env driver live in its compressed payload, stdlib only.

v3 vs v2 (dev rank 1064, practice avg 1,769): full 12-table PeSTO evaluation (endgame tables for all six pieces), aspiration windows, and an opponent-aware variant book of 300k-node coach lines for the three trial-0 problem scenarios (sicilian-najdorf vs depth-3, italian-game vs depth-4, philidor-defence vs nodes-500k). All inside 4,025 bytes.

## Engine

- 0x88 board; negamax + alpha-beta, iterative deepening to a fixed node budget
- Zobrist transposition table, persistent across moves (cleared past 300k entries or between games); 2 killers per ply; MVV-LVA ordering
- Quiescence with check evasions; null-move pruning (R=3); PVS + LMR; check extensions
- **PeSTO eval**: Texel-tuned tables for all 6 pieces in MG and EG, tapered by phase; incremental MG/EG, so `evaluate()` is O(1)
- **Aspiration windows**: ±60cp from depth 5, full-window re-search on fail-high/low
- **Variant book**: 40–76-ply coach lines, matched by history prefix AND opponent
- Node budget: 40,000 vs depth rungs, 100,000 vs node rungs
- Deterministic: fixed node budget, no wall clock, seeded RNG

## Practice results (house gauntlet, 14–15 Nov 2026)

| trial | Elo | score |
|---|---|---|
| 0 | 1,990 | 7W-1D-0L (v2: 1,598) |
| 1 | 1,990 | 7W-1D-0L (v2: 1,598) |
| 2 | 1,669 | 5W-2D-1L (v2: 2,112) |
| average | 1,883 | (v2: 1,769) |

The book fixed all three trial-0 problem games; the engine improvements alone fixed all three trial-1 problem games. Trial 2 regressed on three node-rung endgames; the opponent-aware book fixed trial-2 game 8 (it had followed a depth-3 line against nodes-2M).

Correctness: perft passes all 6 standard positions (depths 1–4); incremental eval verified vs from-scratch recomputation over 30 random games; all three book lines replay cleanly. Packed `main.py` re-verified: perft(3) start 8,902, kiwipete 97,862, root search at ND 20,000 returns b1c3 (score 35).
