# The Complete Guide to Building Skills for Claude — 可參考重點（給本專案用）

本檔整理自：
- `The-Complete-Guide-to-Building-Skill-for-Claude.pdf`
- 文字抽出：`The-Complete-Guide-to-Building-Skill-for-Claude.extracted.txt`

目的：把 PDF 內「可直接轉成規範 / 模板 / 驗證器」的內容，轉成這個 `skillscreater` 專案後續可以落地的設計要點。

---

## 1) Skill 基本定義與設計原則（可當總則）

- Skill 是一個資料夾，核心是 `SKILL.md`（含 YAML frontmatter），可選 `scripts/`、`references/`、`assets/`。
- **Composability**：同時載入多個 skills 時也要能正常運作，不要假設自己是唯一能力。
- **Portability**：同一個 skill 在 Claude.ai / Claude Code / API 介面應盡量一致。
- **Progressive Disclosure（三層載入）**：
  1. Frontmatter（永遠預載）：只放「讓模型知道何時該用」的最小資訊
  2. `SKILL.md` 主體：觸發後才載入
  3. `references/` 等連結資源：需要時再載入

---

## 2) Planning & Design：先定 use cases 再寫（可當創建流程）

PDF 建議：先定 2–3 個具體用例，包含：
- Use Case：使用者想達成的成果
- Trigger：使用者可能會說的話（觸發語句）
- Steps：技能要做的步驟（含需要的工具 / MCP / 內建能力）
- Result：成功長相（可用來做 functional tests）

另外把 skill 需求歸類，有助於選結構與重點：
- Category 1：Document & Asset Creation（產出一致品質的文件/設計/程式等）
- Category 2：Workflow Automation（多步驟流程、需要驗證 gate、模板）
- Category 3：MCP Enhancement（在既有 MCP tool access 上「補 workflow + best practices」）

---

## 3) YAML frontmatter：觸發成敗的關鍵（可當 validator 規則）

### 必要欄位（最小可用）
```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

### name 規則（常見錯誤源）
- kebab-case、無空白、無大寫、無底線
- 通常應與資料夾名稱一致

### description 規則（要同時滿足）
- 必須同時包含：
  - **What**：這個 skill 做什麼（價值）
  - **When**：什麼情境要用（觸發條件/觸發語句）
- 需要包含「使用者真的會說」的 trigger phrases（避免 undertrigger）
- 太泛/太抽象/太技術導向會導致不觸發或亂觸發

### security / 禁止事項（適合做靜態檢查）
- Frontmatter 禁止出現 XML angle brackets：`<` / `>`（避免 prompt injection）
- name 含保留字（例如 `claude` / `anthropic` 前綴）屬限制情境
- YAML 不允許「可執行」內容（只允許標準 YAML 型別）

### 可選欄位（建議納入模板）
- `compatibility`：環境/依賴/網路等需求（例如需要哪些系統套件、是否需 network）
- `metadata`：作者、版本、mcp-server、tags 等（利於版本控管與分發）
- `allowed-tools`：工具存取限制（若你們的 runtime/平台支援）

---

## 4) 指令內容寫法：讓 Claude「照做」而不是「理解」（可當寫作規範）

PDF 方向偏向「Specific & Actionable」：
- 寫成可執行指令（例如：`Run ...`、`Verify ...`、`If ... then ...`）
- 重要約束放在最上面，用 `CRITICAL` / `Important` 類標題凸顯
- 內建錯誤處理段落（Common Issues / Troubleshooting）
- 明確引用 bundled resources（例如：需要查 API patterns 就連到 `references/api-patterns.md`）
- 需要確定性的檢查（validation）時，**可用 scripts 做 deterministic 檢查**，不要只靠語言描述

---

## 5) Testing & Iteration：把「會觸發 + 會做對」拆開驗（可當 evals 策略）

PDF 建議的三層測試思路：
1. Triggering tests：確保「該觸發會觸發、不該觸發不觸發」
   - ✅ obvious tasks
   - ✅ paraphrased requests
   - ❌ unrelated topics（負例必備）
2. Functional tests：確保輸出正確、工具調用成功、錯誤處理能用、涵蓋 edge cases
3. Performance comparison：和 baseline（沒 skill）相比，工具調用/來回次數/token 等是否變好

並有 troubleshooting 訊號：
- Undertriggering：描述太泛/沒有 trigger phrases
- Overtriggering：描述太廣 → 加 negative triggers、縮小 scope、寫清楚「不要用在…」

---

## 6) Distribution & Sharing：Repo README vs Skill folder（可當發佈準則）

PDF 提醒：
- Skill folder 內 **不要** 放 `README.md`（人類文件要放 GitHub repo 層級 README）
- 常見流程：下載 skill folder → 視需要 zip → Claude.ai 上傳（Settings > Capabilities > Skills）或放 Claude Code skills 目錄
- 也提到 2026-01 的分發現況與 org-level 部署（2025-12-18）等資訊（偏產品面參考）

---

## 7) Patterns（可當 skill skeleton 的幾種模版）

PDF 提到的常見結構模式（可變成 `skills/` 的模板集合）：
- Sequential workflow orchestration：明確 step ordering、依賴關係、每步驗證、失敗回滾
- Multi-MCP coordination：phase 分段、跨服務資料傳遞、集中錯誤處理
- Iterative refinement：初稿→品質檢查→修正→重跑→停損點
- Context-aware tool selection：決策樹（依檔案大小/型別/情境選工具）
- Domain-specific intelligence：把領域規則（合規/稽核/治理）內嵌在流程中

---

## 8) 建議本專案可直接落地的事項（下一步 TODO）

如果要把 PDF 的建議工程化，最有價值的三件事：
- 新增「skill 骨架模板」：`SKILL.md` 結構 + 常用段落（Triggers / Negative triggers / Steps / Common issues / Examples / References）
- 新增「靜態驗證器腳本」：檢查 frontmatter delimiters、name/description 規則、禁用字元、是否誤放 README.md、引用 references 是否存在等
- 強化 `evals/evals.json` 規範：每個 skill 至少 2–3 個用例，且必含 1 個不該觸發的負例（避免 overtrigger）
