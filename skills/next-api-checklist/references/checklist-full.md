# Next.js API 完整開發清單

> 涵蓋所有 API 開發階段。逐項勾選，完成後進行驗收。
> 框架特定作法（Supabase、Prisma 等）標記為（範例）— 依專案調整。

---

## Guardrails

- API Route 必須有錯誤處理與適當的 status code
- 必須驗證 auth（必要時包含租戶隔離 / RBAC）
- 回傳格式統一：`{ data }` / `{ error }`
- 業務邏輯放在 Service Layer，不在 Route Handler 直接寫資料存取細節
- 只記錄必要 log（避免把 token/PII/內部錯誤細節回傳給 client）

---

## Phase 1：Route Handler 結構

### 1. Auth 驗證
- [ ] 使用 server-side auth 驗證（cookie / header / session / JWT 依專案而定）
- [ ] 優先用「快速失敗」的方式驗證（避免昂貴或容易卡住的 call）
  - （範例：Supabase + Next.js 用 `getUser()`，避免初次載入的 `getSession()` hang）
- [ ] 未認證回傳 401

### 2. 角色與租戶驗證
- [ ] 若有 RBAC：在 server 驗證 role/permission
- [ ] 若是 multi-tenant：tenant/org ID 從 server-side 可信來源取得
- [ ] 驗證帳號狀態（active/disabled/pending）（範例）
- [ ] 權限不足回傳 403

### 3. Request 驗證
- [ ] 使用 schema 驗證 request（Zod/Yup/Joi 依專案）（範例）
- [ ] 驗證必填欄位與資料型別
- [ ] 驗證失敗回傳 422 + 欄位錯誤

### 4. HTTP Method 定義
- [ ] GET：查詢資料
- [ ] POST：建立資料（成功回傳 201）
- [ ] PUT/PATCH：更新資料
- [ ] DELETE：刪除資料
- [ ] 只導出需要的 method

---

## Phase 2：Service Layer

### 5. Service 結構
- [ ] 業務邏輯封裝在 service module（例如 `src/services/` / `lib/api/` / `server/usecases/`）
- [ ] Route Handler 只負責：解析輸入 → 驗證 → 呼叫 service → 格式化回傳
- [ ] 不在 Route Handler 寫大量 SQL / 交易 / 複雜業務規則

### 6. 資料層查詢
- [ ] 使用一致的 abstraction（Supabase / ORM / Query Builder 依專案）
- [ ] 保持 type-safe（DB types / Prisma types / DTO）（範例）
- [ ] 若是 multi-tenant：查詢包含 tenant 過濾（或由 DB policy/RLS 防線保護）
- [ ] 錯誤能被上層捕捉並映射成正確 status code
- [ ] 回傳欄位明確（避免回傳敏感欄位）

### 7. 資料操作
- [ ] Create：回傳新建資料
- [ ] Update：回傳更新後資料
- [ ] Delete：依策略（軟刪除/硬刪除）一致
- [ ] 批次操作有 transaction 保護
- [ ] 併發操作有衝突處理

---

## Phase 3：錯誤處理

### 8. Status Code 對應
- [ ] 200：成功（GET/PUT/DELETE）
- [ ] 201：成功建立（POST）
- [ ] 400：請求格式錯誤
- [ ] 401：未登入 / session 失效
- [ ] 403：權限不足
- [ ] 404：資源不存在
- [ ] 409：併發衝突
- [ ] 422：驗證錯誤
- [ ] 429：太頻繁
- [ ] 500：伺服器錯誤

詳細對應與 response envelope → `error-handling.md`

### 9. 錯誤回傳格式
- [ ] 錯誤格式：`{ error: 'message' }`
- [ ] 成功格式：`{ data: ... }`
- [ ] 驗證錯誤包含欄位資訊：`{ error: 'message', fields: {...} }`

### 10. 錯誤安全性
- [ ] 不暴露技術錯誤（SQL、RLS code、token、stack trace）
- [ ] 轉譯成使用者可理解的訊息
- [ ] try-catch 包住所有業務邏輯
- [ ] 未預期錯誤回傳 500 + 通用訊息 + 記錄到 logger

---

## Phase 4：Client Cache / Data Fetching 整合

