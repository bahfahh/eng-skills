# Writing Plan（plan.md / todolist.md）

目的：把需求轉成「可執行、可追蹤、可並行」的開發計畫。Plan 只負責 *怎麼做*，不重複寫 *要做什麼*（那是 requirement.md 的事）。

建議位置：`docs/02_workflows/task/<NNN_task_name>/plan.md`

---

## 1) Plan 文件的邊界（不要混到 requirement）

Plan 文件寫：
- 技術方案、架構決策
- 修改的檔案路徑、模組
- 使用的套件/工具
- 實作步驟（Phase → Task）
- 每個 Phase 的 Success Criteria
- 風險與緩解策略
- 時間估算

Plan 文件不寫：
- 需求背景、使用者故事（放到 `requirement.md`）
- 驗收標準細節（放到 `acceptance.md`）
- 測試案例細節（放到 `tests.md`）

---

## 2) 寫作前置作業（必做）

開始寫 plan 之前：
1. **理解專案架構** — 讀 `AGENTS.md`、`.kiro/steering/`
2. **讀取開發記憶** — 讀 `docs/memory/memoryindex.md`，避免重複踩坑
3. **確認 requirement 已 Approved** — 不要在需求未定時開始規劃

---

## 3) 工程流程循環

```
Requirement → Phase → Task → Do → Check → Test → Verify Success Criteria → Next Phase
```

每個 Phase 完成後，必須驗證該 Phase 的 Success Criteria 才能進入下一階段。

---

## 4) Plan 文件模板

```markdown
# <Task Name> — Implementation Plan

> 狀態：Draft / In Progress / Completed
> 對應需求：`<NNN>_requirement.md`

## A. Problem Summary
從工程角度說明問題。
- 現有問題/改進需求：
- 根因分析（架構缺陷/資料流不匹配/模組耦合等）：

## B. Technical Approach
- 選用方案：
- 方案比較（如有多個選項）：
- 架構決策理由：

## C. Implementation Plan

### Phase 1: <Phase Name>

**目標**：（這個 Phase 要達成什麼）

**修改範圍**：
- `path/to/file.ts` — 說明
- `path/to/another.ts` — 說明

**Tasks**：
- [ ] **Task 1.1**: Create `<module>`
  - File: `src/...`
  - Description: ...
  - Validation: ...

- [ ] **Task 1.2**: Refactor `<module>`
  - File: `src/...`
  - Description: ...
  - Validation: ...

**✅ Success Criteria (Phase 1)**：
- [ ] Criteria 1
- [ ] Criteria 2

---

### Phase 2: <Phase Name>
（同上格式）

---

## D. Potential Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| 技術限制 | High | ... |
| 系統整合風險 | Medium | ... |
| 效能/相容性 | Low | ... |

## E. Execution Order & Time Estimate

| Phase | Description | Estimated Time | Risk Level | Dependencies |
|-------|-------------|----------------|------------|--------------|
| 1 | Core refactor | 2 days | Medium | None |
| 2 | Data validation | 1 day | Low | Phase 1 |

## F. Parallel Execution Plan
（哪些可以並行、如何分配 agent）

- **Agent A**: Unit tests (Phase 1 tasks)
- **Agent B**: Frontend implementation (Phase 1 tasks)
- **Sync Point**: Phase 1 完成後合併驗證

---

## G. Checklist

**Before Development**
- [ ] Requirement confirmed (requirement.md Approved)
- [ ] Design reviewed
- [ ] Memory/steering 已讀取
- [ ] Test plan prepared (tests.md)

**During Development**
- [ ] Code follows naming and style guide
- [ ] Intermediate results validated
- [ ] No breaking changes introduced
- [ ] Todolist.md 持續更新

**After Development**
- [ ] All tests passed
- [ ] Each phase's Success Criteria verified
- [ ] Documentation and comments updated
- [ ] acceptance.md 驗收通過
```

---

## 5) Todolist 文件模板

Todolist 是 Plan 的執行追蹤版本，更輕量、更適合日常更新。

```markdown
# <Task Name> — Todolist

> 對應 Plan：`plan.md`
> 最後更新：YYYY-MM-DD

## Current Phase: Phase X

### In Progress
- [ ] Task X.X — @agent-name — 進度說明

### Completed
- [x] Task X.X — @agent-name — ✅ YYYY-MM-DD

### Blocked
- [ ] Task X.X — 原因：...

---

## Phase Summary

| Phase | Status | Completion |
|-------|--------|------------|
| 1 | ✅ Done | 100% |
| 2 | 🔄 In Progress | 60% |
| 3 | ⏳ Pending | 0% |

---

## Parallel Workflow

```
Timeline:
─────────────────────────────────────────
Agent A: [Task 1.1] [Task 1.3] [Task 2.1]
Agent B: [Task 1.2] [Task 1.4] [Task 2.2]
─────────────────────────────────────────
         ↑ Sync    ↑ Sync     ↑ Final
```

## Notes
- 開發中發現的問題/決策變更記錄在這
```

---

## 6) Task 動詞規範

使用明確的動作動詞：
- **Create** — 新建檔案/模組
- **Refactor** — 重構現有程式
- **Update** — 更新/修改
- **Fix** — 修復 bug
- **Integrate** — 整合模組
- **Add** — 新增功能
- **Remove** — 移除程式碼
- **Migrate** — 遷移資料/結構

---

## 7) 完成條件（Gate）

滿足才算「Plan 可進入開發」：
- [ ] 所有 Phase 都有明確的 Success Criteria
- [ ] 風險已識別並有緩解策略
- [ ] 時間估算合理
- [ ] 並行計畫已規劃（如適用）
- [ ] Owner 確認可開工
