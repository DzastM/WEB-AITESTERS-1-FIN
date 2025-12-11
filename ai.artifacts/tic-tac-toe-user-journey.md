# Tic-Tac-Toe Game: Actual User Journey & Test Plan

## 1. Page Analysis

### Page URL and Title
- **URL**: `http://localhost:3000/games/tic-tac-toe-hot-seat.html`
- **Page Title**: 🦎 GAD | Game: Tic Tac Toe (Hot Seat)
- **Version**: v2.8.8

### Initial Page State Description
The page loads with a centered layout displaying:
- A "Start New Game" button (blue, primary style)
- A message label showing "..." (placeholder text)
- An empty 3x3 game board (grid layout with gaps)
- Score tracking for both players (Player X: 0, Player O: 0)
- Navigation links to return to the Tic Tac Toe Collection and Games pages
- Standard header with site navigation

### All Visible UI Elements with Descriptions

| Element | Type | ID/Class | Description | Initial State |
|---------|------|----------|-------------|----------------|
| Start New Game | Button | `id="start-button"` | Primary action to initialize the game | Enabled |
| Message Label | Label | `id="messageLbl"` | Displays game status (whose turn, game result) | Shows "..." |
| Game Board | Grid Container | `class="board"` | 3x3 grid layout with 10px gaps | Empty |
| Board Cell (x9) | Grid Cell | `class="cell"` | Individual clickable cells for moves | Empty, cursor: pointer |
| Player X Score | Span | `id="player-x-score"` | Displays X player's total wins | Shows "0" |
| Player O Score | Span | `id="player-o-score"` | Displays O player's total wins | Shows "0" |
| Alert Popup | Div | `id="alertPopup"` | Alert container for notifications | Hidden/empty |
| Return to Collection Link | Link + Button | Href: `./tic-tac-toe-collection.html` | Navigation button | Visible |
| Return to Games Link | Link + Button | Href: `./games.html` | Navigation button | Visible |

---

## 2. Game Setup Process

### Step-by-Step Instructions to Start a Game

**This is a "Hot Seat" mode game with no player setup required. The game is ready to play immediately.**

### Required Inputs
- **No player name inputs**: The game uses fixed "X" and "O" player designations
- **No difficulty settings**: No game mode selections available
- **No configuration options**: Game starts with default single-player setup against AI or hot-seat mode

### Exact Action Sequence to Start Playing

1. **Navigate to the game page**: User arrives at `http://localhost:3000/games/tic-tac-toe-hot-seat.html`
2. **View initial state**: The page displays an empty board and the "Start New Game" button is enabled
3. **Click "Start New Game"**: User clicks the button with `id="start-button"`
4. **Game state changes**:
   - The "Start New Game" button becomes disabled (`disabled=true`)
   - The message label updates to: "X's turn"
   - Each cell becomes clickable and registers click events (one-time listeners)
   - The board is cleared (in case of replay)
5. **Ready to play**: User can now click any empty cell to make a move

---

## 3. Game Board Structure

### Board Layout and Dimensions
- **Grid Type**: CSS Grid with 3 columns, 3 rows
- **Board Dimensions**: 250px width × 225px height (calculated from: 3 cells × 75px + 2 gaps × 10px)
- **Cell Dimensions**: 75px × 75px
- **Gap Between Cells**: 10px
- **Border**: 1px solid black around each cell
- **Font Size**: 24px for X and O symbols

### Cell Numbering/Identification System

The 9 cells are indexed from 0-8 in the DOM order (top-left to bottom-right):

```
[0] [1] [2]     Row 1, Columns 1-3
[3] [4] [5]     Row 2, Columns 1-3
[6] [7] [8]     Row 3, Columns 1-3
```

**Cell Accessibility Attributes**:
Each cell has the following ARIA attributes:
- `role="gridcell"`
- `tabindex="0"`
- `aria-keyshortcuts="Enter Space"` (keyboard support)
- `aria-rowindex` (1-3)
- `aria-colindex` (1-3)
- `aria-label="Empty cell, row X, column Y"` (updated when filled)

### How Cells Are Selected/Clicked

