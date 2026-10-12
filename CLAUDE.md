# CLAUDE.md

Small Express (Node.js, CommonJS) REST API serving `/users` and `/health` from an in-memory store.

## Commands

- `npm run dev` — start the API on http://localhost:3000 with auto-reload (`PORT` overrides)
- `npm test` — run all tests (Node's built-in `node:test` runner)
- `npm test -- --test-name-pattern="<name>"` — run a single test by name
- `npm run lint` — ESLint; must pass with no errors (CI runs lint, then tests, on Node 22)

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules (`import` / `export`).
- Use double quotes and semicolons, matching the existing files.
- Routes never touch data directly: read and write through functions exported from `db/store.js`; add a new store function rather than reaching into its arrays.
- New resource → new file `routes/<resource>.js` exporting an `express.Router()`, mounted in `server.js` with `app.use("/<resource>", ...)`.
- Errors are JSON `{ "error": "<message>" }` with the right status: 400 for invalid input, 404 for not found; 201 for successful creates.
- Tests use `node:test` + `node:assert` + `supertest` against the exported `app` — not Jest/Mocha, and never start a real server in tests. Test files live in `tests/` as `*.test.js`.
- Add or update a test in `tests/` for every route change, then run `npm run lint` and `npm test` before calling a task done.
- Never read, print, or commit `.env`; document new config keys in `.env.example` instead.

## Architecture

- `server.js` — entry point: builds the app, mounts routers, exports `app`; only calls `listen()` when run directly so tests can import it.
- `routes/` — one router file per resource (`users.js`, `health.js`).
- `db/store.js` — in-memory data helper standing in for a database; data resets on every restart.
- `tests/` — HTTP-level tests that import `app` from `server.js`.
