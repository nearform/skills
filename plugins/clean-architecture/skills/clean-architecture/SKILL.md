---
name: clean-architecture
description: "Enforces Clean Architecture layering for TypeScript factory-based codebases. Use when creating, refactoring, or reviewing handlers, services, domains, and repositories. Covers layer responsibilities, factory function rules, when to create vs inline logic, and file size heuristics for splitting. Trigger terms: service, domain, repository, handler, factory, clean architecture, layer separation, SRP, refactor, split file, too many lines."
metadata:
  tags: architecture, clean-architecture, typescript, solid, ddd, factory, service, domain, repository
---

## When to use

Use this skill when:
- Creating new handlers, services, domains, or repositories
- Refactoring code across architectural layers
- Reviewing whether logic lives in the right place
- Deciding whether to split a file or keep it together
- Unsure if something is orchestration (service) or business logic (domain)

## Layer responsibilities

Each layer has one job. Logic that crosses boundaries is a code smell.

### Handlers (event/route files)

Entry points. Wire input to services, return output. Minimal logic.

- Parse and validate input (via schema/middleware)
- Call one or a few services/domains
- Return/emit the result
- **Should NOT** contain loops with business decisions, multi-step orchestration, or direct repository calls beyond trivial reads

#### Handler shapes by runtime

The responsibilities above are universal, but the file layout depends on the runtime. The two most common shapes:

**Fastify (CQRS-style)** — the HTTP entry point and the command handler live in separate files:

```typescript
// thing.route.ts — HTTP entry point only
export default async function thingRoute(fastify: FastifyRouteInstance) {
  fastify.route({
    method: 'POST',
    url: '/v1/things',
    schema: { body: createThingRequestSchema, response: { 201: idSchema } },
    handler: async (req, res) => {
      const id = await fastify.commandBus.execute(createThingCommand(req.body));
      return res.status(201).send({ id });
    },
  });
}
```

```typescript
// create-thing.handler.ts — command handler factory, registered on the bus
export default function makeCreateThing({ thingRepository, thingDomain, commandBus, eventBus }: Dependencies) {
  return {
    async handler({ payload }: HandlerAction<typeof createThingCommand>) {
      const thing = thingDomain.create(payload);
      await thingRepository.insert(thing);
      eventBus.emit(createThingEvent(thing));
      return thing.id;
    },
    init() {
      commandBus.register(createThingCommand.type, this.handler);
    },
  };
}
```

The route is purely transport (HTTP → command). The command handler is bus-agnostic — the same handler can be triggered by a GraphQL resolver, a queue consumer, or an internal caller.

**Lambda (cradle-injected)** — the route file IS the handler; dependencies arrive through a cradle parameter:

```typescript
// create-thing.route.ts — schema + handler in one place
export const createThingRoute = createRoute({
  method: 'POST',
  path: '/v1/things',
  schema: { body: createThingRequestSchema, response: { 201: idSchema } },
  handler: async ({ thingRepository, thingDomain }, event, ctx) => {
    const thing = thingDomain.create(event.body);
    await thingRepository.insert(thing);
    return ctx.res.status(201).json({ id: thing.id });
  },
});
```

For event-driven Lambdas (SQS, EventBridge, scheduled), a sibling `createEvent` helper produces a handler with the same `(cradle, event)` shape — no HTTP envelope.

**Rules that apply to both shapes:**
- The handler stays thin — schema validation, one or a few service/domain calls, response shaping
- Business decisions belong in the domain; orchestration in the service
- The difference is *where the function lives* and *how dependencies are wired*, not *what the function is allowed to do*

### Services

Orchestrators. They coordinate repositories, domains, and adapters. They perform I/O and delegate decisions.

- Read from repositories
- Call domain functions for decisions
- Write to repositories
- Trigger events via orchestrators
- **Must NOT** contain business logic: no status checks, no threshold comparisons, no aggregations, no conditional evaluation based on domain rules

**When to create a service:**
1. A handler has too much orchestration logic (reading, deciding, writing, notifying)
2. The same orchestration is needed by multiple handlers

If a handler just calls one domain function and returns the result, a service adds no value — skip it.

### Domains

Business logic. Pure functions, computations, evaluations, aggregations. The brain of the system.

