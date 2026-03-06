# Output Format (單一交付物)

All sub-agents output YAML using this schema. The orchestrator merges to one report.

## Sub-agent YAML schema

```yaml
findings:
  - severity: critical | high | medium | low | info
    category: secrets | supply_chain | cicd | auth | authorization | injection | data_protection | logging | ai_security | infra
    location: "path/to/file:line" # if no line, use "path/to/file:(section description)"
    title: "short risk statement"
    evidence: "why this is true (repo evidence); never include secret values"
    impact: "what can happen"
    recommendation: "specific fix steps"
    confidence: high | medium | low
notes:
  limits:
    - "what was not checked / missing tools / missing context"
```

## Orchestrator merge rules

- Dedupe: same `location + title` => keep higher `severity`, merge best `recommendation`.
- Conflicts: keep both recommendations and mark `needs human judgment`.
- Never output separate agent transcripts; merge findings only.

## Unified report template

```md
# Security Audit Report

## Summary
- scope:
- activated_agents: [none | secrets | supplychain | appsec] (with routing reasons)
- overall_risk: low | medium | high
- top_risks:
  - ...

## Immediate Actions (if any critical)
- ...

## Findings
### Critical
### High
### Medium
### Low / Info

## Remediation Plan (ordered)
1. ...

## Limits & Confidence
- ...
```