1. **Mouse Click**: Users click directly on a cell
2. **Event Listener**: Each cell has a one-time click event listener added when the game starts
3. **Handler Function**: Clicking triggers `handleCellClick(event)`
4. **Event Target**: `event.target` is the clicked cell element
5. **Cell Text Update**: The cell's `textContent` is set to the current player's symbol (X or O)

---

## 4. Gameplay Mechanics

### How Moves Are Registered

1. **Click Detection**: When a cell is clicked, the `handleCellClick()` function executes
2. **Symbol Placement**: The current player's symbol (X or O) is immediately written to the cell's textContent: `cell.textContent = currentPlayer;`
3. **State Check**: After placement, the game checks for:
   - **Win condition**: `checkWin()` evaluates all winning combinations
   - **Draw condition**: `checkDraw()` verifies if all cells are filled
   - **Game continuation**: If neither condition is met, the turn switches
4. **One-Time Click**: Each cell can only be clicked once (event listener is registered with `{ once: true }`)
   - Attempting to click an already-filled cell has no effect (no error, no visual feedback)

### Turn Indication Mechanism

- **Display Element**: `messageLbl` (id="messageLbl")
- **Initial Message**: Displays "X's turn" when game starts (X always goes first)
- **Turn Switch**: After each valid move, message updates to: `"{nextPlayer}'s turn"`
- **Update Timing**: Message updates immediately after move validation (before any win/draw checks complete)
- **Live Region**: Label uses `role="status"` and `aria-live="polite"` for accessibility

### Move Validation Rules

| Scenario | Behavior | Validation |
|----------|----------|------------|
| Click empty cell | Symbol placed, turn switches | ✅ Valid |
| Click filled cell | No action, no feedback | ❌ Blocked (one-time listener) |
| Click after win | Board frozen, no moves accepted | ❌ Blocked (listeners removed) |
| Click after draw | Board frozen, no moves accepted | ❌ Blocked (listeners removed) |

### Visual Feedback Provided

- **Symbol Appearance**: X or O appears in the clicked cell (24px font, black text)
- **Turn Message**: Text updates immediately in the message label
- **Cursor**: Cells show `cursor: pointer` when hovered (before game ends)
- **Button State**: "Start New Game" button becomes disabled during gameplay
- **Score Updates**: Player scores update only after a win (not after every move)

---

## 5. Game Outcomes & Rules

### Win Conditions

The game detects a win when the current player occupies all 3 cells in any of these 8 combinations:

**Horizontal Wins**:
- Row 1: Cells [0, 1, 2]
- Row 2: Cells [3, 4, 5]
- Row 3: Cells [6, 7, 8]

**Vertical Wins**:
- Column 1: Cells [0, 3, 6]
- Column 2: Cells [1, 4, 7]
- Column 3: Cells [2, 5, 8]

**Diagonal Wins**:
- Top-left to bottom-right: Cells [0, 4, 8]
- Top-right to bottom-left: Cells [2, 4, 6]

**Win Detection Logic**: 
```javascript
checkWin() returns true when currentPlayer has all 3 cells in any combination
```
- Win is checked **after** each move
- Win ends the current game round immediately
- Winner's score increments by 1

### Draw Conditions

- **Trigger**: All 9 cells are filled AND no player has achieved a winning combination
- **Detection Method**: `checkDraw()` checks if every cell has non-empty textContent
- **Draw Results**: No score increase for either player, game round ends

### How Results Are Displayed

| Outcome | Message Shown | Actions |
|---------|---------------|---------|
| **Win** | *(Not displayed in messageLbl, game resets immediately)* | Score increments, board clears, listeners removed |
| **Draw** | *(Not displayed, game resets immediately)* | No score change, board clears, listeners removed |
| **Ready for Replay** | "Click start to play again" | Start New Game button becomes enabled |

**Note**: There is no explicit "You Win" or "Draw" message displayed. The game automatically resets and the message changes to "Click start to play again", indicating the round has ended.

### How to Restart

