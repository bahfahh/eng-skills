# Agent Skills 進階開發完全指南 (Advanced Notes)

> 本文件彙整了來自 Claude 官方與實戰經驗的進階設計模式、Troubleshooting 指南以及大型專案優化思路。

---

## 1. 設計思路：選擇你的切入點

在動手寫 Skill 前，必須先釐清 Skill 的定位，這會直接影響到後續採用的 Workflow Pattern。

### 🧩 Problem-first (問題導向)
*   **定義**：用戶描述的是「最終結果」，Skill 負責協調工具、決定順序。
*   **類喻**：去特力屋找店員說「我的櫥櫃壞了」，店員幫你配好螺絲起子、膠水並教你修理順序。
*   **場景**：「幫我設置一個新專案的工作空間」、「準備這週的技術週報」。
*   **重點**：Skill 擁有「編排 (Orchestration)」的主導權。

### 🔧 Tool-first (工具導向)
*   **定義**：用戶已經有工具（如 Notion MCP），Skill 提供的是該工具的「最佳實踐與專業知識」。
*   **類喻**：你買了一把新電鑽，看說明書如何正確使用它來鑽牆。
*   **場景**：「我連接了 Jira MCP，幫我建立標準的 Sprint Tasks」。
*   **重點**：Skill 提供「操作專業知識 (Expertise)」。

---

## 2. 五大 Workflow 設計模式 (Design Patterns)

### Pattern 1：循序工作流 (Sequential Workflow Orchestration)
**適用於**：需要按固定順序完成多步驟流程。

**範例結構：**
```markdown
# Workflow: Onboard New Customer

# Step 1: Create Account
Call MCP tool: `create_customer`
Parameters: name, email, company

# Step 2: Setup Payment
Call MCP tool: `setup_payment_method`
Wait for: payment method verification

# Step 3: Create Subscription
Call MCP tool: `create_subscription`
Parameters: plan_id, customer_id (from Step 1)

# Step 4: Send Welcome Email
Call MCP tool: `send_email`
Template: welcome_email_template
```
**關鍵技巧：**
*   **顯式步驟排序**：明確標註 Step 1, 2, 3。
*   **步驟依賴**：明確指出 Step 3 需要 Step 1 產出的 ID。
*   **階段驗證**：每一步完成後確認狀態。
*   **回滾指令**：定義失敗時如何復原。

---

### Pattern 2：跨服務協調 (Multi-MCP Coordination)
**適用於**：工作流跨越多個獨立的 MCP 服務。

**範例結構：**
```markdown
# Phase 1: Design Export (Figma MCP)
1. Export design assets from Figma
2. Generate design specifications
3. Create asset manifest

# Phase 2: Asset Storage (Drive MCP)
1. Create project folder in Drive
2. Upload all assets
3. Generate shareable links

# Phase 3: Task Creation (Linear MCP)
1. Create development tasks
2. Attach asset links to tasks
3. Assign to engineering team

# Phase 4: Notification (Slack MCP)
1. Post handoff summary to #engineering
2. Include asset links and task references
```
**關鍵技巧：**
*   **清晰的 Phase 劃分**：避免不同服務的指令混在一起。
*   **資料傳遞**：定義如何將 Figma 的 Link 帶入 Linear 任務。
*   **進入門檻**：Phase 2 開始前必須驗證 Phase 1 已產出 Manifest。

---

### Pattern 3：迭代精煉 (Iterative Refinement)
**適用於**：輸出品質隨迭代次數提升的任務（如報告內容）。

**範例結構：**
```markdown
# Iterative Report Creation

# Initial Draft
1. Fetch data via MCP
2. Generate first draft report
3. Save to temporary file

# Quality Check
1. Run validation script: `scripts/check_report.py`
2. Identify issues:
   - Missing sections
   - Inconsistent formatting
   - Data validation errors

# Refinement Loop
1. Address each identified issue
2. Regenerate affected sections
3. Re-validate
4. Repeat until quality threshold met

# Finalization
1. Apply final formatting
2. Generate summary
3. Save final version
```
**關鍵技巧：**
*   **量化品質標準**：不要說「檢查品質」，要用驗證腳本或具體 Checklist。
*   **定義終止條件**：知道何時該停止（例如達到 80 分或迭代 3 次）。

