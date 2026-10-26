# CLAUDE.md

A small Express REST API (users + health check) backed by an in-memory store, used as the starter project for a Claude Code course.

## Commands

- `npm run dev`: start the API with auto-reload on http://localhost:3000 (`PORT` overrides it)
- `npm test`: run all tests with Node's built-in runner (`node --test`)
- `npm run lint`: ESLint (`eslint:recommended`, ESLint 8 with `.eslintrc.json`)

CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22, so both must pass.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ESM `import`. ESLint is set to `sourceType: "script"`.
- Write tests with `node:test` + `node:assert` + `supertest` against the exported `app`, not Jest/Mocha, and never call `app.listen` in tests.
- Route handlers access data only through functions exported from `db/store.js`, never by holding the arrays directly.
- Return errors as JSON `{ "error": "<message>" }` with the matching status code (400 validation, 404 not found); return 201 on create.
- Convert `req.params.id` with `Number(...)` before lookups, since store IDs are numbers.

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`/users`, `/health`), and exports `app`. It only calls `listen` when run directly (`require.main === module`), so tests can import it without opening a port.
- `routes/<resource>.js` holds one `express.Router()` per resource. A new resource means a new file here plus an `app.use` line in `server.js`.
- `db/store.js` is an in-memory stand-in for a database. Data resets on restart and is shared module state, so tests in the same file see each other's writes.
- Config comes from env vars; `.env.example` documents them. `.env` is git-ignored and must not be read or committed.