1. **Automatic Reset After Win/Draw**: 
   - Board visually clears (all cells' textContent set to "")
   - Message label updates to: "Click start to play again"
   - "Start New Game" button re-enabled
   
2. **Manual Restart**:
   - Click the re-enabled "Start New Game" button
   - Message updates to "X's turn"
   - All cells become clickable again
   - Scores persist across games (cumulative tracking)

---

## 6. Technical Details for Automation

### Key Element Selectors and IDs

| Purpose | Selector | Type | Notes |
|---------|----------|------|-------|
| Start Game Button | `#start-button` | ID | Primary action button, becomes disabled during play |
| Message Label | `#messageLbl` | ID | Shows turn info and game status messages |
| Game Board | `.board` | Class | Container div, `role="grid"` |
| Individual Cells | `.cell` | Class | 9 elements, can be accessed via `querySelectorAll('.cell')` or by index |
| Player X Score | `#player-x-score` | ID | Span element, shows numeric score |
| Player O Score | `#player-o-score` | ID | Span element, shows numeric score |
| Alert Popup | `#alertPopup` | ID | Alert container, `data-testid="alert-popup"` |

### Cell Access Methods for Testing

```javascript
// Access all cells as a NodeList
const cells = document.querySelectorAll(".cell");

// Access specific cell by index (0-8)
const cell = cells[0]; // Top-left
const cell = cells[4]; // Center
const cell = cells[8]; // Bottom-right

// Read cell content
const symbol = cells[0].textContent; // Returns "X", "O", or ""

// Check if cell is empty
const isEmpty = cells[0].textContent === "";
```

### Timing Considerations

| Aspect | Timing | Details |
|--------|--------|---------|
| Move Registration | Immediate | Symbol appears instantly on click (no delay) |
| Turn Switch | Immediate | Message updates immediately after move |
| Win Detection | Synchronous | Checked immediately after move placement |
| Game Reset | Immediate | Board clears instantly on win/draw |
| Click Listener | One-time | Each cell listener fires only once |
| Button State Change | Immediate | Button disabled on game start, enabled on reset |

### Important Element Attributes and Classes

**Button Classes and States**:
- Class: `button-primary` (styling)
- Attribute: `disabled` (boolean, toggles during game)
- Attribute: `aria-controls="board"` (accessibility link to game board)

**Board and Cell Attributes**:
- Board `role="grid"` with `aria-label="Tic Tac Toe board"` and `aria-describedby="messageLbl"`
- Cells have `role="gridcell"`, `aria-rowindex` (1-3), `aria-colindex` (1-3)
- Cell `aria-label` format: `"Empty cell, row X, column Y"` or `"{Symbol} cell, row X, column Y"` (based on content)

**Score Spans**:
- Both use `aria-live="polite"` for live region updates
- Both have descriptive `aria-label` attributes

### HTML Structure Insights for Playwright

```html
<!-- Game Container (centered) -->
<div align="center" role="main">
  <!-- Start Button -->
  <button id="start-button" class="button-primary" onclick="startGame()" 
          aria-controls="board">Start New Game</button>
  
  <!-- Turn/Status Message -->
  <label id="messageLbl" role="status" aria-live="polite">...</label>
  
  <!-- Game Board (3x3 Grid) -->
  <div class="board" role="grid" aria-label="Tic Tac Toe board">
    <!-- 9 Cells (0-8 in DOM order) -->
    <div class="cell" role="gridcell" aria-rowindex="1" aria-colindex="1"></div>
    <!-- ... 8 more cells ... -->
  </div>
  
  <!-- Scores -->
  <div class="score">
    <span>Player X: <span id="player-x-score" aria-live="polite">0</span></span><br/>
    <span>Player O: <span id="player-o-score" aria-live="polite">0</span></span>
  </div>
</div>
```

---

## 7. Test Scenarios Identified

### Scenario 1: Game Initialization
**Objective**: Verify the game starts correctly and is ready for play
- **Steps**:
  1. Navigate to the game page
  2. Verify "Start New Game" button is visible and enabled
  3. Click "Start New Game"
  4. Verify message label shows "X's turn"
  5. Verify "Start New Game" button becomes disabled
  6. Verify board cells are empty and clickable
- **Expected Result**: Game is initialized in correct state, Player X can make first move

### Scenario 2: Valid Move Placement
**Objective**: Verify that clicking an empty cell places the correct symbol and switches turns
- **Steps**:
  1. Start a new game
  2. Click cell [0] (top-left)
  3. Verify "X" appears in cell [0]
  4. Verify message label shows "O's turn"
  5. Click cell [4] (center)
  6. Verify "O" appears in cell [4]
  7. Verify message label shows "X's turn"
- **Expected Result**: Moves are placed correctly, turns alternate properly

### Scenario 3: Invalid Move on Occupied Cell
**Objective**: Verify that occupied cells cannot be clicked and do not accept new symbols
- **Steps**:
  1. Start a new game
  2. Click cell [0] to place X
  3. Immediately click cell [0] again
  4. Verify cell [0] still shows "X" (not "O")
  5. Verify message label still shows "O's turn" (turn did not advance)
- **Expected Result**: Occupied cells are unclickable, no symbol replacement occurs

### Scenario 4: Horizontal Win Detection
**Objective**: Verify the game correctly detects and handles a horizontal win
- **Steps**:
  1. Start a new game
  2. Execute moves: X→[0], O→[3], X→[1], O→[4], X→[2]
  3. Verify cell [2] contains "X"
  4. Verify message label shows "Click start to play again"
  5. Verify "Start New Game" button is enabled
  6. Verify Player X score incremented from 0 to 1
  7. Verify all cells are now empty
- **Expected Result**: Win is detected, score updates, board resets

### Scenario 5: Vertical Win Detection
**Objective**: Verify vertical winning combinations are detected
- **Steps**:
  1. Start a new game
  2. Execute moves to create a vertical win: X→[0], O→[1], X→[3], O→[4], X→[6]
  3. Verify column 1 ([0], [3], [6]) contains all X's
  4. Verify Player X score incremented
  5. Verify game reset automatically
- **Expected Result**: Vertical win detected correctly

### Scenario 6: Diagonal Win Detection
**Objective**: Verify diagonal winning combinations are detected
- **Steps**:
  1. Start a new game
  2. Execute moves for top-left to bottom-right diagonal: X→[0], O→[1], X→[4], O→[2], X→[8]
  3. Verify cells [0], [4], [8] contain all X's
  4. Verify Player X score incremented
- **Expected Result**: Diagonal win (top-left to bottom-right) detected

### Scenario 7: Alternative Diagonal Win
**Objective**: Verify the other diagonal winning combination is detected
- **Steps**:
  1. Start a new game
  2. Execute moves for top-right to bottom-left diagonal: X→[2], O→[1], X→[4], O→[3], X→[6]
  3. Verify cells [2], [4], [6] contain all X's
  4. Verify Player X score incremented
- **Expected Result**: Diagonal win (top-right to bottom-left) detected

### Scenario 8: Draw Game Handling
**Objective**: Verify that a draw is correctly detected when all cells are filled without a winner
- **Steps**:
  1. Start a new game
  2. Execute moves to fill all cells without creating a winning line:
     - X→[0], O→[1], X→[2], O→[3], X→[5], O→[4], X→[6], O→[8], X→[7]
  3. Verify all cells are filled
  4. Verify message label shows "Click start to play again"
  5. Verify neither player's score increased
  6. Verify "Start New Game" button is enabled
- **Expected Result**: Draw detected, no score update, game reset

### Scenario 9: Score Persistence Across Games
**Objective**: Verify that scores accumulate across multiple game rounds
- **Steps**:
  1. Start a new game
  2. Play until X wins (round 1)
  3. Verify Player X score is 1
  4. Click "Start New Game"
  5. Play until O wins (round 2)
  6. Verify Player O score is 1
  7. Verify Player X score remains 1 (not reset)
- **Expected Result**: Scores persist and accumulate across multiple games

### Scenario 10: Multiple Consecutive Games
**Objective**: Verify game state is completely reset between consecutive games
- **Steps**:
  1. Complete a full game (win or draw)
  2. Click "Start New Game"
  3. Verify board is completely empty
  4. Verify message label shows "X's turn"
  5. Verify "Start New Game" button is disabled
  6. Verify all cells are clickable and accept moves
  7. Complete another full game
- **Expected Result**: Each game round is independent, previous board state is fully cleared

---

## Summary

This is a **two-player hot-seat tic-tac-toe game** where:
- Players alternate turns on the same device/keyboard
- Player X always goes first
- No player setup or configuration is required
- Games are played to completion (win or draw), then reset
- Scores persist across games
- The game is fully accessible with ARIA attributes and keyboard support
- All moves and win detection are synchronous with no animations or delays
