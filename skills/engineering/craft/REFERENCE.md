# Craft — Full Rulebook

## Modern JS Rules

### Always apply

```js
// async/await — never .then() chains or callbacks
const result = await service.getById(userId);

// map/filter/reduce — never for/while on collections
const active = connections.filter(c => c.status === 'connected');
const tokens = connections.map(c => c.accessToken);

// Destructuring for params and return values
const { userId, connectionType, accessToken } = data;

// Promise.all() for parallel async — never await inside a loop
const [meta, instagram] = await Promise.all([
  service.getByType(userId, 'META_ADS'),
  service.getByType(userId, 'INSTAGRAM'),
]);

// Named exports — never default exports for services/utilities
export { PlatformConnectionService };

// const always, let only when reassignment needed, var never
const doc = await model.findOne({ userId });
```

### Apply with caution — `?.` and `??`

Use ONLY for genuinely optional display values or config defaults.
NEVER on values whose absence is a business logic error — those must throw explicitly.

```js
// WRONG — silently propagates undefined into an API call
const token = connection?.accessToken;
await metaApi.fetch(token); // fails 3 hops later with a cryptic error

// CORRECT — fails loudly at the source
if (!connection) throw new ResourceNotFoundException('Connection not found');
const { accessToken } = connection;
await metaApi.fetch(accessToken);

// OK — genuinely optional config default
const timeout = config.requestTimeout ?? 5000;

// OK — optional display value in a template
const label = user.displayName ?? user.email;
```

The rule: if the missing value means something went wrong, throw. If the missing value means "not set yet", use `??`.

---

## Service Shape — SOLID Applied

Every service is a pure class with constructor-injected dependencies:

```js
class PlatformConnectionService {
  constructor({ platformConnectionModel }) {
    this.model = platformConnectionModel;
  }

  /**
   * @param {string} userId
   * @param {'META_ADS'|'INSTAGRAM'|'SHOPIFY'} connectionType
   * @returns {Promise<PlatformConnectionDoc|null>}
   */
  async getByUserAndType(userId, connectionType) {
    return this.model.findOne({ userId, connectionType });
  }
}
```

**S — Single Responsibility**
One service owns one domain concern.
`PlatformConnectionService` manages connection state. It does not send notifications, it does not call external APIs. If you find yourself adding an unrelated method, that method belongs in a different service.

**O — Open/Closed**
Add behavior by adding methods or a new service.
Never add a `type === 'X'` conditional inside an existing method to handle a new case.

**L — Liskov**
If a service implements a contract (e.g., all OAuth services expose `initiateOAuth` and `handleCallback`), every implementation must honour that contract fully — no method that silently does nothing or throws "not implemented".

**I — Interface Segregation**
Keep public interfaces narrow. A service that exposes 15 methods is doing too much. If a caller only ever needs 2 of those methods, the service should be split.

**D — Dependency Inversion**
Inject everything. Never `require()` a model or client inside a method. Never read `process.env` in a service. Config arrives via constructor. Models arrive via constructor.

---

## Config Discipline

```
src/config/
  app.js        ← PORT, NODE_ENV, feature flags
  db.js         ← MONGODB_URI
  aws.js        ← S3, Secrets Manager vars
  auth.js       ← JWT_SECRET, OTP config
  platforms.js  ← META_APP_ID, SHOPIFY_KEY, INSTAGRAM_CLIENT_ID
  internal.js   ← INTERNAL_API_SECRET
  context.js    ← composition root — instantiates everything once
```

- `process.env` appears in zero files outside `src/config/`
- `context.js` is the only file that does `new SomeService(config)`
- No `static fromEnv()` methods — ever

---

## Module Boundary Rules

```
src/services/
  platform/
    index.js                        ← ONLY file the outside world imports
    PlatformConnectionService.js    ← internal, never imported directly
    InstagramOAuthService.js        ← internal
    __test__/
```

- Import `services/platform` — never `services/platform/PlatformConnectionService`
- ESLint `no-restricted-imports` enforces this at commit time
- Cross-domain side effects are async events by default
- Synchronous cross-domain HTTP calls require a documented ADR

---

## Coupling Rules

> "The coupling is the variable, not the architecture style." — The Microservices Scam

Before adding any cross-domain call, ask: can this be an event instead?

**Coupling audit question for every PR:**
If I change this, how many files outside this domain folder break?
More than 3 = the boundary is wrong.

**AI amplifier warning (from the transcript):**
AI-generated code in a tightly coupled codebase creates bugs faster than it creates features.
The only context where AI adds real value is a modular, decoupled system where a change can be isolated and verified quickly. This skill exists to keep that true.

---

## Size Limits

| Rule | Limit | Signal when violated |
|---|---|---|
| File length | 300 lines | File owns too many concerns — split the domain |
| Function length | 50 lines | Function does too many things — extract a named helper |
| Parameters | 4 max | Use an options object: `({ userId, connectionType, accessToken })` |
| Nesting depth | 4 max | Early returns flatten nesting — fail fast at the top |
| Cyclomatic complexity | 10 max | More than 10 branches = untestable by definition |

---

## Test Shape

Tests live in `__test__/` next to the source. They test behavior, not implementation.

```js
// WRONG — tests HOW Mongoose was called (breaks on refactor)
const [, update] = stub.firstCall.args;
expect(update.$set.status).to.equal('connected');

// CORRECT — tests WHAT the system does (survives refactor)
await service.upsert({ userId, connectionType: 'META_ADS', accessToken: 'EAA...' });
const result = await service.getByUserAndType(userId, 'META_ADS');
expect(result.status).to.equal('connected');
```

One test per behavior. Write the test before the implementation (red → green → refactor).
If your test breaks when you rename an internal method but behavior didn't change, the test is wrong.

---

## Monolith vs Microservice

The code rules above are identical in both contexts.
The only thing that changes at a service boundary:

| Context | Calling another domain |
|---|---|
| Modular monolith | `import { platformService } from 'services/platform'` |
| Microservice | `new UserInfoManagerClient().getConnection(userId, type)` |

The internal shape of `PlatformConnectionService` is the same either way.
Where it runs is a deployment detail — not a code detail.
