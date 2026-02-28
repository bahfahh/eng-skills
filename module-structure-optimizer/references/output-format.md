# Output Format (Schema)

Produce a report that is directly actionable.

```yaml
summary:
  input_mode: whole-project | change-focused
  scope: "<paths/modules/keywords or null>"
  languages_detected: ["ts", "dotnet"]
  grounding_sources:
    - "requirements/spec"
    - "tests"
    - "usage/call-graph"
    - "diff (optional)"
  verdict: ok | optimize_recommended | optimize_required

module_candidates:
  - module: "<path or module name>"
    status: ready | not_ready
    confidence: high | medium | low
    grounding:
      - "<spec reference or test evidence or usage signal>"

    encapsulation_plan:
      facade_name: "<DomainFacadeName>"
      boundary:
        inputs: ["<domain inputs>"]
        outputs: ["<domain outputs>"]
      move_inside_facade:
        - "<what becomes internal>"
      keep_outside:
        - "<what stays external>"

    refactor_steps:
      - "<Step 1>"
      - "<Step 2>"

    tests_to_add:
      - type: integration | contract | e2e
        target: "<boundary under test>"
        cases:
          - "<Given/When/Then case>"

    simplifications:
      - location: "file:line"
        description: "<what to simplify/remove>"
        suggestion: "<how>"

    perf_arch_risks:
      - location: "file:line"
        pattern: "n_plus_1 | broad_read | missed_concurrency | repeated_computation | resource_risk | wrong_layer_io"
        impact: "<why it matters>"
        mitigation: "<what to change>"
        validate: "<tests/metrics/logs>"

open_questions:
  - "<question to ask when grounding is missing>"
```

Rules:
- If `status: not_ready`, `tests_to_add` MUST include the prerequisites to pin behavior.
- Avoid suggestions that require behavior changes unless the requirement/spec explicitly demands it.

