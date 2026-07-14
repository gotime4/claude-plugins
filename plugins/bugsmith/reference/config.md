# Configuration

bugsmith auto-detects most things, but every repo-specific command and convention
is overridable via a `bugsmith.config.json` at the repo root. Schema:
`../schemas/bugsmith-config.schema.json`. Starter: `../templates/bugsmith.config.json`.

Resolution order for any value: **CLI flag → `bugsmith.config.json` → host
`CLAUDE.md` conventions → auto-detected default.**

## Fields

```jsonc
{
  "baseBranch": "main",

  "branch": {
    // {id} = ticket id, {slug} = kebab title, {type} = commit type
    "pattern": "{prefix}-{id}-{slug}",
    "prefix": "CU"                 // e.g. ClickUp custom-id prefix
  },

  "commit": {
    "convention": "conventional",  // type(scope): summary
    "scopes": ["auth","calendar","events","ui","api","staff","types","logging"],
    "coAuthorTrailer": "Co-Authored-By: <name> <email>"   // per CLAUDE.md
  },

  "ticket": {
    "provider": "auto",            // auto | clickup | github-issue | raw
    "writeBack": false
  },

  "commands": {
    // Prefer the repo's own skills/scripts. Any of these may be a skill name
    // ("skill:pre-pr-check"), a shell command, or omitted (use built-in).
    "test":       "npx vitest run {files}",
    "testBackend":"go test ./{pkg}/...",
    "lint":       "npx eslint {files} --config eslint.config.js",
    "typecheck":  "npx vue-tsc --noEmit",
    "build":      "yarn build-dev",
    "prePr":      "skill:pre-pr-check",     // or ./scripts/pre-pr-check.sh
    "review":     "skill:code-review",       // the CodeRabbit-mimicking skill
    "lintSkill":  "skill:linting"
  },

  "e2e": {
    "enabled": true,               // false = same as always passing --no-e2e
    "affectedTagsCmd": "npx tsx e2e/scripts/get-affected-tags.ts --files",
    "devServer": {
      "startCmd": "node node_modules/vite/bin/vite.js --mode staging --port {port}",
      "port": 5050,
      "readyLog": "ready in",
      "reuseExisting": true
    },
    "run": "TEST_ENV=stage WEBADMIN_URL=http://localhost:{port} npx playwright test {selectors}",
    "credEnv": ["E2E_CREDENTIALS_KEY"],      // presence gates whether E2E can run
    "browserInstallCmd": "npx playwright install chromium"
  },

  "repos": {
    // Cross-repo map for backend/sibling fixes and backend tests.
    "backend": {
      "name": "vega-cloud",
      "path": "../vega-cloud",
      "testCmd": "firebase emulators:exec 'go test -p=1 ./{pkg}/...'",
      "requires": ["jdk21", "firestore-emulator"]   // preflight; skip+alert if absent
    }
  },

  "gates": {
    // Which steps may skip-with-alert vs must-block. Verification failures always
    // block; these govern *couldn't-run* handling.
    "e2eMissingEnv": "skip",       // skip | block
    "backendMissingToolchain": "skip"
  }
}
```

## Auto-detection (no config file)

- **baseBranch**: the repo's default branch.
- **commands**: detect from `package.json` scripts (`test`, `lint`, `build`,
  `type-check`) and the presence of `scripts/pre-pr-check.sh`, a `code-review` /
  `linting` / `pre-pr-check` skill, and `e2e/scripts/get-affected-tags.ts`.
- **branch prefix / commit scopes / co-author trailer**: read from the host
  `CLAUDE.md` if it documents them (vega repos do).
- **e2e**: enabled only if an affected-tags script and a cred env var are both
  present; otherwise default `--no-e2e` with a note.

## Interaction with `CLAUDE.md`

The host repo's `CLAUDE.md` is authoritative for conventions it states (commit
format, scopes, the "never push to main" rule, co-author trailer, test locations).
bugsmith reads it in Step 0 and treats it as higher priority than its own defaults
but lower than an explicit `bugsmith.config.json` value.
