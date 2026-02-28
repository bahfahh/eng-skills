# Stryker 快速設定 — EduNote 實施

---

## 1. 安裝

```bash
pnpm add -D @stryker-mutator/core @stryker-mutator/jest-runner
```

---

## 2. 設定檔 `stryker.conf.json`

```json
{
  "testRunner": "jest",
  "testFramework": "jest",
  "reporters": ["html", "json", "clear-text"],
  "reporterOptions": {
    "htmlReporter": {
      "baseDir": "reports/mutation"
    }
  },
  "mutate": [
    "lib/api/**/*.ts",
    "hooks/**/*.ts",
    "lib/stores/**/*.ts",
    "!**/*.test.ts",
    "!**/*.spec.ts"
  ],
  "mutator": {
    "excludedMutations": [
      "StringLiteral",
      "LogicalOperator"
    ]
  },
  "thresholds": {
    "high": 80,
    "medium": 70,
    "low": 60
  },
  "timeoutMS": 5000,
  "timeoutFactor": 1.25,
  "concurrency": 4,
  "maxTestRunnerReuse": 1
}
```

**關鍵設定說明：**
- `mutate`: 只檢查這些文件（不檢查測試本身）
- `excludedMutations`: 跳過無意義的 mutation（字符串字面值）
- `thresholds`: 如果 mutation score 低於 60%，CI 失敗
- `concurrency`: 並行執行，加快速度

---

## 3. 執行

```bash
# 執行 mutation testing
pnpm stryker run

# 查看 HTML 報告
open reports/mutation/index.html

# 只檢查特定文件
pnpm stryker run --mutate "lib/api/observations.ts"
```

---

## 4. 理解報告

### HTML 報告結構

```
reports/mutation/
├── index.html              # 總覽
├── mutation-test-elements.js
└── [file-name].html        # 每個文件的詳細報告
```

### 報告中的顏色

- 🟢 **綠色** — Killed mutation（測試捕捉到了）✅ 好
- 🔴 **紅色** — Survived mutation（測試沒捕捉到）❌ 壞
- 🟡 **黃色** — Timeout（mutation 導致無限迴圈）⚠️ 需要檢查
- ⚫ **灰色** — Not covered（沒有測試覆蓋）⚪ 需要寫測試

### 例子

```
Line 45: if (obs.created_by !== userId) {
         ↑ 這行有 3 個 mutation
         
Mutation 1: !== 改為 ===
Result: Killed ✅ 測試有驗證這個檢查

Mutation 2: 移除整個 if 塊
Result: Survived ❌ 測試沒有驗證這個檢查

Mutation 3: 改為 ==
Result: Killed ✅ 測試有驗證
```

---

## 5. 常見 Mutation 類型

| Mutation | 例子 | 意義 |
|----------|------|------|
| Conditional Boundary | `>` → `>=` | 邊界條件 |
| Logical Operator | `&&` → `\|\|` | 邏輯錯誤 |
| Arithmetic Operator | `+` → `-` | 計算錯誤 |
| Assignment | `=` → `+=` | 賦值錯誤 |
| Return Value | `return true` → `return false` | 返回值錯誤 |
| Block Statement | 移除整個 if/for | 控制流錯誤 |

---

## 6. EduNote 優先檢查的文件

### 高優先級（安全相關）

```bash
# 多租戶隔離
pnpm stryker run --mutate "lib/api/observations.ts"

# 權限檢查
pnpm stryker run --mutate "lib/api/auth.ts"

# 資料查詢
pnpm stryker run --mutate "hooks/useObservations.ts"
```

### 中優先級（業務邏輯）

```bash
# 報告生成
pnpm stryker run --mutate "lib/api/reports.ts"

# 標籤管理
pnpm stryker run --mutate "lib/api/tags.ts"
```

---

## 7. 改進 Survived Mutation

### 場景：測試沒有驗證權限檢查

