# Category 3: MCP Enhancement

> 為已有的 MCP server 提供工作流程指導與最佳實踐，讓 Claude 不只能「用工具」，還知道「怎麼用好」。

---

## 特徵

- 前提：使用者已有一個可用的 MCP server
- Skill 本身不提供新工具，而是**教 Claude 怎麼用好既有工具**
- 嵌入領域專業知識（domain expertise）
- 降低使用者的學習曲線

## 適用場景

- 使用者連了 MCP 但不知道下一步該做什麼
- 每次對話都要重新解釋工作流程
- 不同使用者用同一個 MCP 但結果不一致
- 需要在工具操作前加入領域規則（合規、稽核、治理）

## 官方範例

`sentry-code-review` skill（from Sentry）：
```yaml
description: Automatically analyzes and fixes detected bugs in GitHub Pull Requests using Sentry's error monitoring data via their MCP server.
```

---

## 推薦設計模式

### Pattern 1: Domain-Specific Intelligence

在工具操作前嵌入領域規則。

```markdown
## Before Processing (Compliance Check)
1. Fetch transaction details via MCP
2. Apply compliance rules:
   - Check sanctions lists
   - Verify jurisdiction allowances
   - Assess risk level
3. Document compliance decision

## Processing
IF compliance passed:
  - Call payment processing MCP tool
  - Apply appropriate fraud checks
  - Process transaction
ELSE:
  - Flag for review
  - Create compliance case

## Audit Trail
- Log all compliance checks
- Record processing decisions
- Generate audit report
```

**關鍵**：領域專業嵌入邏輯、行動前合規檢查、完整記錄、清楚治理規則。

### Pattern 2: Context-Aware Tool Selection

同一個 MCP 有多個工具，依情境選擇最適合的。

```markdown
## Decision Tree
1. Analyze user request
2. Determine best MCP tool:
   - Query data → use `search` tool (read-only, safe)
   - Create resource → use `create` tool (confirm with user first)
   - Bulk operation → use `batch` tool (validate input count)
3. Apply tool-specific best practices
4. Execute with appropriate parameters
```

### Pattern 3: Sequential Workflow（MCP 專屬流程）

把散落的 MCP 工具串成有意義的工作流程。

```markdown
## Workflow: Code Review with Sentry Data
### Step 1: Fetch Error Data
Call Sentry MCP: `get_issues` for target PR
Filter: only errors related to changed files

### Step 2: Analyze Impact
- Correlate errors with code changes
- Assess severity and frequency
- Identify root cause patterns

### Step 3: Generate Fix Suggestions
Based on error patterns:
- Suggest code fixes
- Reference Sentry documentation
- Prioritize by impact

### Step 4: Create Review Summary
- List all findings
- Attach Sentry issue links
- Provide actionable recommendations
```

---

## Description 撰寫重點

- 明確提到**搭配哪個 MCP server**
- 說明 skill 加了什麼價值（不只是「用 MCP」，而是「用好 MCP」）
- 包含使用者在該服務情境下會說的話

```yaml
# ✅ Good
description: Optimizes Notion workspace setup using Notion MCP. Applies team knowledge base best practices including page hierarchy, database relations, and template structure. Use when user says "set up Notion workspace", "organize my Notion", or "create Notion template".

# ❌ Bad
description: Uses Notion MCP to do things with Notion.
```

---

## 關鍵技巧

- 在 `references/` 放 MCP server 的 API patterns、rate limiting、error codes
- 明確寫出 MCP tool names（大小寫敏感）
- 加入 MCP 連線失敗的錯誤處理
- 把「使用者原本需要自己指定的 context」內建到 skill 中
- YAML frontmatter 的 `metadata.mcp-server` 欄位標明依賴的 MCP

```yaml
metadata:
  mcp-server: sentry
```

---

## 測試重點

| 維度 | 檢查項目 |
|------|----------|
| 觸發 | 在該 MCP 相關情境下觸發，不在無關情境觸發 |
| MCP 連線 | MCP 未連線時能正確提示，不會直接報錯 |
| 工具呼叫 | tool names 正確、參數正確、回傳值正確處理 |
| 領域規則 | 嵌入的領域知識被正確套用（合規檢查、最佳實踐） |
| 獨立測試 | 先不透過 skill，直接測 MCP 能否正常運作 |

---

## 常見問題

| 問題 | 解法 |
|------|------|
| MCP 呼叫失敗 | 先獨立測試：`"Use [Service] MCP to fetch my projects"`，如果這也失敗，問題在 MCP 不在 skill |
| Tool name 錯誤 | 確認 MCP server 文件中的 tool names，大小寫敏感 |
| API key 過期 | 在 Troubleshooting 段落加入認證檢查步驟 |
| Skill 載入但 MCP 沒連 | 在 skill 開頭加 `CRITICAL: Verify MCP server [name] is connected before proceeding` |
| 使用者不知道要先連 MCP | Description 中提到需要哪個 MCP：`Requires Sentry MCP server connected` |
