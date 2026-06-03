# Tic-Tac-Toe — Web Game (MVP)

A browser-based Tic-Tac-Toe game that two people can play on the same screen.
Self-contained front-end, no backend or accounts — the simplest possible version
to validate the gameplay and UI.

## Acceptance Criteria
- A 3×3 board renders in the browser and is playable with a mouse or touch
- Two players alternate turns (X then O); the current player is clearly shown
- A clicked, empty cell is filled with the current player's mark; filled cells are not reusable
- The game detects a win (three in a row: horizontal, vertical, or diagonal) and announces the winner
- The game detects a draw when the board is full with no winner
- A "New game" button resets the board to an empty state
- The UI is responsive and works on mobile and desktop screen sizes
- Basic accessibility: cells are keyboard-focusable and have ARIA labels

## Out of Scope
- Accounts, persistence, networking, or AI opponents