```typescript
// 程式碼
async function deleteObservation(obsId: string, userId: string) {
  const obs = await db.from('observations').select('*').eq('id', obsId).single();
  
  if (obs.created_by !== userId) {
    throw new Error('Permission denied');
  }
  
  await db.from('observations').delete().eq('id', obsId);
}

// 原始測試（不夠好）
it('should delete observation', async () => {
  const obs = await createObservation({ created_by: 'user-1' });
  await deleteObservation(obs.id, 'user-1');  // ← 只測試成功路徑
  const result = await db.from('observations').select('*').eq('id', obs.id);
  expect(result).toHaveLength(0);
});

// Stryker 報告：Mutation "移除 if 檢查" Survived ❌
// 原因：測試沒有驗證失敗路徑

// 改進後的測試
it('should delete observation if creator', async () => {
  const obs = await createObservation({ created_by: 'user-1' });
  await deleteObservation(obs.id, 'user-1');
  const result = await db.from('observations').select('*').eq('id', obs.id);
  expect(result).toHaveLength(0);
});

it('should reject deletion if not creator', async () => {
  const obs = await createObservation({ created_by: 'user-1' });
  await expect(deleteObservation(obs.id, 'user-2')).rejects.toThrow('Permission denied');
});

// 現在 Stryker 會 kill 這個 mutation ✅
```

---

## 8. CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/mutation-testing.yml
name: Mutation Testing

on:
  pull_request:
  schedule:
    - cron: '0 2 * * 0'  # 每週日凌晨 2 點

jobs:
  mutation-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run Stryker
        run: pnpm stryker run
      
      - name: Check mutation score
        run: |
          SCORE=$(cat reports/mutation/metrics.json | jq '.score')
          echo "Mutation score: $SCORE%"
          if (( $(echo "$SCORE < 70" | bc -l) )); then
            echo "❌ Mutation score too low: $SCORE%"
            exit 1
          fi
          echo "✅ Mutation score acceptable: $SCORE%"
      
      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: mutation-report
          path: reports/mutation/
```

---

## 9. 性能優化

### 問題：Stryker 很慢

```bash
# 原始執行時間：5–10 分鐘
pnpm stryker run

# 優化 1：只檢查改變的文件
pnpm stryker run --mutate "lib/api/observations.ts"

# 優化 2：增加並行度
# 在 stryker.conf.json 中設定 "concurrency": 8

# 優化 3：跳過某些 mutation
# 在 stryker.conf.json 中設定 excludedMutations
```

### 建議配置

```json
{
  "concurrency": 4,
  "timeoutMS": 5000,
  "timeoutFactor": 1.25,
  "maxTestRunnerReuse": 1,
  "mutator": {
    "excludedMutations": [
      "StringLiteral",
      "LogicalOperator",
      "ConditionalExpression"
    ]
  }
}
```

---

## 10. 常見問題

### Q: Mutation score 很低，怎麼辦？

A: 不是所有 mutation 都需要 kill。目標是 70–80%。

- 如果 score < 60%：測試品質有問題，需要改進
- 如果 60–70%：可以接受，但有改進空間
- 如果 70–80%：好
- 如果 > 80%：很好，但可能過度測試

### Q: 某個 mutation 無法 kill，怎麼辦？

A: 可能是：
1. 測試無法覆蓋（例如：外部 API 呼叫）
2. Mutation 無意義（例如：改變常數）
3. 程式碼有冗餘邏輯

檢查報告，決定是否需要改進測試或程式碼。

### Q: Stryker 執行超時，怎麼辦？

A: 增加 `timeoutMS` 或 `timeoutFactor`：

```json
{
  "timeoutMS": 10000,
  "timeoutFactor": 1.5
}
```

---

## 11. 分層執行策略

Stryker 慢（5–15 分鐘），不適合高頻執行。AI coding 驗證最需要的是快速回饋，不是完整的測試品質分析。

**原則：快速回饋 > 完整覆蓋**

```
AI coding 驗證（每次改動）
└── pnpm test --testPathPattern="<changed-file>"
    目標：< 30 秒
    只跑改動的文件

PR / commit（每次）
└── pnpm test
    目標：< 2 分鐘
    完整測試套件

定期健康檢查（每週 / 每次 release）
└── pnpm stryker run
    目標：不卡開發流程
    跑在 CI 排程，不 block PR
```

**AI coding 驗證用這個：**
```bash
# 只跑改動的文件，秒級回饋
pnpm test --testPathPattern="observations" --passWithNoTests
```

**Stryker 放排程，不放 PR gate：**
```yaml
# .github/workflows/mutation-testing.yml
on:
  schedule:
    - cron: '0 2 * * 0'  # 每週日凌晨，不影響開發
```

Stryker 的價值是定期健康檢查，不是即時驗證。就像健康檢查不需要每天做，但每季做一次很有價值。
