# Repository Guidelines

## Project Structure & Module Organization
- `client/`: Unity client project (`Assets/`, `ProjectSettings/`, `Packages/`). Build tooling lives in `client/buildscripts/`.
- `games/`: TypeScript game server and simulations. Source is in `games/src/` (entry `games/src/GAMES/server.ts`), Lua scripts in `games/res/lua/`, infra in `games/docker-compose.yml` and `games/helm/`.
- `server/`: Node/TypeScript backend services. Source is in `server/src/` with `LOGIC/`, `ADMIN/`, `PUBLIC/`, `WORKER/`, and `admin_web/`. Tests live under `server/test/`.
- Generated output: `games/gen/` and `server/gen/` are build artifacts and should not be edited.

## Build, Test, and Development Commands
- Games server:
  ```bash
  cd games
  npm ci
  npm run tsc
  docker-compose up
  ```
  Use `npm run start:debug` for local debugging (Node inspector on 9229) and `npm run lint` for ESLint.
- Backend server:
  ```bash
  cd server
  npm ci
  npm run dist-ts
  npm run logic
  ```
  Admin web dev server: `npm run dev-admin-web`. Production bundle: `npm run dist-admin-web`.
- Unity client: open `client/` with Unity 2018.3.8f1. Example build script:
  ```bash
  bash client/buildscripts/generate_android_project.sh -o /tmp/cv-android DEV
  ```

## Coding Style & Naming Conventions
- EditorConfig enforces LF, a final newline, and 2-space indentation for JS/TS.
- `games/` uses ESLint (`games/.eslintrc.js`); semicolons are required.
- Match existing file patterns, for example `blackjack_classic.ts` with paired `blackjack_classic.value.ts` and `blackjack_classic.interface.ts`.

## Testing Guidelines
- Server tests use mocha and live in `server/test/functional_test/` and `server/test/integration_test/` with `test_*.ts` naming.
- Run the full suite with `cd server && npm test` (compiles `server` and `games`, starts Docker services, loads SQL from `server/test/initial_data`).
- There is no shared test runner for `games/`; use targeted scripts or `ts-node` for simulations when needed.

## Commit & Pull Request Guidelines
- Git history is not available in this checkout; use concise, imperative commit summaries. If working in `games/`, follow `games/AGENTS.md` conventions (short summary with optional game tag like `[EWS]`).
- PRs should include: summary, impacted area (`client/`, `games/`, `server/`), test commands run, and screenshots for client/UI changes. Link issues and call out config or data updates.

## Security & Configuration Tips
- Do not commit secrets. Use environment variables or docker-compose configs.
- Key configs live in `games/src/config.ts` and `server/src/config.ts`. Note required env updates in your PR.