- Evaluate conditions (e.g., is this record stale? what action should we take?)
- Compute values (e.g., error rates, staleness thresholds, date ranges)
- Aggregate results (e.g., combine per-item statuses into batch-level summaries)
- Transform data (e.g., serialize params, classify entities)
- **Must NOT** call repositories, trigger events, or perform I/O

**The litmus test:** "Does this function make a decision or perform a computation?" If yes, it's a domain function. "Does it read/write data or trigger side effects?" If yes, it's a service function.

### Repositories

Data access. SQL queries, DynamoDB operations. Nothing else.

- CRUD operations
- Query builders
- Mapper delegation
- **Must NOT** contain business logic or orchestration

## Factory function rules

All services, domains, and repositories use the factory function pattern. No classes (except errors).

```typescript
// The factory takes Dependencies (the Cradle) and returns an object
export default function jobProcessingService({
  jobsRepository,
  jobsDomain,
  logger,
}: Dependencies) {

  // Private helpers as inner closures
  const formatKey = (prefix: string, id: string) => `${prefix}-${id}`;

  // Public methods on the return object
  async function processItem(item: { id: string }) {
    const job = await jobsRepository.findLatest(item.id);
    const action = jobsDomain.evaluate(job);
    // ... orchestration
    return action;
  }

  return { processItem };
}
```

**Rules:**
- All functions live INSIDE the factory — either on the return object (public) or as inner closures (private)
- Never export standalone functions alongside the factory
- Types and interfaces MAY be exported outside the factory
- No `create` prefix on factory names — the name is the thing it produces
- Parameter type is always `Dependencies` (global Cradle alias)
- Never annotate the return type — let TypeScript infer it

## When to split vs keep together

File size is a signal, not a rule. The goal is cohesion — functions that solve the same problem belong together, even in a larger file.

### Handlers: split at ~100 lines

If a handler file exceeds ~100 lines of actual logic (excluding imports, types, and schema definitions), the handler is doing too much. Extract the orchestration into a service, leaving the handler as a thin wrapper that calls the service.

This rule applies to both Fastify route/command-handler files and Lambda route/event files — the shape is different, but the budget is the same.

**Before** (handler doing orchestration — applies to either shape):
```typescript
// 150 lines — handler evaluates jobs, recovers stale ones, triggers retries
handler: async (/* deps */) => {
  const items = await itemsRepository.findAll();
  for (const item of items) {
    const job = await jobsRepository.findLatest(item.id);
    if (jobsDomain.isStale(job)) { /* ... */ }
    // ... 80 more lines of orchestration
  }
}
```

**After** (handler delegates to service):
```typescript
// 30 lines — handler calls service, collects metrics
handler: async (/* deps */) => {
  const items = await itemsRepository.findAll();
  for (const item of items) {
    const action = await jobProcessingService.processItem(item);
    metrics[action]++;
  }
}
```

### Domains and services: split at ~400 lines

If a domain or service exceeds ~400 lines, evaluate whether it has multiple distinct responsibilities. If it does, split into separate files — each with a focused scope.

**Split when:** the file has two or more groups of functions that serve different purposes and don't share private state. Example: a domain file that has both "job evaluation" functions and "date range generation" functions — these are separate concerns.

**Keep together when:** all functions collaborate to solve one problem, share private state (closures, maps, caches), or would create artificial coupling if separated. Example: an `externalApiProvider` that has request building, response parsing, and retry logic — these are tightly coupled steps of one operation.

**The smell test:** if you can describe what the file does in one sentence without using "and", it's probably cohesive enough to stay together. "Evaluates job status and decides processing actions" = one concern. "Manages proxy rotation and computes pricing recommendations" = two concerns.

## Quick reference

| Question | Answer |
|----------|--------|
| Does it check a status or compute a value? | Domain |
| Does it read from DB or call an API? | Service (or repository for raw DB) |
| Does it loop over items and call domain + repo? | Service |
| Is it a pure function with no side effects? | Domain |
| Does it trigger an event or send a notification? | Service |
| Is the handler >100 lines of logic? | Extract to a service |
| Is the domain/service >400 lines? | Consider splitting by concern |
| Do all functions share the same private state? | Keep together regardless of size |
