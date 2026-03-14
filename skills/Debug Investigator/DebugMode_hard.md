
# Systematic Debug Protocol

## Phase 0: Problem Intake
### 0.1 Initial Report
- **Bug Description**: [clear, specific description]
- **Environment**:
  - OS: [operating system]
  - Framework/Language: [version]
  - Branch/Commit: [git info]
- **Reproduction Steps**:
  1. [step 1]
  2. [step 2]
  3. [observed behavior]
- **Expected Behavior**: [what should happen]
- **Actual Behavior**: [what actually happens]

### 0.2 Initial Observations
- Error messages: [any errors in console/logs]
- Affected scope: [UI, API, database, etc.]
- Frequency: [always / intermittent / specific conditions]

---

## Phase 1: Problem Analysis & Scope Definition

### 1.1 Symptom Analysis
**What we know:**
- [observable symptom 1]
- [observable symptom 2]

**What we don't know yet:**
- [question 1]
- [question 2]

### 1.2 Component Mapping
Identify all components involved in the failure path:
```
[User Action] → [Component A] → [Component B] → [Expected Result ❌]
                     ↓              ↓
                [Dependency 1]  [Dependency 2]
```

### 1.3 Narrowing Strategy
Based on symptom, the bug is most likely in:
- **Primary suspects**: [layer/component most likely affected]
- **Secondary suspects**: [backup possibilities]
- **Unlikely but possible**: [edge cases]

---

## Phase 2: Investigation & Data Collection

### 2.1 Logging & Tracing
- [ ] Enable detailed logging for [specific component]
- [ ] Check network requests/responses
- [ ] Examine state changes at critical points
- [ ] Review recent code changes in affected area

### 2.2 Evidence Gathered
```
[Timestamp/Location]: [finding 1]
[Timestamp/Location]: [finding 2]
```

### 2.3 Pattern Recognition
- Does it fail consistently? [yes/no]
- Is it data-dependent? [yes/no]
- Is it timing-dependent? [yes/no]

---

## Phase 3: Hypothesis Generation & Checklist Creation

### 3.1 Create Debug Checklist Document
**Create file**: `/docs/debug/[YYYY-MM-DD]_[bug-identifier].md`

### 3.2 Hypothesis Checklist
Based on investigation, test these possibilities in order:

**High Probability (Test First)**
- [ ] **H1**: [Hypothesis 1 - most likely cause]
  - Why: [reasoning based on evidence]
  - How to test: [specific test method]
  - Status: ⏳ Pending / ✅ Ruled Out / ❌ Confirmed Root Cause

- [ ] **H2**: [Hypothesis 2]
  - Why: [reasoning]
  - How to test: [test method]
  - Status: ⏳

**Medium Probability**
- [ ] **H3**: [Hypothesis 3]
  - Why: [reasoning]
  - How to test: [test method]
  - Status: ⏳

**Low Probability (Test if others fail)**
- [ ] **H4**: [Hypothesis 4]
  - Why: [reasoning]
  - How to test: [test method]
  - Status: ⏳

### 3.3 Elimination Logic
```
Scope: [Full Component]
  ↓ (Test H1) ✅ Ruled out
Scope: [Sub-component A, B]
  ↓ (Test H2) ❌ Found issue in B
Scope: [Specific function in B]
  ↓ (Root cause identified)
```

---

## Phase 4: Fix → Test → Verify Loop

### 4.1 Current Iteration: [#N]
**Testing Hypothesis**: [HX - description]

#### 4.1.1 Fix Implementation
- **Change**: [what will be modified]
- **Files affected**:
  - `path/to/file1.ext`
  - `path/to/file2.ext`
- **Code changes**:
```diff
- [old code]
+ [new code]
```
- **Rationale**: [why this should address HX]

#### 4.1.2 Test Execution
**Test Case 1**: [original reproduction steps]
- Steps: [1, 2, 3...]
- Expected: [correct behavior]
- Actual: [what happened]
- Result: ✅ Pass / ❌ Fail

**Test Case 2**: [edge case]
- Steps: [1, 2, 3...]
- Expected: [correct behavior]
- Actual: [what happened]
- Result: ✅ Pass / ❌ Fail

**Test Case 3**: [regression check]
- Steps: [ensure nothing else broke]
- Result: ✅ Pass / ❌ Fail

#### 4.1.3 Iteration Result
- [ ] **All tests passed** → Proceed to Phase 5
- [ ] **Tests failed** → Update checklist and loop

