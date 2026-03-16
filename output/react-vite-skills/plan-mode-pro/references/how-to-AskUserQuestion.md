# AskUserQuestion Usage Guide

## 核心原則
AI 要像技術顧問：整理好選項、解釋清楚利弊、讓使用者只需要做選擇。

---

## ✅ Appropriate Use Cases

### Architectural Decisions
- **User Authentication Implementation**
  - Requires decisions on: Session vs JWT, token storage location, middleware structure

- **Dark Mode Implementation**
  - Requires decisions on: theme system architecture, affects multiple components

### Multiple Approach Scenarios
- **Database Query Optimization**
  - Requires profiling first, multiple optimization methods available, significant impact

- **API Error Handling Updates**
  - Affects multiple files, requires user approval of approach

### Seemingly Simple but Actually Complex
- **Add Delete Button to User Profile**
  - Involves: button placement, confirmation dialog, API call, error handling, state updates

---

## ❌ Inappropriate Use Cases

### Straightforward Tasks
- Fix typos in README
- Add console.log for debugging
- Research questions like "What files handle routing?"

**Principle**: For simple tasks with obvious implementation and no planning needed, execute directly.

---

## 提問策略：根據難度調整

### 簡單問題（命名、格式、簡單選型）
- 一輪最多 **3 題**
- 背景一兩句帶過即可
- 例：選擇 HTTP status code 組合、命名慣例

### 複雜問題（跨 Context 架構、資料一致性、狀態機設計）
- 一輪最多 **1-2 題**
- 每題需要完整背景鋪墊（見下方範例）
- 例：Outbox pattern vs 直接發 Queue、冪等策略

### 分輪規則
- 問題總數不設上限，但必須分輪問，每輪問完等回覆再繼續
- 告知使用者還有幾個問題（例：「這輪 2 題，後面還有 2 題」）

---

## 📋 Question Format Guidelines

### 每個問題必須包含
1. **背景**：這是系統的哪個部分、為什麼要做這個決定、不處理的後果
2. **選項**：2-3 個方案，每個附 pros/cons
3. **推薦**：明確推薦一個，說明理由

### 背景深度根據難度調整

**簡單問題** — 背景簡短：
```
API 回傳格式需要統一，影響前端解析方式。

A（推薦）：統一用 { success, data, error }
  ✓ 簡單直覺，前端好處理
  ✗ 非業界標準

B：RFC 7807 Problem Details
  ✓ 業界標準
  ✗ POC 階段 over-engineering

推薦 A，POC 簡單優先。
```

**複雜問題** — 背景要完整鋪墊：
```
【跨 Context 訊息傳遞的可靠性】

背景：Document Processing 解析完 PDF 後，把結果（ParsedActivityData）
丟到 Azure Storage Queue，Data Management 監聽 Queue 來接收並寫入資料庫。

風險：如果解析結果已存進 DB（標記「解析完成」），但訊息還沒丟進 Queue
就當機，Data Management 永遠收不到 — 使用者會看到「PDF 解析成功但資料
沒進來」。

A（推薦）：Outbox Pattern
  做法：同一個 DB transaction 同時寫解析結果和待發送訊息，
  背景排程定期補發未發送的訊息。
  ✓ 不會出現「存了但沒發」，當機後自動補發
  ✗ 多一張 outbox 表和 Timer Trigger，有幾秒延遲

B：直接發 Queue + retry
  ✓ 架構簡單，訊息即時
  ✗ DB 成功但 Queue 失敗時資料會卡住，需額外對帳機制

推薦 A，這是資料流咽喉，訊息遺失影響大，Outbox 是業界標準做法。
```

### 關鍵規則：不要假設使用者腦中有 codebase

- 提到技術概念時，用白話解釋它在系統裡的角色
- 不要突然丟出 `QueryOrchestrator` 這種裸名詞，要帶上下文
- Pros/cons 用使用者能感受到的影響描述（等待時間、資料遺失、體驗差），不要只說抽象術語

### Template Structure
```
Which approach should we use for <decision>?

Option A (Recommended): <approach>
✓ Pros: <1–2 benefits>
✗ Cons: <1–2 drawbacks>
→ Best for: <when it fits>

Option B: <approach>
✓ Pros: <...>
✗ Cons: <...>
→ Best for: <...>

Recommendation: Option A because <one-sentence rationale>.
```

---

## ✅ Examples (Generic)

> Note: These are **generic examples** to illustrate the format. Always adapt options/tradeoffs to the **actual repo/system constraints**.

### Good example — multiple choice with recommendation (decision-complete)

Which approach should we use to handle API retries?

Option A (Recommended): Centralized retry policy (shared middleware / client wrapper)
✓ Pros: Consistent behavior, easier observability and tuning
✗ Cons: Slightly more upfront plumbing
→ Best for: Multiple callers or multiple endpoints

Option B: Per-call retries (add retry logic at each call site)
✓ Pros: Fast to ship for one call path
✗ Cons: Inconsistent behavior, harder to audit and maintain
→ Best for: One-off experiments or very isolated calls

Recommendation: Option A to keep behavior consistent and reduce long-term maintenance cost.

### Bad example — vague, no options, no tradeoffs

"How do you want to implement error handling?"

Why it's bad: Too open-ended; the user must supply all context; you can't produce a decision-complete plan from the answer.

---

## Noteit Web（選用）

- 若問題複雜、選項多，或容易因上下文不足而誤解，可使用 **Noteit Web** 建立「互動決策頁」輔助使用者選擇
  （選用，非強制）
- 建議標籤（tag）：`requirement_confirm`

### 呈現綱要（先判斷需求／問題型態）

- **架構／流程／資料流／模組互動／責任切分**
  - 視需要加入圖示化說明（例如 Mermaid）以輔助理解
  - 圖示後接續 **Decision Checklist**
- **設定／選型／政策或規則類（例如雲端設定）**
  - 以以下結構呈現：
    - Options
    - Recommended
    - Pros / Cons
    - 何時適用

### Goal

- 在 Noteit Web 的 HTML 中提供選擇題
  - 每題 2–3 個選項
  - 第一個選項為推薦
- 提供固定回覆格式，讓使用者可直接複製貼上答案
  - 例如：`1A, 2B, 3A`
