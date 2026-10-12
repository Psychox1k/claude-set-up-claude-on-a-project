# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express 4 REST API (users + health check) backed by an in-memory store. It is the starter repo for the Claude Code course; later course projects build on it.

## Commands

```
npm install
npm run dev      # node --watch server.js, serves http://localhost:3000 (PORT overrides)
npm test         # node --test (built-in runner, no Jest/Mocha)
npm run lint     # eslint .
```

- Single test file: `node --test tests/users.test.js`
- Single test by name: `node --test --test-name-pattern="POST /users"`
- CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22; run both before finishing a change.

## Architecture

- `server.js` builds the Express app, mounts each router under its path prefix, and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which is what lets tests import the app without opening a port.
- `routes/` has one file per resource. Each exports an `express.Router()` whose paths are relative to the prefix given in `server.js`.
- `db/store.js` is the only place that holds data: module-level arrays plus exported functions (`getAllUsers`, `getUserById`, `createUser`). Nothing is persisted; state resets on restart and is shared across tests in the same process.
- `tests/` uses `node:test` + `node:assert` with `supertest` against the imported `app`.

## Conventions

- CommonJS only (`require` / `module.exports`). ESLint is configured with `sourceType: "script"`, so `import`/`export` fails lint.
- Route handlers never touch data directly; add a function to `db/store.js` and call it from the route.
- Adding a resource means a new `routes/<resource>.js` plus an `app.use("/<resource>", ...)` line in `server.js`.
- Errors are returned as `res.status(<code>).json({ error: "<message>" })`; creates return `201` with the created object.
- Path params arrive as strings; convert with `Number(req.params.id)` before passing to the store, which compares ids with `===`.
- Put a `// METHOD /path — what it does` comment above each route handler.
- Tests assert with `assert.strictEqual` / `assert.ok` from `node:assert`; don't add a test framework or assertion library.
