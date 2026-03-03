# Category 1: Document & Asset Creation

> 產出一致、高品質的輸出物：文件、簡報、設計、程式碼、應用程式等。

---

## 特徵

- 不需要外部工具或 MCP，使用 Claude 內建能力即可
- 重點在**品質一致性**：每次產出都符合相同標準
- 通常包含樣式指南、模板結構、品質檢查清單

## 適用場景

- 從規格產出 frontend 設計
- 依團隊風格指南產出文件
- 產出報告、簡報、試算表
- 產出符合品牌標準的設計稿

## 官方範例

`frontend-design` skill：
```yaml
description: Create distinctive, production-grade frontend interfaces with high design quality. Use when building web components, pages, artifacts, posters, or applications.
```

其他：`docx`, `pptx`, `xlsx`, `ppt`

---

## 推薦設計模式

### Iterative Refinement（迭代改進）

最適合產出型——先產初稿，再驗品質，再修正。

```markdown
## Instructions

### Step 1: Generate Initial Draft
1. Parse user requirements
2. Apply style guide from `references/style-guide.md`
3. Generate first version

### Step 2: Quality Check
Run validation:
- Structure completeness
- Style consistency
- Content accuracy

### Step 3: Refinement
1. Address each identified issue
2. Regenerate affected sections
3. Re-validate
4. Repeat until quality threshold met

### Step 4: Finalize
1. Apply final formatting
2. Generate summary of what was created
```

### Context-Aware Tool Selection（依情境選工具）

當同一個 skill 需要產出不同格式時：

```markdown
### Output Format Decision
1. Check requested format:
   - Presentation → use `assets/pptx-template.md`
   - Document → use `assets/docx-template.md`
   - Spreadsheet → use `assets/xlsx-template.md`
2. Apply format-specific rules
3. Generate output
```

---

## Description 撰寫重點

- 列出**支援的輸出格式**（.docx, .pptx, frontend components 等）
- 包含使用者會說的動詞：`create`, `generate`, `design`, `build`
- 提到品質標準：`production-grade`, `brand-consistent`

```yaml
# ✅ Good
description: Generates production-grade frontend interfaces following team design system. Use when user asks to "create a page", "build a component", "design a UI", or uploads a design spec.

# ❌ Bad
description: Creates documents.
```

---

## 關鍵技巧

- 在 `references/` 放樣式指南與品牌標準
- 在 `assets/` 放模板檔案
- 用品質檢查清單確保每次產出一致
- 關鍵驗證寫成腳本（`scripts/validate.py`），不要只靠語言描述

---

## 測試重點

| 維度 | 檢查項目 |
|------|----------|
| 觸發 | 「幫我做一份簡報」「generate a report」能觸發 |
| 品質 | 多次產出結構一致、符合樣式指南 |
| 格式 | 輸出格式正確、可直接使用 |
| 邊界 | 缺少資訊時會詢問而非亂猜 |

---

## 常見問題

| 問題 | 解法 |
|------|------|
| 產出風格不一致 | 在 `references/` 放明確的樣式指南，SKILL.md 中引用 |
| 模板結構被忽略 | 用 `CRITICAL:` 標記模板規則，或用腳本驗證結構 |
| 品質時好時壞 | 加入明確的品質檢查清單，每次產出前逐項確認 |
