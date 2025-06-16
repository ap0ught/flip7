# Flip 7 Scoring App - Design Document

## Overview

This document outlines the features, logic, and structure for a Flip 7 scoring app. The goal is to create a minimalist,
real-time scoring application that accommodates the unique game mechanics of Flip 7, including number cards, modifiers,
action resolution, and round-end detection.

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

## Data Models

```python
class Player:
    name: str
    number_cards: List[int]
    modifiers: List[str]  # ["+4", "x2"]
    has_x2: bool
    second_chance: bool
    is_frozen: bool
    is_busted: bool
    stayed: bool
    total_score: int
    round_score: int
    action_history: List[dict]  # for undo
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

## UI Layout

```
[ Player Tabs / Dropdown ]
┌────────────────────┐
│ Number Cards Row   │
└────────────────────┘
┌────────────────────┐
│ Modifier Cards Row │
└────────────────────┘
[ Add Card ] [ Stay ] [ Undo ]
[ Leaderboard Display with Total Scores ]
```

---

## Optional Enhancements (Future)

- Add visual deck size / card counter.
- Implement action card logic: Freeze, Flip Three, Second Chance.
- Sync with online players or allow shared scoring sessions.

---

## Suggested Commit Message

```bash
Add Flip 7 scoring logic and design structure with UI and model definitions 🎴🧮✨
```

---

## Design Credit

This design was structured based on the official Flip 7 game rules by Eric Olsen and includes game-enhancing logic for
digital play.

