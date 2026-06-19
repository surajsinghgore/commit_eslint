# Git Commit Hooks — Setup & Commit Message Rules

A complete, reusable guide to enforce **clean commit messages** and
**auto-lint staged files** before every commit. Drop this into any
Node.js / TypeScript project.

- **Part 1 — Installation & Setup** (Husky + lint-staged + commitlint)
- **Part 2 — Commit message rules** (prefixes, examples, failure scenarios)

---
---

# Part 1 — Installation & Setup

## What you get

| Hook | Tool | What it does |
|------|------|--------------|
| `pre-commit` | **lint-staged** | Runs `eslint --fix` on staged files; blocks the commit on any remaining error |
| `commit-msg` | **commitlint** | Validates the commit message against Conventional Commits rules |
| (optional) `npm run commit` | **commitizen** | Interactive prompt that builds a valid message for you |

## 1. Install dev dependencies

```bash
# npm
npm i -D husky lint-staged @commitlint/cli @commitlint/config-conventional

# pnpm
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional

# yarn
yarn add -D husky lint-staged @commitlint/cli @commitlint/config-conventional
```

Optional interactive commit prompt:

```bash
npm i -D commitizen @commitlint/cz-commitlint
```

## 2. Add the `prepare` script and init Husky

In `package.json`:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

Run once (also runs automatically after every `npm install`):

```bash
npm run prepare   # creates the .husky/ folder
```

## 3. Create the hook files

**`.husky/pre-commit`**

```sh
npx lint-staged
```

**`.husky/commit-msg`**

```sh
npx --no-install commitlint --edit "$1"
```

> Using pnpm? Replace with `pnpm exec lint-staged` and `pnpm exec commitlint --edit "$1"`.

Make them executable:

```bash
chmod +x .husky/pre-commit .husky/commit-msg
```

## 4. Add the config files (project root)

**`commitlint.config.mjs`**

```js
export default {
  extends: ['@commitlint/config-conventional'],
};
```

**`lint-staged.config.mjs`**

```js
export default {
  '*.{js,jsx,mjs,cjs,ts,tsx}': 'eslint --fix',
};
```

## 5. (Optional) commitizen prompt

In `package.json`:

```json
{
  "scripts": {
    "commit": "cz"
  },
  "config": {
    "commitizen": {
      "path": "@commitlint/cz-commitlint"
    }
  }
}
```

Then commit interactively with `npm run commit`.

> ✅ Setup itself is silent — nothing breaks until you make a commit that
> violates a rule. The hooks are a gate, not a background process.

---
---

# Part 2 — Commit message rules

## Format

```
type(scope?): subject

body (optional)

footer (optional)
```

- **type** → the prefix (required, lowercase)
- **scope** → optional area of the codebase, in parentheses (lowercase)
- **subject** → short description (required, starts lowercase, no period)

## commitlint rules (`@commitlint/config-conventional`)

| Rule | Requirement | Level |
|------|-------------|-------|
| `type-enum` | `type` must be one of the allowed values (below) | error |
| `type-case` | `type` must be lower-case (`feat`, not `Feat`) | error |
| `type-empty` | `type` cannot be empty | error |
| `subject-empty` | subject cannot be empty | error |
| `subject-full-stop` | subject must **not** end with `.` | error |
| `subject-case` | subject must **not** be sentence-case, start-case, pascal-case, or upper-case (→ start lowercase) | error |
| `scope-case` | scope must be lower-case | error |
| `header-max-length` | first line ≤ 100 characters | error |
| `body-leading-blank` | blank line between subject and body | warning |
| `body-max-line-length` | each body line ≤ 100 characters | error |
| `footer-leading-blank` | blank line before footer | warning |
| `footer-max-line-length` | each footer line ≤ 100 characters | error |

---

## All prefixes (types) with examples

### `feat` — a new feature

```
feat: add dark mode toggle
feat(auth): support google login
feat(dashboard): add export to CSV button
```

### `fix` — a bug fix

```
fix: prevent crash on empty file upload
fix(cart): correct total price calculation
fix(api): handle 401 response on token expiry
```

### `docs` — documentation only (no code change)

```
docs: update README install steps
docs(api): add examples for the search endpoint
docs: fix typo in contributing guide
```

### `style` — formatting only, no logic change (whitespace, semicolons, commas)

```
style: format files with prettier
style(header): fix indentation
style: remove trailing whitespace
```

