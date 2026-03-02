# Routing (智能派工規則)

Use these rules to decide 0–3 sub-agents. Always produce a single merged report.

## 1) Detect aspects (面向)

Detect aspects from repo signals (prefer scope-limited search):

### `secrets`
Signals (any):
- `.env*`, `.npmrc`, `.pypirc`, `*.pem`, `id_rsa`, `credentials*`
- text patterns: `BEGIN PRIVATE KEY`, `AKIA`, `AIza`, `ghp_`, `github_pat_`, `xoxb-`, `xoxp-`, `sk-`, `-----BEGIN`
- CI usage: `secrets.` in workflows, writing secrets to logs, uploading env files as artifacts

### `supplychain_cicd`
Signals (any):
- `package.json`, `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`
- `requirements.txt`, `pyproject.toml`, `poetry.lock`, `Pipfile.lock`
- `Cargo.toml`, `Cargo.lock`, `pom.xml`, `build.gradle*`, `.csproj`
- `.github/workflows/*.yml`, `Dockerfile`, `docker-compose*.yml`, `*.tf`, `helm/`, `k8s/`

### `appsec`
Signals (any):
- API routes/controllers: `routes/`, `controllers/`, `handlers/`, `api/`
- auth/middleware: `auth`, `login`, `session`, `jwt`, `cookie`, `middleware`
- permissions/tenant: `rbac`, `role`, `permission`, `tenant`, `org`, `rls`, `policy`
- DB access: `query`, `repository`, `orm`, `SELECT`, `INSERT`, `sql`, `prisma`, `sequelize`, `typeorm`

### `ai_llm` (conditional, handled inside AppSec agent)
Signals (any):
- `openai`, `anthropic`, `langchain`, `llamaindex`, `vector`, `embedding`, `retrieval`, `tool`, `function_call`

## 2) Score risk + workload

### `risk_level`
- `high` if any:
  - secrets aspect detected with plausible real secret
  - auth/authorization/tenant isolation (RLS/ACL) present
  - CI/CD can deploy or has write tokens (`id-token: write`, `contents: write`, `pull_request_target`, custom PAT usage)
  - handles payment/admin/PII keywords
- `medium` if appsec or supplychain_cicd present but without high-risk signals
- `low` if none of the above and repo is mostly docs/static content

### `workload`
- `high` if: multi-language repo OR many manifests/workflows/IaC + app code
- `medium` if: single stack but has both app + CI + deps
- `low` if: very small repo (few files, no CI, no deps)

## 3) Decide agent count (0–3)

Let:
- `surface_count` = number of detected aspects among {secrets, supplychain_cicd, appsec} (ignore ai_llm here)

Decision:
- `0 agents` if `surface_count <= 1` AND `risk_level = low` AND `workload = low`
- `1 agent` if `surface_count = 1` AND (`risk_level != high`) AND `workload != high`
- `2 agents` if `surface_count = 2` OR `risk_level = high`
- `3 agents` if `surface_count >= 3` OR (`risk_level = high` AND `workload = high`)

Cap at 3 agents.

## 4) Choose which agents

- If `secrets` detected: include **Secrets Agent**
- If `supplychain_cicd` detected: include **SupplyChain Agent**
- If `appsec` detected: include **AppSec Agent**

If you need to drop one due to the 3-agent cap (rare): keep Secrets first, then SupplyChain, then AppSec; explicitly list what coverage is missing in `limits`.

## 5) Hot files selection (給 agent 的最小上下文)

For each aspect, provide only the top hot files (5–20 paths):
- `secrets`: `.env*`, config files, workflow files, any rg hits
- `supplychain_cicd`: manifests + lockfiles + workflows + Docker/IaC
- `appsec`: auth/middleware + routes/controllers + DB access layer + security headers config
- `ai_llm`: only if detected; include the minimal integration files