**If failed:**
- Update `/docs/debug/[bug-identifier].md`
- Mark HX as: ✅ Ruled Out (not the cause) or 🔄 Partially Correct (refine approach)
- Move to next hypothesis or refine current one
- Document what we learned: [new insight]

---

## Phase 5: Final Verification & Cleanup

### 5.1 Comprehensive Testing
- [ ] Original bug reproduction → ✅ Fixed
- [ ] Edge cases → ✅ No new issues
- [ ] Related features → ✅ No regression
- [ ] Performance impact → ✅ Acceptable
- [ ] Cross-browser/platform → ✅ Consistent

### 5.2 Code Quality Check
- [ ] Code follows project conventions
- [ ] Added/updated tests
- [ ] Updated documentation if needed
- [ ] No console warnings/errors
- [ ] Cleaned up debug/temporary code

---

## Phase 6: Resolution & Documentation

### 6.1 Root Cause Summary
**Confirmed Root Cause**: [HX - the actual problem]

**Why it happened**:
[Detailed explanation of the underlying issue]

**Why it wasn't caught earlier**:
[Gap in testing, edge case, recent change, etc.]

### 6.2 Solution Applied
- **PR/Commit**: [link]
- **Files changed**: [list]
- **Fix description**: [what was done]

### 6.3 Prevention Measures
**Immediate**:
- [What we can do now to prevent similar issues]

**Long-term**:
- [Tests to add]
- [Documentation to improve]
- [Architecture changes to consider]

### 6.4 Lessons Learned
1. [Key insight 1]
2. [Key insight 2]
3. [What would speed up debugging next time]

### 6.5 Checklist Archive
- Move `/docs/debug/[bug-identifier].md` to `/docs/debug/resolved/`
- Final checklist state shows investigation path taken

---

## Quick Reference: Debug Decision Tree
```
Bug Reported
    ↓
Can you reproduce?
    NO → Gather more info (back to Phase 0)
    YES ↓
    ↓
Create hypothesis checklist (Phase 3)
    ↓
Test hypothesis H1
    ↓
    ├→ Fixed? YES → Verify all tests (Phase 5) → Done
    └→ NO → Mark H1 status
            ↓
            Learn from failure
            ↓
            ├→ Refine H1? → Test refined H1
            └→ Move to H2 → Test H2
                (Repeat until fixed)
```

---

## Status Indicators
- ⏳ **Pending**: Not yet tested
- 🔍 **Testing**: Currently investigating
- ✅ **Ruled Out**: Confirmed not the cause
- 🔄 **Refining**: Partially correct, needs adjustment
- ❌ **Root Cause**: Confirmed as the problem
- ✔️ **Resolved**: Fixed and verified

## Debugging Guidance (Feedback-Driven)

The core of this debugging approach is not a strict requirement to write tests, but to ensure that **every code change produces sufficient feedback to quickly validate or invalidate a hypothesis**. The process starts by analyzing the problem and forming hypotheses based on observable signals. Each iteration should apply the smallest possible change targeting a single hypothesis.

After the change, validation must rely on **existing system feedback**, such as server logs, error messages, state transitions, metrics, or externally observable behavior. These signals are the primary source of truth for determining whether the hypothesis was correct.

In some cases, writing a small script or test can significantly reduce validation cost by making the feedback faster, repeatable, and less manual. Scripts and tests are therefore **acceleration tools, not mandatory requirements**. If the available feedback is already clear—such as an error log disappearing, a critical state updating correctly, or an API response matching expectations—additional tests are unnecessary.

The workflow is an iterative loop: analyze → hypothesize → modify → collect feedback. If the feedback contradicts the hypothesis or is insufficient to make a decision, the next step is to refine the hypothesis or improve observability before continuing. The goal of debugging is not to follow a rigid process, but to **obtain reliable feedback at the lowest possible cost** in order to confirm or eliminate causes efficiently.



</Instructions>

<document>

---
name: Universal Debug Methodologies
description: Language-agnostic debugging strategies and thought processes. Provides conceptual approaches that can be adapted to any programming language or framework.
---

# Universal Debug Methodologies

## Philosophy
Debug 的核心不是「寫更多代碼」，而是「讓錯誤無所遁形」。
在 AI 輔助時代，我們可以用「空間換時間」、「冗餘換精確」的策略快速定位問題。

---

## Method 1: 錄影機模式 - 狀態快照流 (State Snapshot Streaming)

### 核心思路
**問題本質**: Bug 通常發生在「狀態變化過程」中,而非最終結果
**解決邏輯**: 不只記錄終點,而是記錄整條路徑

