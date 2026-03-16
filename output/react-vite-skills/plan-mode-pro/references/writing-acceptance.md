# Writing Acceptance（acceptance.md）

目的：將需求文件的驗收標準草案轉化為可執行的 BDD 驗收場景。

建議位置：`docs/02_workflows/task/<NNN_task_name>/acceptance.md`

---

## 1) 前置作業

讀取專案的需求文件，依照專案結構選擇：
- `.kiro/specs/*/requirements.md`（Kiro spec 系統）
- 或其他 `requirements.md` / `spec.md`
- 加上本 workflow Step 1 產出的 `requirement.md`（D 節驗收標準草案）

從需求文件的驗收標準草案推導場景，不憑空假設 Given 條件。

---

## 2) 場景覆蓋範圍

每個主要功能至少覆蓋：

| 類型 | 說明 |
|------|------|
| Happy Path | 正常流程，每條驗收標準對應至少一個場景 |
| Error Cases | 無效輸入、權限不足、資源不存在 |
| Edge Cases | 邊界值、空狀態、剛好達到上限 |

---

## 3) 文件結構

```markdown
# <Task Name> — Acceptance Criteria

> 對應需求：`<NNN>_requirement.md`
> 狀態：Draft / Ready / Verified

## A. Happy Path
## B. Error Cases
## C. Edge Cases
## D. 手動驗收清單（無法自動化的情境）
## E. 驗收完成條件
```

---

## 4) 完成條件（Gate）

- [ ] 每條驗收標準都有對應的 Happy Path Scenario
- [ ] Then 描述全部可觀察、可驗證（不寫「系統正常運作」）
- [ ] Error / Edge cases 已識別並記錄
- [ ] 手動驗收項目列出操作步驟
