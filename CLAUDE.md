# Skills 創建專案

使用者提出需求，由此專案產出對應的 skill。**目標：AI 依照本文件，生成出高品質、可直接使用的 skill。**

---

## 參考來源

- **官方完整指南**：[The Complete Guide to Building Skills for Claude](./The-Complete-Guide-to-Building-Skill/The-Complete-Guide-to-Building-Skill-for-Claude.extracted.txt)
- 結構規範：[SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- 官方最佳實踐：[agent-skills/best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [skills-best-practices](https://github.com/mgechev/skills-best-practices)

---

## 基本原則

### 理解需求再動手

不確定需求時，以選擇題詢問使用者，對齊用途與範圍後再開始建立。

### Skill 的分類維度

每個 Skill 同時具備兩個維度，各自獨立選擇：

**維度 1：實作形式**（指令怎麼寫）

| 形式 | 說明 | 何時用 |
|------|------|--------|
| 工具型 | 提供腳本或命令讓 Claude 執行 | 步驟確定、需要確定性輸出 |
| 指導型 | 提供流程或規則讓 Claude 遵循 | 步驟需判斷、或純知識型任務 |

**維度 2：應用類型**（Skill 解決什麼問題）

| 應用類型 | 說明 | 詳細參考 |
|----------|------|----------|
| 產出型 | 生成文件、設計、程式碼等輸出物 | [category-document-creation.md](./references/category-document-creation.md) |
| 流程自動化型 | 多步驟流程、orchestrator | [category-workflow-automation.md](./references/category-workflow-automation.md) |
| MCP 增強型 | 為既有 MCP 工具補充工作流程指導 | [category-mcp-enhancement.md](./references/category-mcp-enhancement.md) |

---

## Skill 目錄結構

```
skill-name/
├── SKILL.md              # 必要，主入口（500 行以內）
├── scripts/              # 可執行腳本
├── references/           # 參考文件，只從 SKILL.md 一層連結
└── assets/               # 模板、圖示等靜態資源
```

### 三層載入機制（Progressive Disclosure）

| 層級 | 內容 | 何時載入 |
|------|------|----------|
| 1 | `name` + `description`（~100 tokens） | 永遠預載 |
| 2 | `SKILL.md` 主體 | Skill 被觸發時 |
| 3 | 其他資源檔案 | 需要時才讀取 |

**核心原則：只在需要時才載入，避免佔用 context window。**

---

## YAML Frontmatter 規範

```yaml
---
name: skill-name          # 小寫字母、數字、連字號，最多 64 字元
description: ...          # 最多 1024 字元，不能含 XML 標籤
---
```

### name 命名規則

格式：`{category}-{skill-name}`

所有 skill 必須以分類前綴開頭，對應下表的類別：

| 類別前綴 | 說明 |
|---|---|
| `frontend-` | 前端開發（React、Next.js、UI/UX） |
| `architecture-` | 架構設計、模式選型 |
| `codereview-` | 代碼審查 |
| `review-gate-` | 審計、效能、安全、重構分析（不改程式碼） |
| `plan-mode-` | 規劃流程 |
| `db-` | Supabase 資料庫開發（migration、RLS、schema） |
| `backend-` | 後端開發（API、DB、auth） |
| `test-` | 測試策略、覆蓋率、E2E |

範例：`frontend-checklist`、`review-gate-security-audit`、`backend-api-design`

---

## Description 撰寫規則

- 使用**第三人稱**（不用「I」或「You」）
- 說明 **Skill 做什麼** + **何時觸發**
- 包含關鍵觸發詞，讓 Claude 能正確選到此 Skill
- 描述可以稍微「積極」，避免 Claude undertriggering

**範例**
```yaml
description: Analyzes module boundaries and suggests refactoring strategies. Use when reviewing code structure, untangling dependencies, or planning a modular refactor.
```

---

## 撰寫 Skill 內容的規則

### 指導型內容

1. **只寫 Claude 不知道的內容**，不要重述 AI 的既有知識。
2. **寫概念與方法，不鎖定特定技術**。若有技術限制，放入 references。
3. **描述做什麼，不解釋是什麼**（除非 Claude 真的不會知道）。
4. 使用**祈使句**（imperative form）直接下指令，不要用敘述文。

**錯誤示範**
> ASP.NET 是一個使用 DDD 架構的 Web 框架，首先你需要...

**正確示範**
> 使用 DDD 架構，依領域切分模組邊界。

### 自由度設定

根據任務的脆弱程度選擇指令的精確度：

| 情境 | 做法 |
|------|------|
| 多種方式都可行 | 高自由度：給方向，讓 Claude 判斷 |
| 有偏好但可彈性 | 中自由度：給模板，允許調整 |
| 步驟不能出錯 | 低自由度：給精確指令，不允許變動 |

### 工具型腳本

- 將脆弱、重複的操作寫成腳本（Python/Bash/Node），讓 Claude 執行而非生成
- 腳本要有清楚的錯誤訊息，讓 Claude 能自我修正
- 明確說明是「執行」還是「參考閱讀」：
  - 執行：`Run scripts/analyze.py to extract fields`
  - 閱讀：`See scripts/analyze.py for the extraction algorithm`

---

## 結構規範

- `SKILL.md` 主體保持 500 行以內
- 細節內容拆到獨立檔案，從 `SKILL.md` **一層**連結過去（不要多層巢狀）
- 參考檔案超過 100 行時，加上目錄（Table of Contents）
- 路徑統一使用正斜線 `/`（不用 `\`）

---

## 建立流程

1. **捕捉意圖**：確認 Skill 的用途、觸發時機、期望輸出
2. **判斷應用類型**，讀取對應參考文件再繼續：
   - 產出型 → [category-document-creation.md](./references/category-document-creation.md)
   - 流程自動化型 → [category-workflow-automation.md](./references/category-workflow-automation.md)
   - MCP 增強型 → [category-mcp-enhancement.md](./references/category-mcp-enhancement.md)
3. **釐清細節**：詢問邊界案例、格式需求、相依套件
4. **撰寫 SKILL.md**：寫好 frontmatter 與指令內容
5. **建立測試案例**：在 `evals/evals.json` 寫 2-3 個真實測試情境（含 1 個不該觸發的負例）
6. **內容品質迭代**：用 subagent 跑測試，比對輸出品質，根據結果改 SKILL.md 指令內容（見下方）
7. **Description 觸發優化**：內容確定後，再優化 description 的觸發準確度（見下方）

---

## 測試與迭代方法

### 內容品質迭代（先做）

測試 skill 執行後的輸出品質是否達標。

對每個 eval 案例，啟動兩個 subagent 並行：
- **with_skill**：給 subagent SKILL.md 路徑 + eval prompt，讓它依照 skill 執行
- **without_skill**（baseline）：同樣的 prompt，不給 skill，讓 Claude 自己做

比對輸出差異，判斷 skill 有沒有帶來明顯提升。根據問題改 SKILL.md 的指令內容，重複直到滿意。

> **注意**：改的是 SKILL.md 的指令邏輯，不是 description。

### Description 觸發優化（後做）

內容確定後，測試 description 有沒有讓 Claude 在對的時機觸發 skill。

準備 10–20 個 query，分兩類：
- `should_trigger: true`：使用者確實需要這個 skill 的情境
- `should_trigger: false`：近似但 Claude 自己就能處理、不需要 skill 的情境

用 subagent 模擬：給它 skill 的 name + description，問它「面對這個 query，你會不會用這個 skill？」收集結果，針對失敗的 query 調整 description 措辭。

**常見問題與修正方向：**

| 症狀 | 原因 | 修正 |
|------|------|------|
| should_trigger=true 沒觸發 | description 太模糊或被動 | 補入使用者真實語句、具體觸發情境 |
| should_trigger=false 誤觸發 | description 範圍太廣 | 加 `Do NOT use for...` 明確排除情境 |
| 複雜情境觸發、簡單情境也觸發 | 缺少複雜度門檻描述 | 加入「多模組/多面向」作為必要條件 |

---

## All-in-one Skills（Orchestrator）

當使用者要求製作「all in one skills」時，遵守以下概念與約束：

<all in one skills concept>
## all in one skills
- All in one skills：對外看起來是「一個完整 skill 就能把任務做完、交付結果」，但內部其實是編排器（orchestrator）——會自己拆解任務、安排步驟、整合輸出；使用者不需要知道背後有沒有多個 agent。
- 智能分配 agent 數量：根據以下 Decision Tree 決定配置，最後一定要合併成單一結論 / 單一交付物（不是丟一堆分散回覆）。

### Agent 數量 Decision Tree

在 SKILL.md 裡寫出明確的判斷邏輯：

```
1. 任務面向數與複雜度？
   - 單一面向、範圍明確       → 0 subagent，orchestrator 自己執行
   - 中等複雜、需專門知識     → 1 個專家 subagent
   - 多面向、可並行、高風險   → 2–3 個專家 subagent 並行

2. 各面向工作量是否值得獨立 agent？
   - 輕量可快速完成           → 自己做
   - 需深度分析或大量工具呼叫 → 獨立 subagent

3. 無論幾個 agent，最後必須合併成單一交付物
```
</all in one skills concept>

## 已完成的 Skills

### `frontend-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `frontend-checklist` | 前端開發品質清單（兩種模式）：(1) coding/plan 模式—開發前讀取補充 todolist；(2) code review 模式—依清單審查完成的前端程式碼。涵蓋互動狀態、write path、auth/session、響應式設計等 AI 常遺漏的 UX 問題 |
| `next-checklist` | Next.js 開發品質清單（三種模式）：(1) coding/plan 模式；(2) code review 模式；(3) upgrade 模式（delegate 給 `next-official-upgrade`）。涵蓋 App Router、auth/session 安全、資料獲取策略、bundle 優化、Next.js 15 async API 變化 |
| `next-api-checklist` | Next.js Route Handler（API）開發清單（兩種模式）：(1) coding/plan 模式—建立 API route 前的完整 6 階段清單；(2) code review 模式—審查 route handler 安全性與結構。涵蓋 auth 驗證順序、service layer 分離、統一 response envelope、錯誤處理、multi-tenant 隔離、React Query 整合、驗收測試 |

### `architecture-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `architecture-design-patterns` | 設計模式選型指南，依情境推薦 AI 友善的模式，避免過度工程化 |
| `architecture-ai-native-design` | 語言無關的 AI-native 架構設計原則，協助規劃模組邊界、分層責任、穩定介面與測試策略 |

### `codereview-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `codereview-unified-pro` | PR/diff 審查協調器，依變更內容自動選擇審查深度，平行執行正確性、安全性、架構、效率等多維度審查 |

### `review-gate-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `review-gate-security-audit` | All-in-one 全 repo 安全審核編排器：智能啟動 0–3 子 agent（secrets / supply-chain / appsec），最後合併成單一安全報告（不改程式碼） |
| `review-gate-deep-module-refactor-advisor` | 審計成熟 codebase 的淺層模組與高耦合問題，產出 facade 封裝建議報告（不改程式碼） |
| `review-gate-module-structure-optimizer` | 功能完成後的後續優化：找封裝候選、移除冗餘抽象、偵測架構/效能風險（N+1、循序 await 等），輸出重構計畫報告 |
| `review-gate-performance-audit` | All-in-one 全 repo 效能審計編排器：智能啟動 0–4 子 agent（frontend perf / performance engineer / load testing / test automation），最後合併成單一效能報告（不改程式碼） |

### `plan-mode-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `plan-mode-pro` | 結構化規劃流程，產出 requirements.md、plan.md、todolist.md、acceptance.md、tests.md |

### `db-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `db-supabase-checklist` | Supabase DB 開發清單（兩種模式）：(1) coding/plan 模式—DB 變更前補充 todolist；(2) code review 模式—審查 migration、RLS policy、資料完整性。涵蓋 migration 命名規範、RLS policy 設計、多租戶隔離、idempotency、soft delete、TypeScript 型別同步 |

### `backend-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `backend-dotnet-checklist` | ASP.NET Core / C# 開發清單（兩種模式）：(1) coding/plan 模式—開發前補 todolist，優先檢查 8 條 Core Guardrails（DI scope、async/await、EF Core NoTracking、IOptions 等）；(2) code review 模式—依清單審查 .NET 程式碼。內建 C# 標準、EF Core patterns、並發模式三份 references，自給自足不依賴外部 skills |

### `bugfix-` 類
| Skill 名稱 | 用途 |
|-----------|------|
| `bugfix-systematic` | 複雜 bug 系統化調查與修復工作流：適用於已開 ticket、跨服務、難重現、或原因不明的 bug（regression、data corruption、race condition、security vuln、整合性失敗）。三階段流程（bugfix.md → design.md → tasks.md），以 C(X) 正式定義 bug 條件，搭配 property-based exploration test 驗證存在性與修復有效性 |

---

## 規劃中的 Skills

### `backend-` 類（待建立）
| Skill 名稱 | 用途 |
|-----------|------|
| `backend-api-design` | REST/GraphQL API 設計審查：endpoint 命名、錯誤處理、版本策略、冪等性 |
| `backend-database-design` | DB schema 審查：正規化、索引策略、migration 安全性、N+1 偵測 |
| `backend-auth-patterns` | 後端 auth/authz 模式：JWT、session、RBAC/ABAC、token refresh 安全 |

### `test-` 類（待建立）
| Skill 名稱 | 用途 |
|-----------|------|
| `test-strategy` | 測試策略規劃：測試金字塔、邊界選擇、mock vs real 決策樹 |
| `test-coverage-audit` | 覆蓋率分析：找出未測試的 critical path，輸出補測建議報告 |
| `test-e2e-orchestrator` | E2E 測試編排：場景規劃、flaky test 偵測、CI 整合建議 |

## 實踐注意事項


### 跨 Skill 依賴的寫法

不要寫死路徑（如 `.kiro/skills/react-best-practices/SKILL.md`），改寫 skill 名稱：

```markdown
Load the `react-best-practices` skill for detailed performance rules.
```
Claude 看到 skill 名稱會自動在已安裝的 skills 中查找，路徑寫死會在其他專案失效。

---

# 常見錯誤寫法

- **寫知識點**：AI 已知的概念（BDD、TDD、Design Pattern、HTTP 狀態碼…）不需要解釋。只寫 AI 不知道的：專案特有流程、獨特約束、領域專屬規則。
- **寫範例**：示範對話、範例程式碼、填好的模板——AI 看完就忘，不如直接給結構讓 AI 自己填。例外：範例是傳遞「規格要求」而非「教學」時可以保留。
- **重複說同一件事**：同一概念在多個 section 反覆出現（如 Success Criteria 的重要性），只說一次即可。
- **分步驟寫 AI 執行指令**：「步驟 1、步驟 2、步驟 3」這種格式是在教 AI 做牠已經會做的事。改成要求（constraint）或結構（structure）即可。
- **過度解釋為什麼**：「之所以這樣做，是因為……」略去，直接給指令。