### 11. Query Keys（若使用 React Query / SWR）
- [ ] 有一致的 key 策略（集中或約定式）
- [ ] 使用階層式 key 結構，篩選條件納入 key

### 12. Query Hook
- [ ] 使用資料抓取 hook 包裝 GET（例如 `useQuery`）
- [ ] 設定合理的 cache/stale 策略（staleTime / revalidate）
- [ ] 設定 `enabled` 條件（避免缺參數時送無效請求）
- [ ] 錯誤策略一致（例如：不 retry 404 / 422）

### 13. Mutation Hook
- [ ] 使用 mutation hook 包裝寫入（例如 `useMutation`）
- [ ] 成功後更新 cache（invalidate / refetch / setQueryData 依策略）
- [ ] 需要時支援 optimistic update + rollback
- [ ] 回傳 mutation 狀態給 UI（pending / error / success）

---

## Phase 5：安全性

### 14. 認證安全
- [ ] 所有受保護 API 都有 auth 檢查
- [ ] 在 server 做權限決策，不信任前端宣稱的身份/權限
- [ ] 不信任前端傳來的 user ID
- [ ] Session 過期正確處理

### 15. 租戶隔離（若是 multi-tenant）
- [ ] 所有查詢都過濾 tenant/org ID
- [ ] tenant/org ID 從可信來源取得（server-side profile/session/claims）
- [ ] 跨租戶存取被阻擋（403 或空結果）
- [ ] DB policy/RLS 作為最後防線（若專案有）

### 16. 輸入驗證
- [ ] 所有外部輸入都經過驗證
- [ ] 防止 SQL injection（使用參數化查詢 / ORM / query builder）
- [ ] 防止 XSS（不直接回傳使用者輸入）
- [ ] 檔案上傳有大小和類型限制

---

## Phase 6：驗收

### 17. Dev Server
- [ ] `pnpm dev`（或專案對應指令）啟動無錯誤
- [ ] API Route 可正常存取

### 18. 基礎探針（curl）

```bash
# 401 - 未認證
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/api/YOUR_ROUTE

# 403 - 權限不足
curl -s -w "%{http_code}" -H "Authorization: Bearer <wrong_role_token>" http://localhost:3000/api/YOUR_ROUTE

# 422 - 驗證錯誤
curl -s -w "%{http_code}" -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer <valid_token>" \
  -d '{"invalid": "data"}' \
  http://localhost:3000/api/YOUR_ROUTE

# 201 - 建立成功
curl -s -w "%{http_code}" -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer <valid_token>" \
  -d '{"valid": "data"}' \
  http://localhost:3000/api/YOUR_ROUTE
```

- [ ] 401 → `{ "error": "Unauthorized" }`
- [ ] 403 → `{ "error": "Forbidden" }`
- [ ] 422 → `{ "error": "...", "fields": {...} }`
- [ ] 404 → `{ "error": "Not found" }`
- [ ] 200 → `{ "data": ... }`
- [ ] 201 → `{ "data": ... }`
- [ ] 500 → `{ "error": "Internal error" }`（不含技術細節）

### 19. 自動化測試（推薦）

- [ ] Unit tests：驗證 input validation、status mapping、domain logic
- [ ] Integration tests：驗證端到端行為（auth / permissions / tenant）
- [ ] 每個測試有明確 assertion（status、schema、敏感欄位不外洩）

### 20. TypeScript 檢查（若適用）
- [ ] `pnpm tsc --noEmit` 無錯誤
- [ ] Request / Response DTO 型別完整
- [ ] Service layer 型別完整

---

## 常見陷阱

| 錯誤 | 正確 |
|------|------|
| 在 Route Handler 寫 SQL | 封裝在 service，Route Handler 只協調流程 |
| 從 request body 取 tenant ID | 從 server-side profile / session / claims 取得 |
| 暴露技術錯誤 `return jsonError(error.message, 500)` | `return jsonError('系統暫時忙碌', 500)` + logger |
| 忘記 auth 檢查直接處理業務邏輯 | 先 getUser() → 檢查 profile → 再處理 |
| `console.log('error:', error)` | `logger.error('Failed to create', { error })` |
| POST 成功回傳 200 | POST 成功回傳 201 |