### 概念模型
```
[正常思維]
Step 1 → Step 2 → Step 3 → ❌ 錯誤
只看到最後的錯誤訊息

[錄影機思維]
Step 1 📸 → Step 2 📸 → Step 3 📸 → ❌ 錯誤
擁有每一步的完整狀態快照
```

### 實作思路 (與語言無關)

**關鍵動作**:
1. **插入點選擇**: 在所有「會改變狀態」的地方前後插入快照邏輯
   - 變數賦值前後
   - 函式呼叫前後
   - 條件判斷前後
   - 迴圈每次迭代

2. **深拷貝原則**: 必須完整複製當前狀態,避免被後續操作污染
   - 物件/陣列需要深層複製
   - 避免只複製引用

3. **儲存策略**: 將快照存入記憶體集合
   - 只在開發/測試環境啟用
   - 設定快照上限 (如最近 100 筆) 避免記憶體爆炸
   - 錯誤發生時導出完整歷史

### 語言實作差異對照

| 語言 | 深拷貝方法 | 序列化工具 |
|------|-----------|-----------|
| C# | `JsonConvert.SerializeObject()` | Newtonsoft.Json |
| TypeScript/JavaScript | `JSON.parse(JSON.stringify())` | 原生 JSON |
| Python | `copy.deepcopy()` | `json.dumps()` |
| Java | `ObjectMapper.writeValueAsString()` | Jackson |
| Go | `json.Marshal()` | encoding/json |

### 價值分析
- **傳統方式**: 看到錯誤訊息後,猜測哪一步出錯,手動加 log,重現,重複 N 次
- **錄影機方式**: 一次執行就獲得完整軌跡,直接對比哪一步狀態開始異常

### 適用場景
- ✅ 狀態管理錯誤 (Redux, Vuex, State Machine)
- ✅ 資料轉換管道 (ETL, Data Pipeline)
- ✅ 多步驟業務邏輯 (訂單處理, 支付流程)
- ✅ 找不到「哪一步」出錯的問題

---

## Method 2: 影子對比法 (Shadow Execution / Dual Logic)

### 核心思路
**問題本質**: 不確定新代碼是否與預期一致
**解決邏輯**: 讓「舊的穩定代碼」成為對照組

### 概念模型
```
                 Input
                   ↓
        ┌──────────┴──────────┐
        ↓                      ↓
   Legacy Logic          New Logic
   (已知正確)            (待驗證)
        ↓                      ↓
    Result A              Result B
        └──────────┬──────────┘
                   ↓
            A == B ? ✅ : ⚠️
```

### 實作思路 (與語言無關)

**關鍵動作**:
1. **平行執行**: 同時運行舊邏輯和新邏輯
2. **結果比對**: 比較兩者的輸出
3. **差異捕捉**: 不一致時記錄 Input + 兩個 Output
4. **主流程保護**: 影子執行不影響主邏輯,永遠返回 Legacy 結果

### 偽代碼邏輯
```
function processWithShadow(input):
    // 執行可信任的舊邏輯
    legacyResult = executeLegacyLogic(input)

    // 平行執行新邏輯 (不影響主流程)
    try:
        newResult = executeNewLogic(input)

        if legacyResult != newResult:
            logMismatch(input, legacyResult, newResult)
            notifyDevelopers()  // 可選: 即時通知
    catch error:
        logError(error)  // 新邏輯出錯不影響主流程

    // 永遠返回舊邏輯結果確保穩定
    return legacyResult
```

### 語言實作差異對照

| 語言 | 關鍵技術 | 實作方式 |
|------|----------|----------|
| C# | Dynamic Proxies | Castle Core 攔截器 |
| TypeScript | Proxy Object | ES6 Proxy 監控 |
| Python | Decorators | `@shadow_execution` 裝飾器 |
| Java | AOP | Spring AspectJ |
| Go | Interface Wrapping | 自定義 wrapper |

### 價值分析
- **無風險測試**: 使用真實流量測試新代碼,但不影響用戶
- **自動化驗證**: 不需手動編寫測試案例
- **問題前置**: 在問題影響用戶前就發現不一致

### 適用場景
- ✅ 重構核心業務邏輯
- ✅ 替換演算法 (舊 → 新)
- ✅ 驗證 AI 生成的代碼
- ✅ API 版本升級驗證

---

## Method 3: 生成式語意日誌 (Semantic Logging for AI)

### 核心思路
**問題本質**: 傳統 Log 是給人看的,AI 時代的 Log 應該給 AI 分析
**解決邏輯**: 將冷冰冰的數據轉化為「帶完整上下文的語意化記錄」

