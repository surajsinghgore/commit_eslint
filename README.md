# Git Commit Hooks Setup (Husky + lint-staged + commitlint)

A reusable guide to enforce **clean commit messages** and **auto-lint staged files**
before every commit. Drop this into any Node.js / TypeScript project.

## What you get

| Hook | Tool | What it does |
|------|------|--------------|
| `pre-commit` | **lint-staged** | Runs `eslint --fix` on staged files; blocks the commit on any remaining error |
| `commit-msg` | **commitlint** | Validates the commit message against Conventional Commits rules |
| (optional) `npm run commit` | **commitizen** | Interactive prompt that builds a valid message for you |

---

## Setup

### 1. Install dev dependencies

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

### 2. Add the `prepare` script and init Husky

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

### 3. Create the hook files

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

### 4. Add the config files (project root)

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

### 5. (Optional) commitizen prompt

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

---

## Commit message format

```
type(scope?): subject

body (optional)

footer (optional)
```

**Example:** `feat(auth): add google login`

---

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

### Allowed `type` values

| Type | Use for |
|------|---------|
| `feat` | a new feature |
| `fix` | a bug fix |
| `docs` | documentation only |
| `style` | formatting; no code-logic change |
| `refactor` | code change that is neither a fix nor a feature |
| `perf` | performance improvement |
| `test` | adding or fixing tests |
| `build` | build system or dependencies |
| `ci` | CI configuration |
| `chore` | maintenance; no src/test change |
| `revert` | reverts a previous commit |

---

## Rules of thumb (never get blocked)

- Start with a valid **type** + `: ` → `feat: `, `fix: `, etc.
- **Lowercase** the first word of the subject.
- **No period** at the end of the subject.
- Keep the first line **under 100 characters**.
- Put a **blank line** before any body or footer text.

### ❌ Rejected

```
feat: Remove document count from themes
      ↑ capital "R" → sentence-case not allowed
```

### ✅ Valid

```
feat: remove document count from themes, sub-themes and questions
```

---

## Test a message without committing

```bash
echo "feat: remove document count" | npx commitlint
```

## Bypass the hooks (use sparingly)

```bash
git commit --no-verify -m "wip: temporary commit"
```
