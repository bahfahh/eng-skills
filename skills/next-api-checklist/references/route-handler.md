# Route Handler 開發工作流

> 建立或修改單一 HTTP endpoint 時使用。
> 框架 variant（Supabase、Prisma 等）以（範例）標記。

---

## Guardrails

- 每個 API Route 必須有錯誤處理
- 必須驗證 auth（必要時包含 RBAC / multi-tenant 隔離）
- 回傳格式統一（`{ data }` / `{ error }` envelope）

---

## Steps

Track these as TODOs, complete one by one.

**0. 校準：確認本專案的 API 標準**
- endpoint 位置（`app/api/` 或其他）
- auth 機制（cookie / header / session / JWT）
- 錯誤 helper（是否有 `jsonOk` / `jsonError` 等封裝）
- service layer 位置（`src/services/` / `server/usecases/` / `lib/api/`）

**1. 定義 API 規格**
- HTTP Method（GET / POST / PUT / DELETE）
- Request schema（Zod / Yup / Joi 依專案）
- Response schema / DTO（若專案有）
- Status codes（200 / 201 / 400 / 401 / 403 / 404 / 422 / 500）

**2. 實作 Auth &（若有）Tenant / Role 驗證**

```typescript
// 範例：Next.js + Supabase（示意，依專案調整）
const supabase = createClient()
const { data: { user } } = await supabase.auth.getUser()
if (!user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })

// Multi-tenant（若適用）：tenant ID 必須來自可信來源
const tenantId = getTenantIdFromServerContext(request)
if (!tenantId) return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
```

**3. 實作業務邏輯**
- 呼叫 service layer
- 不在 Route Handler 寫大量 SQL / 交易 / 複雜業務規則

**4. 錯誤處理**
- try-catch 包住所有業務邏輯
- 回傳適當的 status code
- 參考 `error-handling.md`

---

## Verify

- [ ] Auth 驗證正常（401 / 403 按情境正確）
- [ ] Tenant / Role 隔離正常（若適用）
- [ ] 錯誤回傳適當 status code
- [ ] Response 格式統一（`{ data }` / `{ error }`）
- [ ] TypeScript typecheck 無錯誤（若適用）

**驗證方式：**
- 基礎：curl / Postman 驗證主要 status code
- 推薦：unit / integration tests（可重現、可比對）
