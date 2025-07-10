# Flip 7 Scoring App - Design Document

## Overview

This document outlines the features, logic, and structure for a Flip 7 scoring app. The goal is to create a minimalist,
real-time scoring application that accommodates the unique game mechanics of Flip 7, including number cards, modifiers,
action resolution, and round-end detection.

---

## Game Rules Overview

Flip 7 is a card game where:
- Players try to get the highest score by playing number cards (0-12)
- Players can enhance scores with modifier cards (+2, +4, +6, +8, +10, x2)
- Getting exactly 7 cards gives a 15-point bonus and ends the round
- Players can "stay" to keep their current score
- Exceeding a certain threshold "busts" a player (score = 0)
- Some action cards allow special moves (Freeze, Second Chance, etc.)

---

## Key Features

1. **Quick Score Input**

    - Interface for selecting number cards from 0 to 12.
    - Card selections visually displayed in a player's card row.
    - Prevent duplicate number entry unless a "Second Chance" card is active.

2. **Smart Modifiers**

    - Add modifiers: `+2`, `+4`, `+6`, `+8`, `+10`.
    - Include `x2` multiplier for number card total only.
    - Modifier cards are displayed above number cards.

3. **Flip 7 Bonus Detection**

    - Automatically detect when a player reaches exactly 7 number cards.
    - Apply a 15-point bonus.
    - Trigger end-of-round for all players.

4. **Multi-Player Support**

    - Add/edit/delete multiple players.
    - Track player state: active, frozen, busted, stayed.
    - Track round score and total score.

5. **Real-Time Leaderboard**

    - Sort players by total score dynamically.
    - Show current round score, player status, and highlight leader.

6. **Undo Functionality**

    - Maintain a history stack per player.
    - Enable undo of last card added, modifier applied, or status change.

7. **Minimalist UI Design**

    - Clean layout using cards as buttons or drag/drop targets.
    - Sections for number cards, modifier cards, player actions.
    - Buttons: `Add Card`, `Stay`, `Undo`.

---

## Technology Stack

- **Frontend**: React Native (for cross-platform mobile support)
- **State Management**: Redux or Context API
- **Styling**: Styled Components or Tailwind CSS
- **Persistence**: AsyncStorage for local storage
- **Testing**: Jest and React Testing Library

---

## Project Structure

```
flip7/
├── src/
│   ├── components/
│   │   ├── Card/
│   │   ├── Player/
│   │   ├── Scoreboard/
│   │   └── ActionBar/
│   ├── screens/
│   │   ├── GameScreen.js
│   │   ├── SettingsScreen.js
│   │   └── StatsScreen.js
│   ├── hooks/
│   │   ├── useGameState.js
│   │   └── usePlayerActions.js
│   ├── utils/
│   │   ├── scoring.js
│   │   └── gameLogic.js
│   ├── context/
│   │   └── GameContext.js
│   └── App.js
├── tests/
└── assets/
    └── cards/
```

---

## Data Models

```typescript
interface Player {
    id: string;
    name: string;
    numberCards: number[];
    modifiers: string[];  // ["+4", "x2"]
    hasX2: boolean;
    secondChance: boolean;
    isFrozen: boolean;
    isBusted: boolean;
    stayed: boolean;
    totalScore: number;
    roundScore: number;
    actionHistory: Action[];  // for undo
}

interface Action {
    type: 'ADD_CARD' | 'ADD_MODIFIER' | 'STAY' | 'BUST' | 'FREEZE';
    value?: number | string;
    timestamp: number;
}

interface GameState {
    players: Player[];
    currentRound: number;
    isRoundActive: boolean;
    roundHistory: RoundSummary[];
}

interface RoundSummary {
    roundNumber: number;
    playerScores: {playerId: string, score: number}[];
    winner: string;
    bonusAchieved: boolean;
}
```

---

## Scoring Logic

```python
def calculate_score(player):
    if player.is_busted:
        return 0

    base_score = sum(player.number_cards)

    score = base_score
    if player.has_x2:
        score *= 2

    for mod in player.modifiers:
        if mod.startswith('+'):
            score += int(mod[1:])

    if len(player.number_cards) == 7:
        score += 15

    return score
```

---

## Round End Conditions

- A player reaches 7 number cards ➝ Flip7 bonus triggers, round ends immediately.
- All players are either frozen, stayed, or busted ➝ round ends normally.

---

## UI/UX Design

### Main Game Screen
```
┌─────────────────────────────────────────┐
│ Round: 3         Settings    New Game   │
├─────────────────────────────────────────┤
│ CURRENT PLAYER: Alice                   │
│ ┌─────────────────────────────────────┐ │
│ │ Modifiers: [+4] [x2]                │ │
│ ├─────────────────────────────────────┤ │
│ │ Cards: [3] [5] [7] [9] [11]         │ │
│ │                                     │ │
│ │ Score: 47                           │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ NUMBER CARDS                        │ │
│ │ [0][1][2][3][4][5][6][7][8][9][10][11][12] │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ MODIFIER CARDS                      │ │
│ │ [+2][+4][+6][+8][+10][x2]           │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ ACTIONS                             │ │
│ │ [STAY][UNDO][NEXT PLAYER]           │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ LEADERBOARD                         │ │
│ │ 1. Alice: 235                       │ │
│ │ 2. Bob: 190                         │ │
│ │ 3. Charlie: 150                     │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

---

## Core Functions

```typescript
// Add a number card to player
function addNumberCard(playerId: string, cardValue: number): void

// Add a modifier to player
function addModifier(playerId: string, modifierValue: string): void

// Player decides to stay
function stayPlayer(playerId: string): void

// Undo last action
function undoLastAction(playerId: string): void

// Check if round has ended
function checkRoundEnd(): boolean

// Calculate and update all scores
function updateScores(): void

// Move to next round
function startNewRound(): void
```

---

## Development Roadmap

### Phase 1: Core Functionality
- Basic UI implementation
- Player management
- Card selection and scoring logic
- Round management

### Phase 2: Enhanced Features
- Undo functionality
- Visual polish and animations
- Settings and preferences
- Game statistics

### Phase 3: Advanced Features
- Action cards implementation
- Multi-device sync (optional)
- AI opponents (optional)

---

## Testing Strategy

- **Unit Tests**: Test scoring logic, game state changes
- **Component Tests**: Test UI components in isolation
- **Integration Tests**: Test full game flows
- **User Testing**: Gather feedback from actual Flip 7 players

---

## Optional Enhancements (Future)

- Add visual deck size / card counter.
- Implement action card logic: Freeze, Flip Three, Second Chance.
- Sync with online players or allow shared scoring sessions.
- Add game statistics and history
- Theme options (dark mode, card styles)
- Tutorial mode for new players

---

## Conclusion
This design document serves as a comprehensive guide for developing the Flip 7 scoring app. It captures the essential
features, game logic, and user interface requirements to create a functional and enjoyable digital companion for the Flip 7 card game.

# Flip 7 Scoring App - Design Document

---

## Design Credit

This design was structured based on the official Flip 7 game rules by Eric Olsen and includes game-enhancing logic for
digital play.
