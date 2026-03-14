# Club Vegas Slots - Project Context

## 1. Project Overview
This repository contains the source code for **Club Vegas Slots**, a large-scale mobile slot game. It is a monorepo consisting of a Unity client, a specialized Game logic server, and a Backend server.

## 2. Directory Structure & Documentation
For detailed context on each sub-project, please refer to their specific `GEMINI.md` files:

- **[Client (Unity)](client/GEMINI.md)**: `client/`
    - The mobile game client built with Unity and the "SlotMaker" framework.
    - Handles rendering, user input, and visual game flow.
- **[Games Server (Logic Engine)](games/GEMINI.md)**: `games/`
    - A specialized, stateless Node.js/TypeScript engine.
    - Responsible for calculating spin results, RNG, and game simulations.
- **[Backend Server (Services)](server/GEMINI.md)**: `server/`
    - The main backend infrastructure (Node.js/TypeScript).
    - Handles user accounts, economy, persistent storage (MySQL/Redis), and API routing.

## 3. High-Level Architecture
1.  **Client** connects to **Backend Server (LOGIC)**.
2.  **Backend Server** authenticates the user and handles the request (e.g., "Spin").
3.  **Backend Server** delegates the specific game math calculation to the **Games Server** (or internal logic library).
4.  **Games Server** returns the result (Symbols, Win Amount).
5.  **Backend Server** updates the user's wallet and saves the state to DB/Redis.
6.  **Backend Server** responds to **Client** with the result.
7.  **Client** plays the appropriate animations based on the result.

## 4. Common Workflows
- **New Game Integration**: Requires changes in all three projects.
    - `games/`: Add math model and logic.
    - `server/`: (Sometimes) Config updates if new feature types are added.
    - `client/`: Visual implementation and FSM setup.
- **Simulation**: Run simulations in `games/` to verify RTP before implementation in `client`.

## 5. Tools & commands
- **Root**: `npm install` (if applicable for shared tools, though mostly sub-project based).
- **Editors**: VS Code is recommended. Use the workspace file `club-vegas-slots.code-workspace`.