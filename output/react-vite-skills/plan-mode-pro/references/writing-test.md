# Writing Tests（tests.md）

目的：依照 requirements.md 和 plan.md 建立測試策略文件，指導測試程式碼的撰寫方向。

建議位置：`docs/02_workflows/task/<NNN_task_name>/tests.md`

---

## 1) 文件邊界

寫：測試分層策略、測試檔案清單與位置、Mock 策略、TDD 開發順序
不寫：需求背景（→ requirements.md）、實作細節（→ plan.md）、驗收場景（→ acceptance.md）、實際測試程式碼

---

## 2) 前置作業

讀取專案測試規範（`.kiro/steering/testing-standards.md` 或同等文件）。確認 requirements.md 和 plan.md 都已完成。

---

## 3) 測試分層決策

依照專案測試規範選擇分層比例。常見判斷邏輯：

```
有業務規則永遠成立？     → Property-Based Test
涉及多個組件協作？       → Integration Test
是完整使用者流程？       → E2E Test
是純函數/工具？          → Unit Test
```

---

## 4) tests.md 結構

```markdown
# <Task Name> — Test Strategy

## A. 測試目標
從 requirements.md 驗收標準提取核心驗證點與業務不變性。

## B. 測試分層策略
| 測試類型 | 數量 | 主要目的 | 檔案位置 |

## C. Property-Based Tests（如適用）
業務不變性清單，對應需求 AC，列出 arbitraries 需求。

## D. Integration Tests
使用者流程清單，每個流程對應測試檔案與 Mock 需求。

## E. Unit Tests
純函數/工具模組清單，測試重點。

## F. Mock 策略
外部依賴清單（DB、Auth、第三方 API）與 mock 位置。

## G. TDD 開發順序
先寫哪層測試、並行策略（Agent A / Agent B）、Sync Point。
```

---

## 5) 完成條件（Gate）

- [ ] 每條需求 AC 都有對應測試類型和檔案
- [ ] Mock 策略明確可實作
- [ ] TDD 開發順序清楚
- [ ] 測試覆蓋正常流程、錯誤情況、邊界條件
