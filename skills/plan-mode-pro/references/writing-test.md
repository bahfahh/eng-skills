# Writing Tests（tests.md）

目的：依照 `plan.md` 和 `requirement.md` 建立測試文件，指導 AI agent 撰寫測試程式碼，支援工程師 TDD 開發。

建議位置：`docs/02_workflows/task/<NNN_task_name>/tests.md`

---

## 1) Tests 文件的邊界（不要混到其他文件）

Tests 文件寫：
- 測試策略和分層
- 具體測試檔案清單和位置
- 測試案例描述（對應驗收標準）
- Mock 策略和測試資料需求
- AI agent 執行指令

Tests 文件不寫：
- 需求背景（放到 `requirement.md`）
- 實作技術細節（放到 `plan.md`）
- 驗收標準細節（放到 `acceptance.md`）
- 實際測試程式碼（由 AI agent 依此文件生成）

---

## 2) 寫作前置作業（必做）

開始寫 tests.md 之前：
1. **讀取專案測試規範** — `.kiro/steering/testing-standards.md`、`.kiro/steering/testing/property-based-testing-guide.md`
2. **理解測試架構** — Property-Based (50%) > Integration (25%) > Unit (20%) > E2E (5%)
3. **確認 requirement 和 plan 已完成** — 測試策略基於需求和實作計畫

---

## 3) 專案測試決策樹

```
這個功能需要什麼測試？
│
├─ 有業務規則永遠成立？
│  └─ YES → Property-Based Test（主要）
│
├─ 涉及多個組件協作？
│  └─ YES → Integration Test
│
├─ 是完整使用者流程？
│  └─ YES → E2E Test
│
└─ 是純函數/工具？
   └─ YES → Unit Test
```

---

## 4) Tests 文件模板