### 概念對比
```
[傳統 Log - 人類導向]
ERROR: Payment failed
→ 資訊不足,需要翻代碼理解

[語意 Log - AI 導向]
EVENT: Payment processing failed
INTENT: User attempted to purchase premium plan
CONTEXT:
  - User ID: 12345
  - Account Balance: $5.00
  - Purchase Amount: $9.99
  - Payment Method: Visa ending 4242
  - Previous Successful Payment: 2024-12-01
SYSTEM_STATE:
  - API Gateway: Healthy
  - Database Connection: Active
  - Payment Provider: Stripe (responsive)
HYPOTHESIS: Insufficient funds (balance < amount)
```

### 實作思路 (與語言無關)

**關鍵動作**:
1. **擴充 Log 結構**: 不只記錄「發生什麼」,還要記錄:
   - **意圖** (Intent): 操作的目的
   - **上下文** (Context): 所有相關變數的當前值
   - **預期** (Expected): 應該發生什麼
   - **實際** (Actual): 實際發生什麼
   - **系統狀態** (System State): 環境資訊

2. **自然語言描述**: 用人類語言描述技術細節
```
   不好: "var x = null"
   良好: "User object became null after database query, suggesting user not found"
```

3. **結構化輸出**: 使用 JSON 或 Markdown 格式化
   - 便於 AI 解析
   - 包含變數名稱與值的映射

### 語言實作差異對照

| 語言 | 關鍵技術 | 實作方式 |
|------|----------|----------|
| C# | Reflection + Serilog | 自動提取 local variables |
| TypeScript | Error.stack + scope | 捕捉函式作用域變數 |
| Python | `inspect.currentframe()` | 取得 `f_locals` 字典 |
| Java | StackWalker API | 遍歷調用堆疊 |
| Go | runtime.Caller() | 獲取調用者資訊 |

### 價值分析
**AI 分析能力提升**:
```
[簡單 Log] → AI: "看起來是錯誤,但我需要更多資訊"
[語意 Log] → AI: "根據 balance=5.00 和 amount=9.99,
                  這是餘額不足導致的支付失敗,
                  建議提示用戶儲值或選擇較低金額方案"
```

### 適用場景
- ✅ 複雜的多元件交互錯誤
- ✅ 間歇性 Bug (難以重現)
- ✅ 生產環境問題分析
- ✅ 需要 AI 輔助診斷的場景

---

## Method 4: 混沌注入測試 (Chaos Injection / Fault Injection)

### 核心思路
**問題本質**: 真實世界充滿極端情況,但測試時很難想到
**解決邏輯**: 讓「會搞事的虛擬依賴」自動測試各種故障場景

### 概念模型
```
[一般測試]
Your Code → Real API → 一切正常 ✅
→ 但沒測試到極端情況

[混沌測試]
Your Code → Chaos Mock → 隨機故障
                ↓
          ├─ 10% 機率: Timeout
          ├─ 5% 機率: 返回 null
          ├─ 5% 機率: 500 錯誤
          ├─ 10% 機率: 網路延遲 10s
          └─ 70% 機率: 正常執行
```

### 實作思路 (與語言無關)

**關鍵動作**:
1. **建立虛擬依賴**: 為所有外部依賴建立 Mock 版本
   - 資料庫連接
   - HTTP API 呼叫
   - 檔案系統操作
   - 第三方服務

2. **故障劇本設計**: Mock 需要能「演出」各種災難
```
   故障類型:
   - 超時 (Timeout)
   - 空值/錯誤格式 (null/malformed data)
   - 部分成功 (partial failure)
   - 延遲 (latency spike)
   - 連線中斷 (connection drop)
```

3. **可控的隨機性**: 設定故障機率
```
   config = {
     timeout_rate: 0.1,      // 10% 超時
     null_return_rate: 0.05, // 5% 返回 null
     delay_range: [0, 5000]  // 0-5秒隨機延遲
   }
```

4. **替換機制**: 開發/測試環境自動替換真實依賴

### 語言實作差異對照

| 語言 | 關鍵技術 | 實作方式 |
|------|----------|----------|
| C# | Interface + DI | 注入 Chaos 實作 |
| TypeScript | Class/Function Wrapping | Higher-order functions |
| Python | Monkey Patching | `unittest.mock.patch` |
| Java | Mockito | `@MockBean` with custom answers |
| Go | Interface Mocking | testify/mock |

### 價值分析
**發現隱藏的脆弱點**:
```
沒有混沌測試:
- API 一直正常 → 代碼沒有 retry 邏輯
- 上線後真實 API 不穩定 → 系統崩潰

有混沌測試:
- Mock API 隨機超時 → 發現代碼沒處理
- 強迫開發者加入 retry, timeout, fallback
- 上線後系統有韌性
```

