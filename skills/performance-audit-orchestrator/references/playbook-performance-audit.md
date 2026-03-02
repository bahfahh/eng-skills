# General Performance Audit Guide

Use this playbook when no specific aspect playbook applies, or as a starting checklist before narrowing focus. Produce YAML findings using the schema in `output-format.md`.

## Analysis areas

1. **Technology stack analysis**
   - Identify language, framework, runtime, and build tools
   - Check for existing performance monitoring (APM/RUM/logging/tracing)
   - Review build and deployment pipeline configuration

2. **Code performance**
   - Identify inefficient algorithms and O(n²) loops
   - Look for unnecessary blocking operations or redundant computations
   - Review memory allocation patterns and potential leaks

3. **Database performance**
   - Locate N+1 query patterns (ORM `include` chains, loop queries)
   - Check for missing indexes on frequently filtered/sorted columns
   - Review connection pool configuration and migration history

4. **Frontend performance (if applicable)**
   - Check bundle size and code-splitting configuration
   - Review image optimization and lazy loading
   - Identify re-render hotspots and missing memoization

5. **Network and API performance**
   - Review API call patterns (sequential vs parallel, waterfall chains)
   - Check payload sizes and compression settings
   - Identify missing or incorrect cache headers

6. **Asynchronous operations**
   - Detect sequential awaits that could run in parallel
   - Check for unhandled promise rejections or fire-and-forget patterns
   - Review background job and queue configurations

7. **Build pipeline**
   - Measure clean vs incremental build times
   - Check for missing cache configurations in CI
   - Identify large or duplicate dependencies in the build graph

8. **Observability gaps**
   - Note missing APM instrumentation, tracing, or RUM coverage
   - Identify key metrics that are not currently collected

For each finding, ground it with a file path and line number where possible. If a metric cannot be measured from static analysis, record it under `limits` and propose the minimal instrumentation step needed to unblock.
