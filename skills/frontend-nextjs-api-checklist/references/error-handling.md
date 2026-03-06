# API 錯誤處理與 Status Code 對應

> 統一 API 錯誤格式、status code 映射、前端顯示策略。

---

## Guardrails

- 不把技術錯誤暴露給 client（SQL detail、RLS code、token、stack trace）
- 所有 API 錯誤必須有對應的 status code
- 錯誤訊息要對使用者友善（可理解、可行動）
- 錯誤處理要可測試（unit / integration 可 assert mapping）

---

## Status Code Mapping

| Code | 情境 | 回傳格式 | 前端處理 |
|------|------|---------|---------|
| 200 | 成功（GET/PUT/DELETE） | `{ data: ... }` | 顯示資料 |
| 201 | 成功建立（POST） | `{ data: ... }` | Toast 成功 |
| 400 | 請求格式錯誤 | `{ error: 'message' }` | Toast 錯誤 |
| 401 | 未登入 / session 失效 | `{ error: 'Unauthorized' }` | 導回登入 |
| 403 | 權限不足 | `{ error: 'Forbidden' }` | Toast + 回上一頁 |
| 404 | 資源不存在 | `{ error: 'Not found' }` | Toast + 回列表 |
| 409 | 衝突（如併發） | `{ error: 'Conflict' }` | Toast + 重新整理 |
| 422 | 驗證錯誤 | `{ error: 'message', fields: {...} }` | Inline error |
| 429 | 太頻繁 | `{ error: 'Too many requests' }` | Toast + 延遲重試 |
| 500 | 伺服器錯誤 | `{ error: 'Internal error' }` | Toast + 重試 |

---

## Steps

**0. 校準：確認本專案的 response envelope**
- 成功格式（`{ data }` 或直接回 resource）
- 失敗格式（`{ error }` / `{ error, code, fields }`）
- 需要支援的錯誤種類：validation / auth / permission / not-found / conflict / rate-limit / unexpected

**1. 建立統一 Response Helper（或沿用既有）**

```typescript
// 範例：Next.js Route Handler（示意）
// lib/utils/api-response.ts
export function apiError(message: string, status: number) {
  return NextResponse.json({ error: message }, { status })
}

export function apiSuccess(data: unknown, status = 200) {
  return NextResponse.json({ data }, { status })
}
```

**2. Route Handler 使用**

```typescript
// 把「技術錯誤」轉成「語意錯誤」再映射到 status code
try {
  const result = await someService.create(data)
  return apiSuccess(result, 201)
} catch (error) {
  // 範例：資料層錯誤碼 → 語意錯誤
  if (error.code === 'PGRST116') return apiError('Resource not found', 404)
  logger.error('Unexpected error', { error })
  return apiError('Internal error', 500)
}
```

**3. 前端統一處理**
- 把 status code 映射到一致的 UI 行為（toast / inline / dialog / redirect）

---

## Verify

- Unit / integration：覆蓋主要 mapping（422 / 401 / 403 / 404 / 409 / 500）
- UI：測試每個 status code 的顯示策略
- 確認技術錯誤不會出現在 response body
