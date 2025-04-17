# Klotski Game (华容道游戏) - Design and Algorithm Documentation

## Overview
This project implements the Klotski puzzle game (known as 华容道 in Chinese), a sliding block puzzle where the objective is to move the largest block, representing Cao Cao, to a specific exit position at the bottom center of the board. The game includes multiple layouts, interactive drag-and-drop mechanics, and an automated solution demo using a breadth-first search (BFS) algorithm.

## Game Description
Klotski is a classic sliding puzzle game often themed around the historical narrative of Cao Cao's escape during the Battle of Red Cliffs. The game board consists of a 4x5 grid where various shaped pieces (1x1, 1x2, 2x1, and 2x2) can be slid horizontally or vertically into empty spaces. The goal is to maneuver the 2x2 piece (Cao Cao) to the exit at the bottom middle of the board (positions x=1, y=3).

## States Design
The game manages several key states to handle user interaction and automated solving:

- **Piece State**: Each piece on the board is represented by an object with properties for position (`x`, `y`), size (`w`, `h`), and a unique identifier (`id`). The state of all pieces collectively represents the current board configuration.
- **Layout State**: The game supports multiple predefined layouts (e.g., Classic, Besieged, Corner Escape), each with a unique initial arrangement of pieces. The current layout is tracked to allow resetting and switching between different puzzle configurations.
- **Interaction State**: Flags like `isDragging` track whether a user is currently moving a piece, preventing multiple simultaneous interactions. Variables store the active piece and starting positions for drag-and-drop or touch interactions.
- **Solving State**: A flag `isSolving` indicates whether the automated solution demo is running. During this state, user interactions are disabled, and the board visually indicates a solving mode with a wait cursor.
- **Solution Sequence State**: When a solution is computed, the sequence of moves is stored in `solutionSequence`, which is played back step-by-step during the demo with a timed delay between moves.

## Algorithm for Solution
The automated solution uses a **Breadth-First Search (BFS)** algorithm to find the shortest path from the current board state to the goal state (Cao Cao at x=1, y=3). Here's a detailed breakdown of the algorithm as implemented:

1. **Board Encoding**:
   - The board state is encoded into a compact string representation (20 characters for a 4x5 grid) where each cell is represented by a character (e.g., 'C' for Cao Cao, 'h' for horizontal 2x1 pieces, 'v' for vertical 1x2 pieces, 's' for soldiers, and '.' for empty spaces).
   - To optimize and avoid duplicate states, the left-right mirror of the board is computed, and the lexicographically smaller encoding is used as the canonical representation. This reduces the state space by recognizing symmetrical configurations.

2. **State Generation (Neighbor Positions)**:
   - For each board state, the algorithm identifies the two empty spaces on the board (Klotski always has exactly 2 empty cells in a 4x5 grid with the given piece sizes).
   - It then checks adjacent cells to these empty spaces to find pieces that can be moved into an empty spot with a single slide.
   - For each possible move, it ensures the move is valid (within bounds and not blocked by other pieces) using the `canMoveTo` function, then generates a new board state by cloning the current state and updating the moved piece's position.
   - Duplicate moves are avoided by tracking attempted moves for each piece in the current iteration.

3. **BFS Implementation**:
   - The BFS starts from the current board state and explores all possible moves level by level to guarantee the shortest path to the goal.
   - A queue stores states to explore, where each entry includes the board configuration (pieces array) and the path of moves taken to reach that state.
   - A `Set` tracks visited states (using the encoded string) to avoid cycles and redundant exploration.
   - The search continues until the goal state is reached (Cao Cao at x=1, y=3), at which point the path of moves is returned as the solution.
   - If no solution is found, it returns null (though this should not occur for valid Klotski puzzles).

4. **Solution Playback**:
   - Once the solution path is computed, it is stored in `solutionSequence`.
   - The `playSolutionStep` function iterates through the sequence, executing each move with a 1-second delay between steps using `setTimeout`.
   - During playback, user interaction is disabled by setting `isSolving` to true and applying a CSS class to prevent pointer events.
   - Playback can be stopped manually via the reset function, which clears the timeout and resets the state flags.

## Key Features
- **Multiple Layouts**: Players can choose from various puzzle configurations, each presenting a different challenge.
- **Interactive Controls**: Supports both mouse and touch input for dragging pieces, with visual feedback during interaction.
- **Solution Demo**: An automated solver visually demonstrates the shortest path to the solution using BFS.
- **Multilingual Support**: The interface supports Chinese and English, with translated text for all UI elements and messages.
- **Reset Functionality**: Allows reverting to the initial state of the current layout at any time.

## Technical Details
- **Grid System**: The board uses a 4x5 grid with each cell being 90x90 pixels. Pieces are positioned absolutely using CSS `top` and `left` properties for smooth transitions during moves.
- **Move Validation**: The `canMoveTo` function checks if a move is valid by ensuring the target area is within bounds, unobstructed, and reachable via sliding (though strict single-step sliding is not always enforced in the current implementation).
- **Performance Optimization**: BFS uses an efficient queue with O(1) pop operation (via array indexing) and minimizes state duplication by using canonical board encodings.

## Usage
- Open the `index.html` file in a web browser to play the game.
- Use the dropdown menus to switch languages or select different layouts.
- Drag pieces to move them, click "Reset" to start over, or click "Show Solution (Demo)" to watch the automated solver.
- The "Help" button provides instructions on how to play.

## Conclusion
This implementation of Klotski combines an intuitive user interface with a robust BFS-based solver to provide both an engaging puzzle experience and an educational demonstration of algorithmic problem-solving. The state management ensures smooth transitions between interactive play and automated demos, making it accessible to players and learners alike.
