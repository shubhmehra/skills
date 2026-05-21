# Craft — Post-Generation Checklist

Output this after every generation, pre-answered for the code just written.
Mark each item ✅ or ❌. Flag anything the dev must still handle manually.

---

## Architecture
- [ ] Code belongs to exactly one domain (`src/services/<domain>/`)
- [ ] Only `index.js` is the public interface — no internal files imported directly by other domains
- [ ] Any new cross-domain call is async (event-driven) or has a documented ADR

## Config
- [ ] Zero `process.env` outside `src/config/`
- [ ] No `static fromEnv()` methods
- [ ] All external dependencies (models, config) arrive via constructor

## Service Shape
- [ ] Service is a class with constructor-injected dependencies
- [ ] Every public method has a JSDoc `@param` and `@returns` annotation
- [ ] No `require()` or `import` calls inside methods

## Modern JS
- [ ] No `for`/`while` loops on collections — uses `map`/`filter`/`reduce`
- [ ] No `.then()` chains — uses `async/await`
- [ ] No `await` inside a loop — parallel ops use `Promise.all()`; sequential pagination uses a recursive async function
- [ ] No `/* eslint-disable no-await-in-loop */` — if you think you need it, the structure is wrong
- [ ] `?.` and `??` only on genuinely optional values — never on required business data
- [ ] `var` nowhere — `const` by default, `let` only when reassignment needed

## Express Handlers & Middleware
- [ ] No `return res.json(...)` mixed with no-return paths — uses if/else throughout
- [ ] No bare `return;` as a band-aid for `consistent-return` — if/else is the fix
- [ ] No `/* eslint-disable consistent-return */` — fix the structure instead
- [ ] No `eslint-disable` of any kind without explicit developer approval

## Coupling
- [ ] Changing this code breaks ≤ 3 files outside this domain folder
- [ ] No new `utils/update*.js` style cross-domain HTTP calls added without ADR

## Tests
- [ ] Test file exists at `src/services/<domain>/__test__/<ServiceName>.test.js`
- [ ] Tests assert what the system does — not how Mongoose/HTTP was called
- [ ] Each test covers one behavior with a plain-English description

## Size
- [ ] No file over 300 lines (500 for test files)
- [ ] No function over 50 lines
- [ ] No function with more than 4 parameters

---

## What the dev must still do manually

List here anything the skill could not generate or verify automatically:
- e.g. "Wire the new service into `context.js`"
- e.g. "Add the new connectionType constant to `src/constants/user.js`"
- e.g. "Write the ADR for the cross-domain call in `docs/adr/`"
- e.g. "Add the `index.js` export for this new service"