### 適用場景
- ✅ 測試錯誤處理是否完善
- ✅ 驗證重試機制
- ✅ 測試降級策略 (fallback)
- ✅ 發現競態條件 (race condition)

---

## Method 5: 性能瓶頸剖析 (Performance Bottleneck Profiling)

### 核心思路
**問題本質**: "慢" 也是一種 Bug,但很難定位在哪一步慢
**解決邏輯**: 測量所有步驟的執行時間,找出最慢的路徑

### 概念模型
```
[黑盒視角]
整個操作耗時 5 秒 → 不知道哪裡慢

[剖析視角]
總計: 5000ms
├─ Step 1: 50ms   ✅ 正常
├─ Step 2: 4800ms ❌ 瓶頸!
└─ Step 3: 150ms  ✅ 正常

→ 直接定位到 Step 2
```

### 實作思路 (與語言無關)

**關鍵動作**:
1. **插入計時點**: 在每個操作前後記錄時間戳
```
   start_time = current_time()
   perform_operation()
   end_time = current_time()
   duration = end_time - start_time
```

2. **階層式測量**: 支援巢狀計時
```
   Main Function (5000ms)
   ├─ Sub-function A (4800ms)
   │  ├─ Database Query (4700ms) ← 真正的瓶頸
   │  └─ Processing (100ms)
   └─ Sub-function B (200ms)
```

3. **閾值警告**: 超過預期時間自動標記
```
   if duration > threshold:
       log_warning("Slow operation detected")
```

4. **報告生成**: 產生排序後的性能報告

### 語言實作差異對照

| 語言 | 計時工具 | 關鍵 API |
|------|----------|----------|
| C# | Stopwatch | `Stopwatch.StartNew()` |
| TypeScript/JS | Performance API | `performance.now()` |
| Python | time module | `time.perf_counter()` |
| Java | System.nanoTime() | `System.nanoTime()` |
| Go | time package | `time.Now()` |

### 價值分析
**從猜測到精確**:
```
[猜測模式]
開發者: "可能是資料庫查詢慢?"
→ 優化資料庫
→ 還是慢
→ "可能是網路請求慢?"
→ 重複 N 次...

[測量模式]
Profiler: "Database query: 4700ms (94%)"
→ 直接優化資料庫查詢
→ 問題解決
```

### 適用場景
- ✅ 回應時間過長
- ✅ 使用者反應「卡頓」
- ✅ API 超時
- ✅ 找不到具體慢在哪裡

---

## 方法選擇指南

根據問題類型選擇對應方法:

| 問題徵兆 | 建議方法 | 原因 |
|---------|---------|------|
| 資料錯誤,但不知道哪一步 | Method 1: 狀態快照 | 需要看完整變化軌跡 |
| 重構後不確定正確性 | Method 2: 影子對比 | 需要與舊版本比對 |
| 間歇性錯誤,難以重現 | Method 3: 語意日誌 | 需要完整上下文分析 |
| 錯誤處理不夠健全 | Method 4: 混沌注入 | 需要測試極端情況 |
| 操作很慢,不知道瓶頸 | Method 5: 性能剖析 | 需要測量每個步驟 |
| 複雜的狀態管理 Bug | Method 1 + 3 | 需要軌跡 + 上下文 |
| API 整合問題 | Method 3 + 4 | 需要日誌 + 故障測試 |

---
## 與 Debug Protocol 的整合

這些方法論對應到 Debug Protocol 的階段:

| Protocol 階段 | 使用方法 |
|--------------|---------|
| Phase 2: Investigation | Method 1, 3, 5 收集資料 |
| Phase 3: Hypothesis | Method 4 測試邊界條件 |
| Phase 4: Fix-Test-Verify | Method 2 驗證正確性 |
| Phase 6: Resolution | 決定哪些 instrumentation 要保留 |

---

## 通用標記慣例

無論使用哪種語言,建議統一使用以下標記:
```
// 註解格式
//debugmode - start
[debug instrumentation code]
//debugmode - end

// 或單行
//debugmode [single line debug code]
```

**清理時搜尋**: `//debugmode` 即可找到所有臨時代碼

**保留判斷**:
- ✅ 保留: 有長期監控價值的日誌
- ✅ 保留: 效能關鍵路徑的測量
- ❌ 移除: 僅用於這次 debug 的詳細追蹤
- ❌ 移除: 影子執行的對比邏輯
