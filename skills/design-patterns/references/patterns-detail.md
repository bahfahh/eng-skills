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

- **Use When**: Caching, access control, lazy loading
- **Note**: May introduce unnecessary latency; verify the indirection is justified

### State

- **Risk**: State count growth leads to state explosion; AI easily confuses state transition logic
- **Condition**: Persist state to database rather than in-memory objects only

### Decorator

- **Risk**: With deep nesting (e.g., `new Logging(new Caching(new Auth(service)))`), AI cannot reason about final behavior order
- **Alternative**: AOP frameworks (annotation-based) — AI understands annotations better than dynamic wrapping
- **Condition**: Limit to 2-3 decorator layers max, each with clear responsibility

### Observer

- **Risk**: Implicit control flow makes it hard for AI to trace event propagation paths and debug
- **Condition**: Only use in event-driven architectures, keep subscriber count manageable

### Template Method

- **Risk**: Deep inheritance causes AI to override wrong methods
- **Condition**: Keep inheritance to 1-2 levels max
