# Pattern Details

## Recommended Patterns (★★★★☆ ~ ★★★★★)

### Strategy

- **Scenario**: Dynamic algorithm/behavior switching — search strategies, prompt routing, optimizer selection
- **AI Advantage**: Perfect fit for RAG and Agent systems; encapsulating prompts as strategy objects is more testable than if-else chains
- **Guidance**: Define strategy interface, one implementation class per algorithm, context switches strategy at runtime

### Factory Method

- **Scenario**: Create different object types by condition — payment methods, LLM providers
- **AI Advantage**: Adding features only requires a new class registered in the factory, no core logic changes (Open-Closed Principle), reduces regression risk
- **Guidance**: Factory receives config parameter, returns interface instance; explicitly handle unknown types (throw exception, never fail silently)

### Adapter

- **Scenario**: Integrate different APIs or legacy systems — AWS S3 vs Azure Blob, TensorFlow vs PyTorch format conversion
- **AI Advantage**: AI excels at reading two different API docs and writing conversion layers; this repetitive code is where AI is most precise
- **Guidance**: Provide source and target interface definitions, Adapter implements target interface and wraps source object

### Builder

- **Scenario**: Construct complex objects with many parameters — HTTP request config, ML pipeline, prompt context
- **AI Advantage**: Fluent interface lets AI explicitly specify each property, avoiding parameter order mistakes
- **Guidance**: Use `setX()` chained methods instead of long constructor parameter lists, call `build()` to produce final object

### Facade

- **Scenario**: Provide simplified unified interface for complex subsystems
- **AI Advantage**: AI only needs to read the Facade interface definition to use the subsystem, saving context window and preventing internal API misuse
- **Guidance**: Each module exposes a Facade as its sole external entry point

### Command

- **Scenario**: Task queues, undo/redo, operation logging
- **AI Advantage**: Encapsulate prompts or agent actions as executable Command objects with parameters and results, forming complete audit trails
- **Guidance**: Each Command implements `execute()`, optionally implements `undo()`

### Chain of Responsibility

- **Scenario**: Requests passing through multiple processing steps — agent pipelines, middleware, validation chains
- **AI Advantage**: Each handler has single responsibility, AI can correctly implement individual nodes
- **Guidance**: Ensure chain has explicit termination condition to prevent infinite passing

## Caution Patterns (★★☆☆☆ ~ ★★★☆☆)

### Proxy

- **Scenario**: Adding caching, access control, rate limiting, or lazy initialization around an existing service — without modifying the service itself. Common in API gateway layers, repository caching wrappers.
- **Risk**: May introduce unnecessary latency; verify the indirection is justified before adding
- **Guidance**: Proxy implements the same interface as the real subject; keep it thin — one cross-cutting concern per Proxy

### State

- **Scenario**: An object's behavior changes significantly based on its current state, and those states have clear named transitions — e.g., order lifecycle (pending → confirmed → shipped → delivered), subscription status, workflow approvals.
- **Risk**: State count growth leads to state explosion; AI easily confuses state transition logic when states are implicit
- **Condition**: Persist state to database rather than in-memory objects; name states explicitly as enums or constants

### Decorator

- **Scenario**: Adding optional, composable behaviors to objects at runtime — e.g., adding logging, retry logic, or metrics to a service without modifying it. Useful when subclassing would produce a combinatorial explosion of classes.
- **Risk**: With deep nesting (e.g., `new Logging(new Caching(new Auth(service)))`), AI cannot reason about final behavior order
- **Alternative**: AOP frameworks (annotation-based) — AI understands annotations better than dynamic wrapping
- **Condition**: Limit to 2-3 decorator layers max, each with a single clear responsibility

### Observer

- **Scenario**: Decoupling event producers from consumers in genuinely event-driven architectures — UI frameworks, message queues, domain event systems where producers should not know about consumers.
- **Risk (AI-specific)**: Implicit control flow makes it hard for AI to trace event propagation paths during debugging or code generation. AI tends to miss subscriber registrations or duplicate event emissions.
- **Condition**: Only use when the event-driven decoupling is architecturally required; keep subscriber count manageable and document the event contract explicitly

### Template Method

- **Scenario**: Multiple classes share the same algorithm skeleton but differ in specific steps — e.g., report generators that all follow fetch → transform → render but with different implementations per step.
- **Risk**: Deep inheritance causes AI to override wrong methods or introduce unintended side effects in base class hooks
- **Condition**: Keep inheritance to 1-2 levels max; prefer Strategy over Template Method when the varying parts can be injected as objects
