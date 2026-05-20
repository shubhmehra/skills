---
name: craft
description: Generates and refactors JavaScript code following modern ES6+ patterns, SOLID principles, modular architecture, and TDD-ready structure. Use when writing a new service, refactoring existing code, implementing a feature, or when the user says "craft this", "write this properly", "refactor this", "build this right", "follow coding standards", or asks to write JS that is clean, modular, maintainable, or production-ready. Also use when code has old for/while loops, scattered process.env, missing JSDoc, or violates single responsibility.
---

# Craft

Generate and refactor JavaScript code that is modular, maintainable, and follows the full rulebook: modern ES6+, SOLID, config discipline, coupling-free boundaries, and TDD-ready structure.

Works at any layer — new service, new feature, refactoring existing code, any domain.

## Workflow

### Phase 1 — AUDIT (silent, read before writing anything)

- [ ] Find the domain folder this belongs to (`src/services/<domain>/`)
- [ ] Check if a service for this domain already exists
- [ ] Check for `process.env` scattered outside `src/config/`
- [ ] Identify cross-domain imports or `update*.js` utils in the area
- [ ] Flag any file over 300 lines or function over 50 lines nearby
- [ ] Check if `__test__/` folder exists next to the target file

### Phase 2 — GATE (ask before generating — all 4 required)

1. **Domain** — which `src/services/<domain>/` does this belong to?
2. **Single behavior** — one sentence: what does this add or change?
3. **Not owned** — what does this module explicitly NOT do?
4. **Interface shape** — public method name, params, and return type?

### Phase 3 — GENERATE

Produce code following all rules in [REFERENCE.md](REFERENCE.md).

Every new service follows this shape:

```js
class ExampleService {
  constructor({ model, config }) {
    this.model = model;
    this.config = config;
  }

  /**
   * @param {string} userId
   * @returns {Promise<ExampleDoc|null>}
   */
  async getById(userId) {
    const doc = await this.model.findOne({ userId });

    if (!doc) throw new ResourceNotFoundException('Example not found');

    return doc;
  }
}
```

### Phase 4 — VALIDATE

Output the [CHECKLIST.md](CHECKLIST.md) pre-answered for everything just generated.
Flag anything the dev must still handle manually.

## See also

- [REFERENCE.md](REFERENCE.md) — full JS rules, SOLID, coupling, config, size limits
- [CHECKLIST.md](CHECKLIST.md) — post-generation validation checklist
