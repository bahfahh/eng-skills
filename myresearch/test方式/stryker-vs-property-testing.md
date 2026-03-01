# Stryker vs Property-Based Testing — 本質差異

你現在有 property-based test（fast-check）。Stryker 是完全不同的東西。

---

## 核心差異

### Property-Based Testing（你現在有的）
**問題：** 我的測試涵蓋了所有邊界嗎？
**方法：** 自動生成隨機輸入，看程式碼是否通過

```typescript
// fast-check: 生成隨機輸入
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

// 執行：生成 100 個隨機 UUID，都通過 ✅
```

**測試的是：** 程式碼的行為

---

### Mutation Testing（Stryker）
**問題：** 我的測試本身有沒有問題？測試是不是真的在驗證邏輯？
**方法：** 改變程式碼（注入 mutation），看測試能否捕捉

```typescript
// 原始程式碼
function createObservation(data) {
  if (!data.student_id) throw new Error('Missing student_id');
  return { ...data, created_at: new Date() };
}

// Stryker 注入 mutation 1：移除檢查
function createObservation(data) {
  // if (!data.student_id) throw new Error('Missing student_id');  ← 刪除
  return { ...data, created_at: new Date() };
}
// 執行測試 → 測試還通過嗎？
// 如果通過 = 測試沒有驗證這個檢查 ❌ 壞測試
// 如果失敗 = 測試有驗證 ✅ 好測試

// Stryker 注入 mutation 2：改變邏輯
function createObservation(data) {
  if (data.student_id) throw new Error('Missing student_id');  // ← 反轉邏輯
  return { ...data, created_at: new Date() };
}
// 執行測試 → 測試還通過嗎？
```

**測試的是：** 測試本身的品質

---

## 視覺對比

```
Property-Based Testing (fast-check)
┌─────────────────────────────────────┐
│ 程式碼                               │
│ function add(a, b) { return a + b; }│
└────────────┬────────────────────────┘
             │ 生成隨機輸入
             ↓
┌─────────────────────────────────────┐
│ 測試                                 │
│ expect(add(1, 2)).toBe(3)           │
│ expect(add(-1, 2)).toBe(1)          │
│ expect(add(999, 999)).toBe(1998)    │
│ ... (100 個隨機組合)                 │
└────────────┬────────────────────────┘
             │ 都通過？
             ↓
        ✅ 程式碼正確


Mutation Testing (Stryker)
┌─────────────────────────────────────┐
│ 程式碼                               │
│ function add(a, b) { return a + b; }│
└────────────┬────────────────────────┘
             │ 注入 mutation
             ↓
┌─────────────────────────────────────┐
│ 變異程式碼 1                          │
│ function add(a, b) { return a - b; }│ ← 改 + 為 -
└────────────┬────────────────────────┘
             │ 執行測試
             ↓
        ❌ 測試失敗 ✅ 好測試
        
┌─────────────────────────────────────┐
│ 變異程式碼 2                          │
│ function add(a, b) { return a; }    │ ← 移除 + b
└────────────┬────────────────────────┘
             │ 執行測試
             ↓
        ❌ 測試失敗 ✅ 好測試
```

---

## 實際例子：EduNote 觀察記錄

### 場景：驗證多租戶隔離

```typescript
// 程式碼
async function getObservations(userId: string, orgId: string) {
  return db
    .from('observations')
    .select('*')
    .eq('organization_id', orgId)  // ← 多租戶檢查
    .eq('created_by', userId);
}

// 你寫的測試
it('should only return observations from the same org', async () => {
  const obs1 = await createObservation({ org_id: 'org-1', created_by: 'user-1' });
  const obs2 = await createObservation({ org_id: 'org-2', created_by: 'user-1' });
  
  const result = await getObservations('user-1', 'org-1');
  expect(result).toHaveLength(1);
  expect(result[0].id).toBe(obs1.id);
});
```

### Property-Based Test 會做什麼

```typescript
// fast-check：生成隨機 org_id 和 user_id
it('should handle any org/user combination', () => {
  fc.assert(
    fc.property(
      fc.uuid(),  // orgId
      fc.uuid(),  // userId
      async (orgId, userId) => {
        const obs = await createObservation({ org_id: orgId, created_by: userId });
        const result = await getObservations(userId, orgId);
        expect(result).toContainEqual(obs);
      }
    )
  );
});

// 執行：生成 100 個隨機 org/user 組合
// 都通過 ✅ 程式碼在各種輸入下都正確
```

**發現的問題：** 程式碼在邊界情況下是否正確（null org、特殊字符等）

---

### Mutation Testing 會做什麼

```typescript
// Stryker 注入 mutation 1：移除 org_id 檢查
async function getObservations(userId: string, orgId: string) {
  return db
    .from('observations')
    .select('*')
    // .eq('organization_id', orgId)  ← 刪除
    .eq('created_by', userId);
}

// 執行你的測試 → 測試還通過嗎？
// ❌ 測試失敗 ✅ 好測試（測試有驗證 org_id 檢查）

// Stryker 注入 mutation 2：改變邏輯
async function getObservations(userId: string, orgId: string) {
  return db
    .from('observations')
    .select('*')
    .eq('organization_id', 'org-999')  // ← 硬編碼錯誤的 org
    .eq('created_by', userId);
}

// 執行你的測試 → 測試還通過嗎？
// ❌ 測試失敗 ✅ 好測試
```