---

### Pattern 4：情境感知選擇 (Context-aware Tool Selection)
**適用於**：相同目標，但依不同條件選擇不同工具。

**範例結構：**
```markdown
# Smart File Storage

# Decision Tree
1. Check file type and size
2. Determine best storage location:
   - Large files (>10MB): Use cloud storage MCP
   - Collaborative docs: Use Notion/Docs MCP
   - Code files: Use GitHub MCP
   - Temporary files: Use local storage

# Execute Storage
Based on decision:
- Call appropriate MCP tool
- Apply service-specific metadata
- Generate access link

# Provide Context to User
Explain why that storage was chosen
```
**關鍵技巧：**
*   **決策邏輯清單**：明確寫出「大於 10MB 選 A」。
*   **透明度**：AI 必須告訴用戶它「為什麼」選這個工具。

---

### Pattern 5：領域專業知識 (Domain-specific Intelligence)
**適用於**：Skill 本身除了呼叫工具外，還包含專業判斷。

**範例結構：**
```markdown
# Payment Processing with Compliance

# Before Processing (Compliance Check)
1. Fetch transaction details via MCP
2. Apply compliance rules:
   - Check sanctions lists
   - Verify jurisdiction allowances
   - Assess risk level
3. Document compliance decision

# Processing
IF compliance passed:
  - Call payment processing MCP tool
  - Apply appropriate fraud checks
  - Process transaction
ELSE:
  - Flag for review
  - Create compliance case

# Audit Trail
- Log all compliance checks
- Record processing decisions
- Generate audit report
```
**關鍵技巧：**
*   **先合規後動作**：將專業限制硬性嵌入流程開頭。
*   **審計追蹤**：每一步決策都要留下記錄以便查對。

---

## 3. Troubleshooting 疑難排解

### ❌ Skill 無法上傳 (Wrong Skill won't upload)
*   **Error: "Could not find SKILL.md"**
    *   原因：檔名大小寫不對。
    *   解法：確保檔名是 `SKILL.md` 的全大寫格式。
*   **Error: "Invalid frontmatter"**
    *   原因：YAML 格式錯誤。
    *   解法：確認前後都有 `---`，引號有無閉合。
*   **Error: "Invalid skill name"**
    *   原因：名稱包含空格或大寫。
    *   解法：一律小寫並使用連字符 `-`。

### ❌ Skill 不觸發 (Skill doesn't trigger)
*   **原因**：`description` 欄位寫得太模糊或太廣泛。
*   **診斷**：直接問 AI：「你什麼時候會使用這個 Skill？」看 AI 引用什麼描述。
*   **解法**：
    1.  加入 trigger phrases（用戶常說的關鍵詞）。
    2.  明確寫出 "Use when..." 動機。
    3.  描述相關的檔案類型（如 .pdf, .json）。

### ❌ 指令不被遵循 (Instructions not followed)
*   **過於冗長**：指令超過 5,000 字會導致 AI 忽略細節。
*   **資訊埋沒**：關鍵驗證應放在最頂端，使用 `## CRITICAL` 標題。
*   **語言模糊**：
    *   Bad: "Validate things properly"
    *   Good: "CRITICAL: Before Step X, verify the project name is non-empty."
*   **AI 偷懶 (Laziness)**：在大任務中加入顯式的性能筆記（Take your time, Quality is more important than speed）。

---

## 4. 大型上下文優化 (Large Context Issues)

*   **漸進式揭露 (Progressive Disclosure)**：`SKILL.md` 僅保留核心。詳細文件移到 `references/` 資料夾。
*   **限制啟用量**：同一環境下啟用的 Skills 不要超過 20-50 個。
*   **Skill Packs**：依據功能將相關 Skills 打包成資料組。

---

## 5. 安全性指南

*   **防止 Path Traversal**：在執行任何系統操作 Skill 前，腳本內應實施路徑規範化（realpath）與白名單目錄檢查。
*   **確定的驗證**：與其用語言告訴 AI 檢查環境，不如綑綁一個 `.py` 或 `.sh` 腳本來執行確定性的硬性檢查。
