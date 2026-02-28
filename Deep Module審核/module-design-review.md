# Module Design Review Guidelines

AI agent code review 用途。審核時依序檢查以下三個面向，每條規則給出 `PASS` / `WARN` / `SUGGEST`。

---

## 1. 模組深度（Module Depth）

### 核心問題
> 呼叫端需要知道多少細節，才能完成一個業務操作？

### 規則

**[D-1] 單一業務操作應只需一次呼叫**

呼叫端執行一個完整的業務動作（如「建立觀察記錄」、「發送週報」），不應需要組合多個 function 才能完成。

```typescript
// WARN：呼叫端需要自己組合 3 步
const path = getFilePath(userId)
const raw = readFile(path)
const config = mergeDefaults(parse(raw))

// PASS：一次呼叫完成業務動作
const config = getUserConfig(userId)
```

**[D-2] 同一組 inputs 反覆出現在多個呼叫，應視為封裝訊號**

如果呼叫端在同一個 context 裡，對同一個物件連續呼叫多個 utils，代表這些 utils 之間有隱含的組合關係，應該提供 facade。

```typescript
// WARN：content 被傳給 3 個 utils，每次出現都是同樣的組合
createMessage({
  dateRange: getDateRangeLabel(content),
  highlights: getHighlights(content),
  counts: getDomainCounts(content),
})

// SUGGEST：提供 facade，隱藏組合細節
createMessage(prepareMessageParams(content))
```

**[D-3] 區分「自由工具」與「固定步驟序列」**

Utils 不一定要封裝，但需判斷它是**獨立工具**還是**固定序列的一部分**：

| 特徵 | 判斷 |
|------|------|
| 在不同地方、不同組合下出現 | 獨立工具，保持分離 ✅ |
| 幾乎每次都一起出現、順序固定 | 固定序列，應提供 facade ⚠️ |
| 只在一個地方被呼叫 | 考慮是否直接內聯到 facade 內部 |

**[D-4] 介面不應比實作複雜**

如果呼叫端需要傳入大量參數才能完成呼叫，而這些參數其實都能從更少的 inputs 推導出來，代表介面設計倒置了複雜度。

```typescript
// WARN：介面把內部步驟都暴露出來了
sendReport(reportId, lineToken, studentName, className, teacherName, pdfUrl, flexParams)

// PASS：複雜度留在內部，介面只要業務識別碼
sendReport(reportId)
```

---

## 2. 測試策略（Test Strategy）

### 核心問題
> 測試耦合在介面上，還是實作細節上？

### 規則

**[T-1] 主要測試對象應是 facade / deep module，不是 utils**

Facade 的測試驗證「給定業務輸入，產出正確業務結果」。Utils 的測試是補充，不是主力。

```
PASS：tests/api/observations.test.ts 測 createWithRelations 的完整行為
WARN：tests/utils/report.test.ts 每個 util 都有獨立測試，但 facade 層測試缺失
```

**[T-2] Utils 值得獨立測試的三個條件**

滿足其中一條才需要為 util 寫獨立測試：

1. **邊界條件複雜**：有明確的輸入分支（如日期跨週、null 處理、格式邊界）
2. **被多個 facade 共用**：改動影響範圍廣，需要獨立保護
3. **Pure function 且輸入輸出邊界清晰**：測試成本低，且是其他測試的基礎

**[T-3] 不應測試「有沒有呼叫到特定 util」**

測試不應依賴內部實作路徑。如果重構內部（把兩個 utils 合成一個），測試不應因此壞掉。

```typescript
// WARN：測試耦合實作
expect(getHighlights).toHaveBeenCalledWith(content)
expect(getDomainCounts).toHaveBeenCalledWith(content)

// PASS：測試耦合行為
expect(result.highlights).toEqual(['本週觀察重點...'])
expect(result.domainCounts.emotion).toBe(3)
```

**[T-4] 一個 facade 測試應涵蓋完整的業務場景**

不要為了覆蓋率把測試切太細。一個好的 facade 測試案例應能描述一個完整的使用情境。

```typescript
// PASS：完整場景描述
it('should skip sending if student has no LINE binding', async () => {
  // arrange: report exists, but no parent_binding
  // act: sendWeeklyReportViaLine(reportId)
  // assert: result.status === 'skipped', result.reason === 'no_binding'
})
```

---

## 3. 命名與介面設計（Naming & Interface）

### 核心問題
> 光看函數名稱，能知道它做什麼業務事情嗎？

### 規則

**[N-1] 函數名稱應反映業務行為，而非技術操作**

```typescript
// WARN：技術導向命名
parseAndMergeConfig(raw)
buildQueryWithFilters(supabase, orgId)

// PASS：業務導向命名
getUserConfig(userId)
getObservations(orgId, filters)
```

**[N-2] Facade 應有獨立的命名，不是 util 的前綴組合**

如果你的 facade 叫 `getAndFormatAndMergeReport`，代表它只是把 utils 串起來，沒有真正封裝業務概念。

```typescript
// WARN：名稱只是步驟描述
parseAndNormalizeContent(raw)

// PASS：名稱描述業務概念
normalizeReportContent(raw)   // 「使報告內容符合可用格式」是個業務概念
```

**[N-3] 介面參數應使用業務語言，不應洩漏儲存層細節**

```typescript
// WARN：介面洩漏了 Supabase row 結構
createObservation(supabase, orgId, { observation_date, created_by, organization_id })

// PASS：介面用業務語言
createObservation(orgId, { date, teacherId, note, studentIds, tagIds, photos })
```

**[N-4] 回傳型別應穩定，不隨底層查詢結構變動**

Facade 的回傳型別應該是業務定義的型別（如 `ObservationWithRelations`），而不是 Supabase query 的原始結構。這樣底層換 join 方式時，呼叫端不受影響。

---

## 審核執行方式

對每個被審核的模組，逐條給出結果：

```
[D-1] PASS  — createWithRelations 一次呼叫完成完整業務操作
[D-2] WARN  — report-sender.ts 對 content 連續呼叫 3 個 utils，建議提供 facade
[D-3] PASS  — week.ts 的三個函數有不同使用場景，保持分離合理
[T-1] WARN  — lib/utils/report.test.ts 存在但對應 facade 測試缺失
[T-3] PASS  — 測試驗證行為輸出，未 mock 內部 utils
[N-1] PASS  — 函數命名均反映業務動作
```

發現 `WARN` 時，給出具體的重構建議（不強制，需說明理由）。
