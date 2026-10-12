# Notes

## What is in CLAUDE.md, and what I left out

I kept four things: a one-line description of the project, the commands I run (`npm run dev`, `npm test`, `npm run lint`, plus how to run a single test file or a single test by name), a short architecture section, and the conventions.

The architecture section only covers what is not obvious from a quick look at the files: `server.js` exports `app` and only calls `listen()` when run directly, which is why tests can import it without opening a port, and `db/store.js` is the only place that holds data, in memory, shared across tests in the same process.

The conventions are written as rules Claude can follow without asking: CommonJS only (ESLint fails on `import`/`export`), route handlers go through `db/store.js` instead of touching data, errors are `res.status(code).json({ error })`, creates return `201`, path params are converted with `Number()`, and tests use `node:assert` with no extra framework.

I deliberately left out:

- The list of endpoints and dependencies. Claude can read `routes/` and `package.json`, and a copy in `CLAUDE.md` would go stale.
- Setup steps from the README (cloning, opening a terminal). They are for a person, not for every Claude session.
- Anything from `.env`. Secrets do not belong in a file that is committed and loaded into every conversation.
- One-off tasks and generic advice like "write clean code". Neither changes what Claude does on this project.

The file is loaded into every session, so each line costs context. I only kept lines that would change Claude's behaviour.

## Permission rules

In `.claude/settings.json`:

- **allow** `Bash(npm test:*)`: I run tests constantly and they are safe, so I do not want a prompt each time.
- **ask** `Bash(git push:*)`: pushing is visible to other people, so I want to confirm it every time.
- **deny** `Read(./.env)`: Claude never needs real secrets to work on this code.
- **deny** `Bash(git push --force:*)`: there is no case in this repo where I want Claude to rewrite remote history.

Without the `.env` deny rule, Claude could read real credentials (for example a `DATABASE_URL` with a password) while exploring the project. They would then be in the conversation and could end up echoed into output, a commit, or a log. Without the force-push deny rule, one wrong command could overwrite commits on the remote, including other people's work, with no easy way back.

## Verification

- `/memory` in a fresh session shows the project `CLAUDE.md` loaded.
- `/permissions` shows the allow, ask and deny rules above.
- I asked "How do I run the tests here?" and Claude answered `npm test`, with the single-file and single-test variants, straight from `CLAUDE.md` without me explaining anything.
