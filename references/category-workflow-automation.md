# Category 2: Workflow Automation

> 多步驟流程自動化，需要一致的方法論、驗證關卡、可協調多個工具或 MCP servers。

---

## 特徵

- 有明確的步驟順序與依賴關係
- 需要驗證關卡（validation gates）：每步完成才進下一步
- 可能協調多個 MCP servers 或工具
- 包含錯誤處理與回滾機制
- 常有迭代改進迴圈

## 適用場景

- 專案建立與初始化流程
- Sprint 規劃與任務分配
- CI/CD pipeline 協調
- 多服務間的資料同步
- All-in-one orchestrator（編排器）

## 官方範例

`skill-creator` skill：
```yaml
description: Interactive guide for creating new skills. Walks the user through use case definition, frontmatter generation, instruction writing, and validation.
```

---

## 推薦設計模式

### Pattern 1: Sequential Workflow Orchestration

最基本的流程自動化——按順序執行，每步驗證。

```markdown
## Workflow: Onboard New Customer

### Step 1: Create Account
Call MCP tool: `create_customer`
Parameters: name, email, company

VALIDATE: customer_id returned successfully

### Step 2: Setup Payment
Call MCP tool: `setup_payment_method`
Wait for: payment method verification

VALIDATE: payment method status = "verified"

### Step 3: Create Subscription
Call MCP tool: `create_subscription`
Parameters: plan_id, customer_id (from Step 1)

### Step 4: Send Welcome Email
Call MCP tool: `send_email`
Template: welcome_email_template

### Rollback
If any step fails:
1. Log which step failed and why
2. Reverse completed steps in reverse order
3. Report failure with actionable next steps
```

**關鍵**：明確步驟順序、依賴關係、每階段驗證、失敗回滾。

### Pattern 2: Multi-MCP Coordination

工作流程跨越多個服務時，用 phase 分段。

```markdown
## Phase 1: Design Export (Figma MCP)
1. Export design assets from Figma
2. Generate design specifications
3. Create asset manifest

GATE: All assets exported, manifest complete

## Phase 2: Asset Storage (Drive MCP)
1. Create project folder in Drive
2. Upload all assets
3. Generate shareable links

GATE: All links generated

## Phase 3: Task Creation (Linear MCP)
1. Create development tasks
2. Attach asset links to tasks
3. Assign to engineering team

## Phase 4: Notification (Slack MCP)
1. Post handoff summary to #engineering
2. Include asset links and task references
```

**關鍵**：清楚的階段分離、MCP 間資料傳遞、進入下階段前驗證、集中錯誤處理。

### Pattern 3: Iterative Refinement（用於流程中的品質迴圈）

```markdown
### Quality Loop
1. Execute workflow
2. Run `scripts/check_output.py`
3. If issues found:
   - Address each issue
   - Re-execute affected steps
   - Re-validate
4. Repeat until quality threshold met or max 3 iterations
```

---

## All-in-one / Orchestrator 模式

當 skill 需要智能分配子任務時：

```markdown
## Task Analysis
1. Assess complexity, risk, scope
2. Decide execution strategy:
   - Simple → handle directly (0 sub-agents)
   - Medium → 1 specialist agent
   - Complex → multiple parallel agents

## Execution
- Dispatch to chosen strategy
- Each agent produces partial result

## Merge
CRITICAL: Always merge into single deliverable.
Never return scattered partial results.
```

---

## Description 撰寫重點

- 列出**流程涵蓋的步驟**（讓使用者知道 skill 做多少事）
- 包含流程動詞：`automate`, `orchestrate`, `coordinate`, `set up`, `initialize`
- 提到涉及的服務或工具

```yaml
# ✅ Good
description: Automates sprint planning workflow including backlog analysis, task prioritization, capacity check, and task creation in Linear. Use when user says "plan sprint", "create sprint tasks", or "organize backlog".

# ❌ Bad
description: Helps with project workflows.
```

---

## 關鍵技巧

- 每個步驟之間加驗證關卡，不要盲目往下走
- 失敗時提供明確的回滾指令或錯誤訊息
- 跨 MCP 時，用變數明確傳遞上一步的輸出（如 `customer_id from Step 1`）
- 關鍵操作寫成腳本確保確定性
- Orchestrator 模式最後一定要合併成單一交付物

---

## 測試重點

| 維度 | 檢查項目 |
|------|----------|
| 觸發 | 「幫我規劃 sprint」「set up a new project」能觸發 |
| 順序 | 步驟按正確順序執行，依賴關係正確 |
| 驗證 | 每步驗證關卡有效，不會跳過 |
| 錯誤 | 中間步驟失敗時能正確處理（回滾或報錯） |
| 整合 | 多 MCP 間資料正確傳遞 |

---

## 常見問題

| 問題 | 解法 |
|------|------|
| 步驟被跳過 | 用 `CRITICAL:` 標記必要步驟，加 `VALIDATE:` 關卡 |
| MCP 呼叫失敗 | 加錯誤處理段落，先獨立測試 MCP：`"Use [Service] MCP to fetch my projects"` |
| 步驟間資料遺失 | 明確寫出「使用 Step N 的 XXX」，不要假設 Claude 會記住 |
| Orchestrator 結果分散 | 在最後加 `CRITICAL: Merge all results into single deliverable` |
