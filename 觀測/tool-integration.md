# 工具搭配方案 — 強化 Semantic Review 的觀測與驗證

Semantic review 不是孤立的。它需要搭配工具來：
1. **驗證** API 行為是否符合 spec
2. **觀測** 程式碼在真實環境的表現
3. **測試** 邊界條件和隱性假設
4. **追蹤** AI agent 的推理過程

---

## 層級架構

```
┌─────────────────────────────────────────────────────────┐
│ Semantic Review (AI-First + Semantic LLM)               │
│ ↓ 發現問題 ↓                                             │
├─────────────────────────────────────────────────────────┤
│ 驗證層 (Contract Testing + API Testing)                 │
│ ↓ 確認 API 符合 spec ↓                                   │
├─────────────────────────────────────────────────────────┤
│ 觀測層 (Observability + Tracing)                        │
│ ↓ 看到程式碼實際在做什麼 ↓                               │
├─────────────────────────────────────────────────────────┤
│ 測試層 (Mutation Testing + Property Testing)            │
│ ↓ 驗證測試本身的品質 ↓                                   │
└─────────────────────────────────────────────────────────┘
```

---

## 1. 驗證層 — Contract Testing + API Testing

### 問題
Semantic review 說「API 應該返回 {student, inviteCode}」，但實際上呢？

### 工具選擇

**A. Contract Testing（推薦）**

| 工具 | 特點 | 適合場景 |
|------|------|--------|
| **Pact** | Consumer-driven contracts，JavaScript/TypeScript 支援好 | 微服務、多層架構 |
| **Spring Cloud Contract** | 基於 Groovy DSL，Java 生態 | 後端服務間契約 |
| **Specmatic** | OpenAPI 原生支援，自動生成測試 | API-first 開發 |
| **Karate DSL** | 簡單 Gherkin 語法，快速寫測試 | 快速驗證 API 行為 |

**EduNote 推薦：Pact**
- TypeScript 支援完整
- 可以驗證 hook → API → database 的整個鏈路
- 自動生成 pact 文件作為 spec 的補充

**使用方式：**
```typescript
// tests/pacts/observations.pact.ts
import { PactV3 } from '@pact-foundation/pact';

const pact = new PactV3({
  consumer: 'TeacherApp',
  provider: 'ObservationAPI',
});

pact.addInteraction({
  states: [{ description: 'observation exists' }],
  uponReceiving: 'a request to get observation',
  withRequest: {
    method: 'GET',
    path: '/api/observations/123',
  },
  willRespondWith: {
    status: 200,
    body: {
      id: '123',
      text: 'observation text',
      tags: ['情緒', '社會'],
      created_by: 'teacher-1',
      organization_id: 'org-1', // ← 驗證多租戶隔離
    },
  },
});
```

**B. API Testing（補充）**

| 工具 | 特點 | 適合場景 |
|------|------|--------|
| **Bruno** | Postman 替代品，輕量、開源 | 快速手動測試 |
| **Insomnia** | 類似 Postman，支援 GraphQL | 開發時探索 API |
| **dotMock** | AI 驅動的 API mock，快速生成 | 前端開發不等後端 |
| **Karate** | 自動化 API 測試，支援 CI/CD | 集成測試 |

**EduNote 推薦：Bruno + Karate**
- Bruno：開發時快速驗證 API（Postman 的開源替代品）
- Karate：CI/CD 中自動化驗證 API 契約

---

## 2. 觀測層 — Tracing + Logging

### 問題
Semantic review 說「這裡有隱性假設」，但我看不到程式碼在真實環境的執行流程。

### 工具選擇

**A. 應用層 Observability（Application Observability）**

| 工具 | 特點 | 適合場景 |
|------|------|--------|
| **Langfuse** | 開源 LLM tracing，支援自定義 span | AI agent 推理過程追蹤 |
| **PostHog** | 開源分析 + 錄影，看用戶行為 | 前端行為觀測 |
| **Datadog** | 商用，完整可觀測性 | 生產環境監控 |
| **New Relic** | 商用，APM + 日誌 | 性能監控 |

**EduNote 推薦：Langfuse（開源）+ PostHog（開源）**

Langfuse 用途：
- 追蹤 semantic review agent 的推理過程
- 記錄每個維度（D1–D4）的評估步驟
- 看到 AI 是怎麼得出結論的

```typescript
// lib/observability/langfuse.ts
import { Langfuse } from 'langfuse';

const langfuse = new Langfuse({
  publicKey: process.env.LANGFUSE_PUBLIC_KEY,
  secretKey: process.env.LANGFUSE_SECRET_KEY,
  baseUrl: process.env.LANGFUSE_BASE_URL,
});

// 在 semantic review 中追蹤
export async function reviewWithTracing(diff: string, context: string) {
  const trace = langfuse.trace({
    name: 'semantic-review',
    input: { diff, context },
  });

  // D1: Intent Alignment
  const d1Span = trace.span({
    name: 'dimension_1_intent_alignment',
  });
  // ... 評估邏輯
  d1Span.end({ output: d1Result });

  // D2: Boundary Reasoning
  const d2Span = trace.span({
    name: 'dimension_2_boundary_reasoning',
  });
  // ... 評估邏輯
  d2Span.end({ output: d2Result });

  trace.end({ output: finalVerdict });
  return finalVerdict;
}
```

PostHog 用途：
- 看教師端上傳觀察時的實際流程
- 追蹤 offline sync 的行為
- 發現前端邊界條件（網路中斷、session 過期等）

**B. 代碼層 Observability（Code Metrics）**

