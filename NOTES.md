# NOTES

## What's in `CLAUDE.md`, what's left out, and why

I started from `/init`-style output and cut it down to the four parts that change how Claude works here: a one-line description, the commands (`npm run dev`, `npm test`, running a single test, `npm run lint`), conventions, and a short architecture map. The conventions are the things Claude would otherwise guess wrong or have to ask about: CommonJS rather than ES modules, routes go through `db/store.js` instead of touching data, one router file per resource mounted in `server.js`, the `{ error }` JSON shape with 400/404/201, and `node:test` + `supertest` instead of Jest. Each one is a "do X, not Y" rule Claude can follow without asking. I also wrote down the "done" bar (lint + tests pass, a test for every route change) because CI enforces it.

I deliberately left out things Claude can read from the code in seconds (every endpoint, the seeded users, the package versions, the full ESLint config), the README's setup walkthrough, and anything course- or task-specific, since those would go stale or bloat every session's context. There are no secrets, URLs with credentials, or `.env` values in it — just a rule never to read or commit `.env`.

## Permission rules, and what the deny rules protect

- **Allow:** `npm test`, `npm run lint`, and read-only git (`status`, `diff`, `log`). These run constantly, are safe, and can't change anything outside the working tree, so prompting for them is just friction.
- **Ask:** `git push` and `npm install`. Pushing publishes work and installing pulls third-party code (with install scripts) from the network, so I want to confirm each time.
- **Deny:** reading/editing `.env` (and `cat .env`), `git push --force` / `-f`, `git reset --hard`, and `rm -rf`.

Without the `.env` deny rule, Claude could read real secrets (e.g. a `DATABASE_URL` with a password) into the conversation while "debugging config", where they could end up in logs, summaries, or a pasted snippet. This matters even more in this repo: the `.gitignore` entries for `.env` and `.claude/settings.local.json` are indented with spaces, and git treats leading spaces as part of the pattern, so **neither file is actually ignored right now** (`git check-ignore .env` matches nothing). A `git add .` would commit them. I didn't change it because this task is setup-only, but it should be fixed by removing the leading whitespace. The force-push / hard-reset / `rm -rf` denies stop a single bad command from destroying shared history or uncommitted work, which can't be undone.

## Verification

- `claude --version` → `2.1.292 (Claude Code)`; signed in.
- `npm test` (4/4 passing) and `npm run lint` (clean) both run.
- [ ] In a fresh `claude` session, `/memory` lists the project `CLAUDE.md` as loaded.
- [ ] `/permissions` shows the allow / ask / deny rules from `.claude/settings.json`.
- [ ] "How do I run the tests here?" is answered with `npm test` straight from `CLAUDE.md`.

The `.env` deny rules match `.env` exactly, so Claude can still read `.env.example` to document new config keys.
