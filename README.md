# SzorzoKraft - Multiplication Learning Game

2D browser game that helps kids practice multiplication tables (2x2 through 9x9) through resource gathering and building.

## How It Works

Players explore a randomly generated top-down world filled with trees and rocks. Every interaction — chopping a tree, mining a stone, or placing a building — requires solving a multiplication problem before the action completes. There is no time limit on answers, so players can think at their own pace.

## Game Mechanics

### Movement
- **Desktop**: WASD or arrow keys
- **Mobile**: Swipe in any direction

### Resource Gathering
Walk next to a tree or rock and collect it (click/tap it, press Space, or use the "Gyujtes" button). A multiplication question appears — answer correctly to receive the resources:
- **Tree**: gives 3 Wood
- **Rock**: gives 3 Stone

### Building
Use collected resources to build structures. Each build also requires solving a multiplication problem:

| Structure | Cost | Key |
|-----------|------|-----|
| Wall (Fal) | 3 Stone | 1 |
| Floor (Padlo) | 2 Wood | 2 |
| Door (Ajto) | 2 Wood + 1 Stone | 3 |
| Roof (Teto) | 3 Wood + 2 Stone | 4 |

Select a building type, then click/tap an empty tile to place it. Press Escape or the "Megse" button to cancel.

### Zombie
A zombie roams the world. When the player walks next to it, a **duel** begins:
- A timed multiplication problem appears (15–30 second limit)
- **Solve it in time** → zombie is defeated and respawns far away
- **Time runs out** → lose 1 Wood and 1 Stone; zombie respawns far away
- There is always exactly one zombie on the map

### Multiplication Questions
- Random problems from the 2-9 range (e.g. 3 x 7 = ?)
- No time pressure on resource/building problems — players can take as long as they need
- Zombie duels are timed (see above)
- Wrong answers prompt a retry with the same problem

### Focus Mode
The hamburger menu contains a "Fókusz" submenu where players can focus on a specific multiplication table:
- **Mind** (default): random problems from the full 2-9 range
- **2–9**: questions always include the selected number as one operand (e.g. selecting 3 gives problems like 3 × 7, 5 × 3, etc.)

The focus setting is saved with the world and persists across sessions.

### World Persistence
The game automatically saves to localStorage after every action (moving, collecting, building). Returning to the page restores the previous session.

### Regenerate
The hamburger menu (top-left) contains the "Uj Vilag" option, which generates a fresh world and resets all inventory.

## Tech
Single-file HTML/CSS/JS app with canvas rendering. No dependencies. Works on desktop and mobile browsers.

## Architecture

```
index.html (single file)
├── HTML
│   ├── <canvas>           — game world rendering
│   ├── #hamburger-menu     — hamburger menu with Uj Vilag, save/load, and Focus submenu (top-left)
│   ├── #ui                — inventory display (top-left, below hamburger)
│   ├── #build-menu        — building buttons [1]-[4] (top-right)
│   ├── #touch-actions     — action buttons: Gyujtes, Megse (bottom)
│   ├── #math-modal        — multiplication challenge overlay
│   └── #message           — toast notifications
│
├── CSS
│   └── Inline <style>     — responsive layout, dark theme, modal styling
│
└── JavaScript
    ├── World
    │   ├── generateWorld()    — creates 60x45 grid with random trees/rocks
    │   ├── saveWorld()        — serializes world + player + inventory to localStorage
    │   └── loadWorld()        — restores saved state on page load
    │
    ├── Rendering
    │   ├── render()           — camera-relative tile drawing loop (requestAnimationFrame)
    │   ├── drawTile()         — renders grass, trees, rocks, walls, floors, doors, roofs
    │   └── drawPlayer()      — directional character sprite
    │
    ├── Input
    │   ├── Keyboard           — WASD/arrows (move), 1-4 (build), Space (collect), Esc (cancel)
    │   ├── Mouse              — click to collect/place, hover for build preview
    │   └── Touch              — swipe to move, tap to collect/place
    │
    ├── Game Logic
    │   ├── movePlayer()       — collision check against tile types
    │   ├── tryCollect()       — find adjacent tree/rock, trigger math challenge
    │   ├── startBuild()       — enter build mode if resources sufficient
    │   ├── tryPlace()         — place building on empty tile after math challenge
    │   └── cancelBuild()      — exit build mode
    │
    ├── Zombie
    │   ├── spawnZombie()      — places zombie on empty tile far from player
    │   ├── updateZombie()     — random slow movement, adjacency check triggers duel
    │   ├── startDuel()        — timed math challenge (15-30s)
    │   └── endDuel()          — win: respawn zombie; lose: remove resources + respawn
    │
    └── Math Challenge
        ├── showMath()         — generates random AxB (2-9), displays modal
        └── checkAnswer()      — validates input, calls success callback or retries
```

### Data Model

- **world**: 2D array (`WORLD_H x WORLD_W`) of `{ type, grass }` objects
  - `type`: EMPTY(0), TREE(1), STONE(2), WALL(3), FLOOR(4), DOOR(5), ROOF(6)
  - `grass`: 0-3, cosmetic grass variant for the base layer
- **player**: `{ x, y, dir }` — grid position and facing direction (0=up, 1=right, 2=down, 3=left)
- **zombie**: `{ x, y, dir }` — same structure as player; one always present on the map
- **inventory**: `{ wood, stone }` — resource counts

### Game Flow

```
Player Action → Math Modal → Correct Answer → State Change → saveWorld() → updateUI()
```

Every meaningful action (collect, build) is gated by a multiplication problem. Movement is free but still persisted to localStorage.