已在 `觀測.md` 中涵蓋：
- ESLint complexity rules
- Roslyn analyzers
- SonarCloud

---

## 3. 測試層 — Mutation Testing + Property Testing

### 問題
Semantic review 說「測試涵蓋了這個邊界」，但測試本身的品質如何？

### 工具選擇

**A. Mutation Testing（推薦）**

| 工具 | 特點 | 適合場景 |
|------|------|--------|
| **Stryker** | TypeScript/JavaScript，快速，支援 Jest | EduNote 首選 |
| **PIT** | Java，成熟穩定 | 後端 C# 可用 Stryker.NET |
| **Mutant** | Python | 不適用 |

**Stryker 用途：**
- 在測試中注入 mutation（改變程式碼），看測試能否捕捉
- 發現「假陽性」測試（看起來通過，但沒真的驗證邏輯）
- 計算 mutation score（測試品質指標）

**EduNote 推薦配置：**
```json
// stryker.conf.json
{
  "testRunner": "jest",
  "testFramework": "jest",
  "reporters": ["html", "json"],
  "mutate": [
    "lib/api/**/*.ts",
    "hooks/**/*.ts",
    "!**/*.test.ts"
  ],
  "mutator": {
    "excludedMutations": ["StringLiteral"]
  },
  "thresholds": {
    "high": 80,
    "medium": 70,
    "low": 60
  }
}
```

**使用方式：**
```bash
# 執行 mutation testing
pnpm stryker run

# 生成 HTML 報告
open reports/mutation/index.html
```

**B. Property-Based Testing（補充）**

| 工具 | 特點 | 適合場景 |
|------|------|--------|
| **fast-check** | JavaScript，生成隨機測試用例 | 邊界條件探索 |
| **Hypothesis** | Python | 不適用 |
| **QuickCheck** | Haskell | 不適用 |

**fast-check 用途：**
- 自動生成邊界測試用例（null、empty、overflow）
- 發現手寫測試遺漏的邊界

```typescript
// tests/lib/api/observations.property.test.ts
import fc from 'fast-check';

describe('createObservation - property tests', () => {
  it('should handle any valid student ID', () => {
    fc.assert(
      fc.property(fc.uuid(), (studentId) => {
        const result = createObservation({
          student_id: studentId,
          text: 'test',
          organization_id: 'org-1',
        });
        expect(result.student_id).toBe(studentId);
      })
    );
  });

  it('should reject null text', () => {
    fc.assert(
      fc.property(fc.anything(), (invalidText) => {
        if (invalidText === null || invalidText === undefined) {
          expect(() =>
            createObservation({
              student_id: 'id',
              text: invalidText,
              organization_id: 'org-1',
            })
          ).toThrow();
        }
      })
    );
  });
});
```

---

## 4. 整合方案 — 完整工具鏈

### 開發流程

```
1. 開發者寫程式碼
   ↓
2. Semantic Review (AI-First + Semantic LLM)
   ↓ 發現問題 ↓
3. 修改程式碼
   ↓
4. Contract Testing (Pact)
   ↓ 驗證 API 符合 spec ↓
5. API Testing (Bruno/Karate)
   ↓ 手動/自動驗證 ↓
6. Unit Tests + Property Tests (Jest + fast-check)
   ↓ 寫測試 ↓
7. Mutation Testing (Stryker)
   ↓ 驗證測試品質 ↓
8. Observability (Langfuse + PostHog)
   ↓ 監控生產環境 ↓
9. 提交 PR
```

### CI/CD 集成

```yaml
# .github/workflows/review.yml
name: Code Review Pipeline

on: [pull_request]

jobs:
  semantic-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Semantic Review
        run: pnpm review:semantic

  contract-testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Pact Tests
        run: pnpm test:pact
      - name: Publish Pacts
        run: pnpm pact:publish

  mutation-testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Stryker
        run: pnpm stryker run
      - name: Check Mutation Score
        run: |
          SCORE=$(cat reports/mutation/metrics.json | jq '.score')
          if (( $(echo "$SCORE < 70" | bc -l) )); then
            echo "Mutation score too low: $SCORE"
            exit 1
          fi

  api-testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Karate Tests
        run: pnpm test:api
```

---

## 5. 工具成本對比

| 工具 | 成本 | 推薦度 |
|------|------|--------|
| Pact | 開源 | ⭐⭐⭐⭐⭐ |
| Bruno | 開源 | ⭐⭐⭐⭐ |
| Karate | 開源 | ⭐⭐⭐⭐ |
| Langfuse | 開源 + 商用 | ⭐⭐⭐⭐⭐ |
| PostHog | 開源 + 商用 | ⭐⭐⭐⭐ |
| Stryker | 開源 | ⭐⭐⭐⭐⭐ |
| fast-check | 開源 | ⭐⭐⭐⭐ |
| Postman | 商用 | ⭐⭐⭐ (Bruno 替代) |
| Datadog | 商用 | ⭐⭐⭐ (生產環境) |

---

## 6. EduNote 實施優先級

### Phase 1（立即）
- ✅ Semantic Review（已完成）
- ✅ ESLint + Roslyn metrics（已完成）
- 🔲 Stryker mutation testing（快速贏）

### Phase 2（1–2 週）
- 🔲 Pact contract testing
- 🔲 Bruno for API exploration
- 🔲 Langfuse for agent tracing

### Phase 3（1 個月）
- 🔲 Karate for CI/CD API testing
- 🔲 PostHog for production observability
- 🔲 fast-check for property testing

### Phase 4（可選）
- 🔲 Datadog for production monitoring
- 🔲 Advanced mutation testing strategies
