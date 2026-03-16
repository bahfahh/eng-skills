# Writing Requirements（requirements.md）

目的：先理解專案與 codebase，再提出有建設性的方案，最後產出 requirements.md。

建議位置：`docs/02_workflows/task/<NNN_task_name>/requirements.md`

---

## 1) 文件邊界

寫：As-is 現況分析、To-be 目標需求、非目標、驗收標準草案、影響分析
不寫：技術實作細節（→ plan.md）、測試策略（→ tests.md）、具體程式碼位置（→ plan.md）

---

## 2) 前置作業：讀取專案規格

依照專案結構讀取現有規格，常見位置：
- `.kiro/specs/*/requirements.md`（Kiro spec 系統）
- `spec.md` / `AGENTS.md` / `docs/memory/memoryindex.md`（其他專案）

若使用者需求與現有規格衝突，提出 2-3 個解決方案選項再繼續。

---

## 3) 確認原則

先調查 codebase，再提問。確認時：
- **優先提選擇題**，附 pros/cons 與推薦選項
- **只在 codebase 無法回答時**才開放提問
- 每個問題都要說明技術背景與影響範圍

---

## 4) 文件結構

```markdown
# <功能名稱> — Requirements

> 狀態：Draft / Approved

## A. As-is（現況）
現有功能如何運作、問題點、Mermaid 流程圖（如需要）

## B. To-be（目標）
核心需求、資料模型調整、整合策略

## C. Non-goals
明確不做什麼

## D. 驗收標準草案
具體可驗證的條件列表

## E. 影響分析
影響的模組、DB 變更、API 變更、風險評估
```

---

## 5) 完成條件（Gate）

- [ ] As-is 基於實際 codebase 分析，非猜測
- [ ] 需求具體可執行且無衝突
- [ ] 驗收標準清楚可測試
- [ ] 使用者確認所有關鍵決策點
