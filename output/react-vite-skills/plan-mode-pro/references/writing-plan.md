# Writing Plan（plan.md / todolist.md）

目的：把需求轉成可執行、可追蹤、可並行的開發計畫。Plan 只負責「怎麼做」，不重複寫「要做什麼」。

建議位置：`docs/02_workflows/task/<NNN_task_name>/plan.md`

---

## 1) 文件邊界

寫：技術方案、架構決策、修改的檔案路徑、使用套件、Phase → Task、每個 Phase 的 Success Criteria、風險緩解
不寫：需求背景（→ requirements.md）、驗收標準細節（→ acceptance.md）、測試案例（→ tests.md）

---

## 2) 前置作業

讀取專案架構與慣例（`AGENTS.md`、`.kiro/steering/` 或同等文件），以及開發記憶（避免重複踩坑）。確認 requirements.md 已 Approved 再開始。

---

## 3) 工程流程

```
Requirement → Phase → Task → Do → Check → Test → Verify Success Criteria → Next Phase
```

每個 Phase 必須驗證 Success Criteria 才能進入下一階段。

---

## 4) plan.md 結構

```markdown
# <Task Name> — Implementation Plan

> 狀態：Draft / In Progress / Completed

## A. Problem Summary
從工程角度說明問題與根因。

## B. Technical Approach
選用方案、架構決策理由、方案比較（如有多個選項）。

## C. Implementation Phases

### Phase 1: <Name>
**目標**：
**修改範圍**：`path/to/file` — 說明
**Tasks**：
- [ ] Task 1.1: <動詞 + 模組>
- [ ] Task 1.2: <動詞 + 模組>
**✅ Success Criteria**：

### Phase 2: <Name>
（同上）

## D. Risks & Mitigation
| Risk | Impact | Mitigation |

## E. Parallel Execution Plan
Agent A / Agent B 分工，Sync Point 說明。
```

Task 動詞用：Create / Refactor / Update / Fix / Add / Remove / Migrate / Integrate

---

## 5) todolist.md 結構

Todolist 是 plan 的輕量追蹤版，開發中持續更新。

```markdown
# <Task Name> — Todolist

## Current Phase: Phase X

### In Progress / Completed / Blocked
- [ ] Task X.X — @agent — 說明

## Phase Summary
| Phase | Status | Completion |

## Notes
開發中的決策變更記錄在這。
```

---

## 6) 完成條件（Gate）

- [ ] 每個 Phase 都有明確 Success Criteria
- [ ] 風險已識別並有緩解策略
- [ ] 並行計畫已規劃（如適用）
