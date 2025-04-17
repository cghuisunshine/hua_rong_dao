# Klotski (华容道) Game – Code Architecture & Solver Algorithm

## Overview

This project implements a fully‑functional **Klotski / 华容道** sliding‑block puzzle in pure HTML + CSS + JavaScript.  The app supports:

- Multiple preset layouts ("classic", "besieged", "corner" …)
- Touch & mouse dragging with smooth CSS‑transitioned motion
- Automatic solving via a breadth‑first search (BFS) algorithm
- Live solution playback with step‑by‑step animation
- Language switching (Chinese ⇆ English)

The source code is self‑contained in a single HTML file.  This README explains how the game state is modelled and how the solver works.

---

## 1. State Design

### 1.1 Piece Object

Every block is stored as a `` record inside the global `pieces` array:

```js
{
  id : "曹操",   // unique name, doubles as display text
  x  : 1,        // left‑most grid column (0 – 3)
  y  : 0,        // top‑most grid row    (0 – 4)
  w  : 2,        // width  in cells      (1 or 2)
  h  : 2         // height in cells      (1 or 2)
}
```

*The board is a fixed 4 × 5 rectangle; **`x`** & **`y`** always keep the piece within bounds.*

### 1.2 Layouts

The constant `layouts` maps a **layout‑key** to its initial `pieces` array.  Selecting a layout performs:

```js
pieces = structuredClone(layouts[key].pieces);
currentLayoutKey = key;
render();
```

No other global state needs resetting because every visual element is recreated from `pieces` on each `render()`.

### 1.3 Transient UI State

| Flag / var          | Purpose                                  |
| ------------------- | ---------------------------------------- |
| `isDragging`        | User is currently dragging a block       |
| `isSolving`         | Solver playback is running; UI is frozen |
| `solutionSequence`  | Array of `{id,x,y}` moves from BFS       |
| `solutionTimeoutId` | `setTimeout` handle for next animation   |

These flags keep the model (board state) cleanly separated from view‑layer concerns.

---

## 2. Board Encoding

Efficient BFS requires a *hashable* representation of a position.  `encodeBoardFast()` tiles the 4 × 5 grid into a 20‑character string:

| Char | Meaning              |
| ---- | -------------------- |
| `C`  | 2×2 Cao Cao block    |
| `B`  | 2×2 (generic big)    |
| `h`  | 2×1 horizontal block |
| `v`  | 1×2 vertical block   |
| `s`  | 1×1 soldier          |
| `.`  | empty cell           |

Example (classic start):

```
.vCB
h.h.
......
```

flattened to `vCBh.h...............` (`20` chars).

To exploit left‑right symmetry, the code also builds a **mirror** string (row‑wise reversal) and returns the lexicographically smaller of the two.  This halves the search space with zero extra cost.

---

## 3. Move Generation

`nextPositionsFast(statePieces)` yields neighbours in two steps:

1. **Locate the two empty cells** (always exactly two in Klotski).  For each empty cell, examine its four von‑Neumann neighbours.  If a neighbour contains a piece ID, attempt to slide that piece **one step toward the empty cell**.
2. **Validate** the candidate using `canMoveTo(p, tx, ty, state)` – this checks board bounds *and* path clearance (single‑step slide).

Using empties as the expansion frontier drastically reduces the branching factor compared with naïvely trying to move every piece in four directions.

---

## 4. Breadth‑First Search Solver

```js
function solveFromCurrentBoard() {
  const startKey = encodeBoardFast(pieces);
  const queue = [{ pieces: pieces, path: [] }];
  const seen  = new Set([startKey]);
  let head = 0;
  while (head < queue.length) {
    const {pieces: cur, path} = queue[head++];
    if (isGoal(cur)) return path;           // reached Cao Cao at (1,3)
    for (const nxt of nextPositionsFast(cur)) {
      const key = encodeBoardFast(nxt.pieces);
      if (seen.has(key)) continue;
      seen.add(key);
      queue.push({ pieces: nxt.pieces, path: [...path, nxt.move] });
    }
  }
  return null;  // unsolved – theoretically unreachable for valid boards
}
```

- **Queue with numeric **`` gives *O(1)* pop without expensive `shift()`.
- Path is stored as an array of incremental moves `{id,x,y}` which later drives the animation.
- Time‑complexity is exponential but highly pruned; the classic board solves in ≈ 100 ms in modern browsers.

---

## 5. Solution Playback

`startSolution()` freezes the UI (`isSolving = true`), then schedules `playSolutionStep(0)` after a short delay.  Each step:

1. Calls `executeMove()` – updates `pieces` **and** re‑renders.
2. Enqueues the next step with `setTimeout` (1 s interval) so users can watch.
3. If the move is invalid (should never happen), playback halts and an i18n‑aware error alert appears.

Stopping (via `stopSolution()` or reset) clears the timer, re‑enables UI, and removes the CSS class that hides pointer events.

---

## 6. Internationalisation (i18n)

A simple `translations` object maps language code → UI strings.  `changeLanguage(lang)` rewrites:

- Page `<title>` & `<h1>`
- Button labels
- Layout dropdown labels
- Run‑time alerts via `showAlert(key, …args)`

Adding a new language only requires appending a dictionary.

---

## 7. Running Locally

1. Save the HTML file as `klotski.html`.
2. Open it in any modern desktop or mobile browser – no build step needed.

> **Tip:** On mobile Safari/Chrome you can “Add to Home Screen” for an app‑like full‑screen experience.

---

## 8. Extending the Code

| Task                                   | How‑to                                                                                         |
| -------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Add a new layout**                   | Append an entry to `layouts` with a unique key and piece array; translation label is optional. |
| **Change board size** *(experimental)* | Update `gridSize`, CSS grid rows/cols, bounds in `canMoveTo`, and goal test.                   |
| **Faster solver**                      | Replace BFS with A\* and a heuristic (e.g., Manhattan distance of Cao Cao to exit).            |
| **Save/Load progress**                 | Serialize `pieces` to `localStorage` on `beforeunload`; hydrate on page load.                  |

---

## 9. License

This demo is released into the public domain (CC0).  Use freely in personal or commercial projects – attribution appreciated but not required.