> Note: this is **code formatting**, not CSS/UI styling. UI changes are `feat` or `fix`.

### `refactor` — code change that is neither a fix nor a feature

```
refactor: extract validation into a helper
refactor(user): rename getUser to fetchUser
refactor: simplify date-formatting logic
```

### `perf` — performance improvement

```
perf: memoize expensive table calculation
perf(images): lazy-load gallery thumbnails
perf: reduce bundle size by code-splitting
```

### `test` — adding or fixing tests

```
test: add unit tests for login form
test(cart): cover empty-cart edge case
test: fix flaky checkout e2e test
```

### `build` — build system or dependencies

```
build: upgrade to webpack 5
build(deps): bump react to 19
build: add postcss config
```

### `ci` — continuous integration config

```
ci: add github actions workflow
ci: cache node_modules in pipeline
ci(release): run tests before publish
```

### `chore` — maintenance, no src/test change

```
chore: update .gitignore
chore: bump version to 2.1.0
chore(release): 1.4.0
```

### `revert` — undo a previous commit

```
revert: feat: add dark mode toggle
revert: "fix: correct total price calculation"
```

---

## Scope examples (optional)

The scope narrows where the change happened. It must be lowercase.

```
feat(auth): ...
fix(navbar): ...
docs(readme): ...
refactor(api): ...
```

A commit without a scope is perfectly valid:

```
feat: add dark mode toggle
```

---

## Multi-line commit (with body & footer)

```
feat: add password reset flow

Users can now request a reset link from the login page.
The link expires after 30 minutes for security.

Closes #142
```

Rules:
- **Blank line** between subject and body
- **Blank line** before the footer
- Each body/footer line ≤ 100 characters

---

## ❌ Scenarios when the commit FAILS

### commit-msg (commitlint) failures

| # | Bad message | Why it fails | Fix |
|---|-------------|--------------|-----|
| 1 | `update files` | No type prefix | `chore: update files` |
| 2 | `fixed login bug` | No type prefix (`fixed` is not a type) | `fix: login bug` |
| 3 | `Feat: add button` | Type must be lowercase | `feat: add button` |
| 4 | `feature: add button` | `feature` is not a valid type (it's `feat`) | `feat: add button` |
| 5 | `feat: Add button` | Subject starts with capital (sentence-case) | `feat: add button` |
| 6 | `feat: add button.` | Subject ends with a period | `feat: add button` |
| 7 | `feat:` | Subject is empty | `feat: add button` |
| 8 | `: add button` | Type is empty | `feat: add button` |
| 9 | `feat(Auth): add login` | Scope must be lowercase | `feat(auth): add login` |
| 10 | `feat: add a really long subject line that keeps going past one hundred characters total here` | Header > 100 characters | shorten it |
| 11 | `WIP` | No type, not conventional | `chore: wip` (or use `--no-verify`) |
| 12 | `feat: add button` + body on the very next line (no blank line) | Missing blank line before body | add a blank line |

### pre-commit (lint-staged) failures

The commit also fails if a **staged** file has an ESLint **error** that
`eslint --fix` cannot auto-repair.

| Scenario | Result |
|----------|--------|
| Unused variable / import in a staged file | ❌ blocked (if rule = error) |
| Undefined variable used | ❌ blocked |
| Auto-fixable formatting issue | ✅ fixed automatically, commit proceeds |
| ESLint **warning** only | ✅ allowed, commit proceeds |
| Error in a file you did **not** stage | ✅ ignored (only staged files are checked) |

---

## ✅ Quick valid examples (copy these patterns)

```
feat: add csv export
fix: handle null user in profile page
docs: update setup instructions
style: run prettier on src
refactor: split utils into modules
perf: cache api responses
test: add tests for date helper
build: bump next.js to 15
ci: add lint step to pipeline
chore: clean up unused assets
revert: feat: add csv export
```

---

## Rules of thumb (never get blocked)

- Start with a valid **type** + `: ` → `feat: `, `fix: `, etc.
- **Lowercase** the first word of the subject.
- **No period** at the end of the subject.
- Keep the first line **under 100 characters**.
- Put a **blank line** before any body or footer text.

## Test a message before committing

```bash
echo "feat: add csv export" | npx commitlint
```

- No output / exit 0 → ✅ valid
- Lists problems → ❌ fix and retry

## Bypass the hooks (use sparingly)

```bash
git commit --no-verify -m "wip: skip hooks"
```
