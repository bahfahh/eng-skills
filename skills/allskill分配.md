# All Skills 分配

> 標記說明：
> - `(你的)` = 自建 skill
> - `(官方)` = 官方 skill（Vercel），安裝時使用 `next-official-` / `react-official-` 前綴命名
> - `⭐ 共用` = 多個 stack 都會用到

---

## Next.js 開發流程

| Phase | Skill | 說明 |
|-------|-------|------|
| 規劃 | `plan-mode-pro` (你的) ⭐ 共用 | requirements / plan / todolist / acceptance / tests |
| 架構設計 | `architecture-ai-native-design` (你的) ⭐ 共用 | 模組邊界、分層責任 |
| 架構設計 | `architecture-design-patterns` (你的) ⭐ 共用 | pattern 選型 |
| 架構設計 | `db-supabase-checklist` coding mode (你的) | DB schema / RLS 規劃 |
| 開發前準備 | `next-checklist` coding/plan mode (你的) | 補 todolist，防止 Next.js 常見陷阱 |
| 開發中 | `next-official-best-practices` (官方, user-invocable: false) | 被其他 skill 引用；RSC / async API / routing 規則 |
| 開發中 | `next-official-cache-components` (官方) | caching 策略（Next.js 16+ 才適用） |
| 開發中 | `next-api-checklist` (你的) | Route Handler / API 設計 |
| 開發中 | `db-supabase-checklist` coding mode (你的) | migration / RLS 變更 |
| Code Review | `codereview-unified-pro` (你的) ⭐ 共用 | PR 全面審查（主入口） |
| Code Review | `next-checklist` review mode (你的) | Next.js 專項深度審查 |
| Code Review | `db-supabase-checklist` review mode (你的) | DB 專項審查 |
| Code Review | `web-official-design-guidelines` (官方) | UI/UX 設計規範 |
| 深度審計 | `review-gate-security-audit` (你的) ⭐ 共用 | 全 repo 安全審計 |
| 深度審計 | `review-gate-performance-audit` (你的) ⭐ 共用 | 全 repo 效能審計 |
| 深度審計 | `review-gate-module-structure-optimizer` (你的) ⭐ 共用 | 封裝候選、N+1、循序 await |
| 深度審計 | `review-gate-deep-module-refactor-advisor` (你的) ⭐ 共用 | 淺層模組、高耦合分析 |
| Bug 修復 | `bugfix-systematic` (你的) ⭐ 共用 | 複雜 bug 系統化調查（開 ticket / 跨服務 / 難重現） |
| Deploy | `vercel-official-deploy` (官方) | Vercel 部署腳本 |
| 版本升級 | `next-official-upgrade` (官方) | 動態抓最新 migration guide + 跑 codemod |

---

## React（非 Next.js，例如 Vite + React）開發流程

| Phase | Skill | 說明 |
|-------|-------|------|
| 規劃 | `plan-mode-pro` (你的) ⭐ 共用 | |
| 架構設計 | `architecture-ai-native-design` (你的) ⭐ 共用 | |
| 架構設計 | `architecture-design-patterns` (你的) ⭐ 共用 | |
| 開發前準備 | `frontend-checklist` coding/plan mode (你的) | 補 todolist，前端通用陷阱 |
| 開發中 | `react-official-best-practices` (官方) | 57 條 React 效能規則 |
| Code Review | `codereview-unified-pro` (你的) ⭐ 共用 | PR 全面審查 |
| Code Review | `frontend-checklist` review mode (你的) | 前端通用專項審查 |
| Code Review | `web-official-design-guidelines` (官方) | UI/UX 設計規範 |
| 深度審計 | `review-gate-security-audit` (你的) ⭐ 共用 | |
| 深度審計 | `review-gate-performance-audit` (你的) ⭐ 共用 | |
| 深度審計 | `review-gate-module-structure-optimizer` (你的) ⭐ 共用 | |
| 深度審計 | `review-gate-deep-module-refactor-advisor` (你的) ⭐ 共用 | |
| Bug 修復 | `bugfix-systematic` (你的) ⭐ 共用 | 複雜 bug 系統化調查（開 ticket / 跨服務 / 難重現） |

---

## Backend .NET / C# 開發流程

| Phase | Skill | 說明 |
|-------|-------|------|
| 規劃 | `plan-mode-pro` (你的) ⭐ 共用 | |
| 架構設計 | `architecture-ai-native-design` (你的) ⭐ 共用 | |
| 架構設計 | `architecture-design-patterns` (你的) ⭐ 共用 | |
| 開發前準備 / 開發中 | `backend-dotnet-checklist` coding/plan mode (你的) | 補 todolist + 開發中參考 Core Guardrails 與 references/ |
| Code Review | `codereview-unified-pro` (你的) ⭐ 共用 | PR 全面審查（主入口） |
| Code Review | `backend-dotnet-checklist` review mode (你的) | .NET 專項深度審查 |
| 深度審計 | `review-gate-security-audit` (你的) ⭐ 共用 | |
| 深度審計 | `review-gate-performance-audit` (你的) ⭐ 共用 | |
| 深度審計 | `review-gate-module-structure-optimizer` (你的) ⭐ 共用 | |
| 深度審計 | `review-gate-deep-module-refactor-advisor` (你的) ⭐ 共用 | |
| Bug 修復 | `bugfix-systematic` (你的) ⭐ 共用 | 複雜 bug 系統化調查（開 ticket / 跨服務 / 難重現） |

---

## 共用 Skills 一覽

| Skill | 適用 |
|-------|------|
| `plan-mode-pro` | Next.js / React / .NET |
| `architecture-ai-native-design` | Next.js / React / .NET |
| `architecture-design-patterns` | Next.js / React / .NET |
| `codereview-unified-pro` | Next.js / React / .NET |
| `review-gate-security-audit` | Next.js / React / .NET |
| `review-gate-performance-audit` | Next.js / React / .NET |
| `review-gate-module-structure-optimizer` | Next.js / React / .NET |
| `review-gate-deep-module-refactor-advisor` | Next.js / React / .NET |
| `bugfix-systematic` | Next.js / React / .NET |

---

## 官方 Skills 命名對照

| 官方原名 | 安裝後使用名稱 |
|---------|--------------|
| `next-best-practices` | `next-official-best-practices` |
| `next-cache-components` | `next-official-cache-components` |
| `next-upgrade` | `next-official-upgrade` |
| `vercel-react-best-practices` | `react-official-best-practices` |
| `web-design-guidelines` | `web-official-design-guidelines` |
| `vercel-deploy` | `vercel-official-deploy` |

---

## 待建立 Skills

| Skill | 對應 Stack | 用途 |
|-------|-----------|------|
| _(目前無待建立項目)_ | | |