```markdown
# <Task Name> — Test Strategy

> 對應需求：`<NNN>_requirement.md`
> 對應計畫：`plan.md`
> 狀態：Draft / Ready / Completed

## A. 測試目標

從 requirement.md 提取的核心驗證點：
- 驗證需求 AC X.Y: [簡述]
- 確保業務規則: [關鍵不變性]
- 防止回歸: [風險點]

## B. 測試分層策略

| 測試類型 | 數量 | 主要目的 | 檔案位置 |
|---------|------|----------|----------|
| Property-Based | X 個 | 業務規則驗證 | `tests/unit/lib/[domain]/` |
| Integration | Y 個 | 組件協作 | `tests/integration/components/` |
| Unit | Z 個 | 工具函數 | `tests/unit/lib/utils/` |
| E2E | W 個 | 完整流程 | `tests/e2e/` |

---

## C. Property-Based Tests（主要）

### C.1 業務不變性清單

從 requirement.md 驗收標準提取：

| 需求 AC | 業務不變性描述 | 測試檔案 |
|---------|---------------|----------|
| 1.1 | 觀察記錄按建立時間降序排列 | `observation-properties.test.ts` |
| 1.2 | 標籤選擇限制 1-3 個 | `tag-selection-properties.test.ts` |
| 2.1 | 多租戶資料完全隔離 | `isolation-properties.test.ts` |

### C.2 Property Tests 檔案清單

#### `tests/unit/lib/observations/observation-properties.test.ts`
**測試目標**: 觀察記錄核心業務規則
**需要的 arbitraries**: `observationArbitrary`, `createObservationArbitrary`
**Property 清單**:
- 排序一致性（任意觀察記錄陣列都按時間降序）
- 資料完整性（建立的觀察記錄包含所有必填欄位）
- 關聯一致性（觀察記錄與學生/標籤的關聯正確）

#### `tests/unit/lib/isolation/isolation-properties.test.ts`
**測試目標**: 多租戶隔離規則
**需要的 arbitraries**: `userArbitrary`, `resourceArbitrary`
**Property 清單**:
- 權限邊界（使用者只能存取同組織資源）
- 資料隔離（跨組織資料完全不可見）

### C.3 資料生成器需求

需要在 `tests/__mocks__/arbitraries.ts` 建立或更新：
```typescript
// 新增或修改的 arbitraries
export const observationArbitrary = fc.record({...})
export const userArbitrary = fc.record({...})
export const studentArbitrary = fc.record({...})
```

---

## D. Integration Tests

### D.1 使用者流程清單

從 requirement.md 使用者故事提取：

| 使用者故事 | 測試場景 | 檔案位置 |
|-----------|----------|----------|
| 教師建立觀察記錄 | 完整 CRUD 流程 | `observation-crud.integration.test.tsx` |
| 園長查看統計 | Dashboard 載入和互動 | `dashboard.integration.test.tsx` |
| 行動裝置操作 | 觸控介面流程 | `mobile-interface.integration.test.tsx` |

### D.2 Integration Tests 檔案清單

#### `tests/integration/components/observation-crud.integration.test.tsx`
**測試目標**: 觀察記錄 CRUD 完整流程
**Mock 需求**: Supabase client, file upload
**測試案例**:
- 建立觀察記錄（多學生選擇、照片上傳、標籤選擇）
- 編輯自己的觀察記錄
- 刪除自己的觀察記錄
- 拒絕編輯他人觀察記錄

#### `tests/integration/components/mobile-interface.integration.test.tsx`
**測試目標**: 行動裝置介面優化
**Mock 需求**: viewport, touch events
**測試案例**:
- 觸控友善控制項
- 拇指可及範圍操作
- 大型照片選擇按鈕

---

## E. Unit Tests

### E.1 工具函數清單

從 plan.md 實作細節提取：

| 函數/模組 | 測試重點 | 檔案位置 |
|----------|----------|----------|
| `formatDate` | 台灣日曆格式轉換 | `tests/unit/lib/utils/format.test.ts` |
| `compressImage` | 圖片壓縮品質和大小 | `tests/unit/lib/utils/image.test.ts` |
| `observationsApi` | API 呼叫和錯誤處理 | `tests/unit/lib/api/observations.test.ts` |

---

## F. Mock 策略

### F.1 外部依賴 Mock

| 依賴 | Mock 檔案 | Mock 重點 |
|------|----------|-----------|
| Supabase | `tests/__mocks__/supabase.ts` | 資料庫操作、認證 |
| Next.js Router | `tests/__mocks__/next-navigation.ts` | 路由導航 |
| File Upload | `tests/__mocks__/file-api.ts` | 檔案上傳 |

### F.2 測試資料

需要在 `tests/__mocks__/test-data.ts` 準備：
```typescript
export const mockObservation = { id: 'test-1', content: '...', ... }
export const mockUser = { id: 'user-1', role: 'teacher', ... }
export const mockStudent = { id: 'student-1', name: '小明', ... }
```

---

## G. AI Agent 執行指令

### G.1 Property-Based Tests 生成指令
```
任務：為 [feature] 建立 Property-Based Tests
參考：tests.md C 節
步驟：
1. 讀取 `tests/__mocks__/arbitraries.ts` 了解現有資料生成器
2. 依照 C.2 節建立測試檔案
3. 實作 C.1 節列出的業務不變性驗證
4. 每個 property 執行 100 次以上
5. 標註對應需求：`// Feature: requirement-X, Property Y: description`
```

### G.2 Integration Tests 生成指令
```
任務：為 [feature] 建立 Integration Tests
參考：tests.md D 節
步驟：
1. 使用 @testing-library/react 和 @testing-library/user-event
2. 依照 F.1 節設定必要的 Mock
3. 實作 D.2 節列出的測試案例
4. 模擬完整使用者操作流程
5. 驗證 UI 狀態變化和資料更新
```

### G.3 Unit Tests 生成指令
```
任務：為 [module] 建立 Unit Tests
參考：tests.md E 節
步驟：
1. 測試 E.1 節列出的函數/模組
2. 涵蓋正常情況、邊界條件、錯誤情況
3. 使用適當的 Mock 隔離外部依賴
4. 確保 100% 程式碼覆蓋率
```

---

## H. TDD 開發流程

### H.1 開發順序
```
1. 依照 tests.md 建立 Property-Based Tests（紅燈）
2. 寫最小實作讓測試通過（綠燈）
3. 重構程式碼（保持綠燈）
4. 建立 Integration Tests
5. 完整功能實作
6. 依照 acceptance.md 驗收
```

### H.2 並行開發策略
- **Agent A**: Property-Based Tests + Unit Tests
- **Agent B**: Integration Tests + 功能實作
- **同步點**: 所有測試通過後整合驗證

---

## I. 驗收檢查清單

**Property-Based Tests**
- [ ] 所有業務不變性都有對應測試
- [ ] 資料生成器涵蓋所有領域模型
- [ ] 每個 property 標註需求 AC 引用
- [ ] 執行次數 ≥ 100 次

**Integration Tests**
- [ ] 所有使用者故事都有對應測試
- [ ] Mock 策略完整且可執行
- [ ] 測試涵蓋正常和錯誤流程
- [ ] 行動裝置特殊情況已測試

**Unit Tests**
- [ ] 工具函數達到 100% 覆蓋率
- [ ] API 層完整測試
- [ ] 邊界條件和錯誤處理測試

**整體品質**
- [ ] 測試檔案命名符合專案規範
- [ ] 需求追溯標註完整
- [ ] 所有測試執行通過
- [ ] 總覆蓋率 ≥ 80%
```

---

## 5) 與其他文件的協作關係

- **requirement.md** → 提供驗收標準 → **tests.md** → 轉換為測試案例清單
- **plan.md** → 提供實作模組 → **tests.md** → 決定測試範圍和策略
- **tests.md** → 指導 AI agent → 生成實際測試程式碼
- **acceptance.md** → 最終驗收標準 → 確認測試覆蓋完整性

---

## 6) 完成條件（Gate）

滿足才算「Tests 文件可進入開發」：
- [ ] 所有需求 AC 都有對應測試類型和檔案
- [ ] Property-Based Tests 涵蓋所有業務不變性
- [ ] Integration Tests 涵蓋所有使用者流程
- [ ] Mock 策略明確且可實作
- [ ] AI agent 指令清楚可執行
- [ ] TDD 開發流程可操作