**發現的問題：** 你的測試是不是真的在驗證 org_id 檢查，還是只是碰巧通過

---

## 關鍵差異表

| 面向 | Property-Based Test | Mutation Testing |
|------|-------------------|-----------------|
| **測試什麼** | 程式碼的行為 | 測試本身的品質 |
| **怎麼做** | 生成隨機輸入 | 改變程式碼 |
| **發現的問題** | 邊界條件、輸入驗證 | 假陽性測試、測試覆蓋不足 |
| **輸出** | 測試通過/失敗 | Mutation score（0–100%） |
| **執行時間** | 快（100 個輸入） | 慢（每個 mutation 跑一次測試） |
| **何時執行** | 每次測試 | 定期（CI/CD 或 pre-commit） |

---

## 為什麼兩個都需要？

### 只有 Property-Based Test

```typescript
// 你的測試
it('should handle any org/user', () => {
  fc.assert(
    fc.property(fc.uuid(), fc.uuid(), async (orgId, userId) => {
      const obs = await createObservation({ org_id: orgId, created_by: userId });
      const result = await getObservations(userId, orgId);
      expect(result).toContainEqual(obs);  // ← 這個 expect 真的在驗證嗎？
    })
  );
});

// 問題：測試通過了，但你不知道是因為：
// A. 程式碼正確
// B. 測試沒有真的驗證邏輯（假陽性）
```

### 加上 Mutation Testing

```
Stryker 注入 mutation：移除 org_id 檢查
↓
執行 property-based test
↓
測試還通過？ ← 這表示你的 expect 沒有驗證 org_id 檢查
↓
Mutation score 下降 ⚠️ 警告：測試品質有問題
```

---

## EduNote 實際場景

### 場景 1：觀察記錄的權限檢查

```typescript
// 程式碼
async function deleteObservation(obsId: string, userId: string, orgId: string) {
  const obs = await db
    .from('observations')
    .select('*')
    .eq('id', obsId)
    .eq('organization_id', orgId)
    .single();

  if (obs.created_by !== userId) {
    throw new Error('Permission denied');  // ← 只有創建者能刪除
  }

  await db.from('observations').delete().eq('id', obsId);
}

// 你的測試
it('should only allow creator to delete', async () => {
  const obs = await createObservation({ created_by: 'user-1' });
  
  // 創建者可以刪除
  await expect(deleteObservation(obs.id, 'user-1', 'org-1')).resolves.not.toThrow();
  
  // 其他人不能刪除
  await expect(deleteObservation(obs.id, 'user-2', 'org-1')).rejects.toThrow();
});
```

**Property-Based Test 會做什麼：**
```typescript
// 生成隨機 userId，驗證邏輯在各種 user 下都正確
fc.assert(
  fc.property(fc.uuid(), fc.uuid(), async (creatorId, otherId) => {
    const obs = await createObservation({ created_by: creatorId });
    
    // 創建者可以刪除
    await expect(deleteObservation(obs.id, creatorId, 'org-1')).resolves.not.toThrow();
    
    // 其他人不能刪除
    if (otherId !== creatorId) {
      await expect(deleteObservation(obs.id, otherId, 'org-1')).rejects.toThrow();
    }
  })
);

// 發現：邊界情況（同一個 user、特殊字符等）
```

**Mutation Testing 會做什麼：**
```
Mutation 1：移除權限檢查
if (obs.created_by !== userId) {
  // throw new Error('Permission denied');  ← 刪除
}

執行測試 → 測試失敗嗎？
如果失敗 ✅ 好測試（有驗證權限檢查）
如果通過 ❌ 壞測試（沒有驗證權限檢查）

Mutation 2：反轉邏輯
if (obs.created_by === userId) {  // ← 改為 ===
  throw new Error('Permission denied');
}

執行測試 → 測試失敗嗎？
```

---

## 何時用哪個？

| 情況 | 用什麼 |
|------|-------|
| 寫新功能，想驗證邊界 | Property-Based Test |
| 寫完測試，想驗證測試品質 | Mutation Testing |
| 重構程式碼，想確保行為不變 | 兩個都用 |
| 安全相關（auth、多租戶） | 兩個都用 |
| 快速開發迭代 | Property-Based Test |
| 定期品質檢查（CI/CD） | Mutation Testing |

---

## EduNote 建議

### 現在（已有 property-based test）
✅ 繼續用 fast-check 驗證邊界

### 加上 Stryker（推薦）
```bash
# 安裝
pnpm add -D @stryker-mutator/core @stryker-mutator/jest-runner

# 執行
pnpm stryker run

# 查看報告
open reports/mutation/index.html
```

**重點檢查的文件：**
- `lib/api/observations.ts` — 權限檢查、多租戶隔離
- `lib/api/auth.ts` — 認證邏輯
- `hooks/useObservations.ts` — 資料查詢邏輯

**目標 mutation score：** 70–80%（不需要 100%，有些 mutation 是無意義的）

---

## 簡單總結

```
Property-Based Test (fast-check)
→ 「我的程式碼在各種輸入下都正確嗎？」
→ 發現邊界條件問題

Mutation Testing (Stryker)
→ 「我的測試本身有沒有問題？」
→ 發現假陽性測試、測試覆蓋不足

兩個都用
→ 既驗證程式碼，也驗證測試
→ 最高品質保證
```
