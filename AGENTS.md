# Skills 創建專案

使用者提出需求，由此專案產出對應的 skill。

---

## 參考來源

- 結構規範：[SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- 官方最佳實踐：[agent-skills/best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [skills-best-practices](https://github.com/mgechev/skills-best-practices)

---

## 基本原則

### 理解需求再動手

不確定需求時，以選擇題詢問使用者，對齊用途與範圍後再開始建立。

### Skill 的兩種類型

| 類型 | 說明 |
|------|------|
| 工具型 | 提供腳本或命令讓 Claude 執行 |
| 指導型 | 提供流程或規則讓 Claude 遵循 |

---

## Skill 目錄結構

```
skill-name/
├── SKILL.md              # 必要，主入口（500 行以內）
├── scripts/              # 可執行腳本
├── references/           # 參考文件，只從 SKILL.md 一層連結
└── assets/               # 模板、圖示等靜態資源
```

### 三層載入機制（Progressive Disclosure）

| 層級 | 內容 | 何時載入 |
|------|------|----------|
| 1 | `name` + `description`（~100 tokens） | 永遠預載 |
| 2 | `SKILL.md` 主體 | Skill 被觸發時 |
| 3 | 其他資源檔案 | 需要時才讀取 |

**核心原則：只在需要時才載入，避免佔用 context window。**

---

## YAML Frontmatter 規範

```yaml
---
name: skill-name          # 小寫字母、數字、連字號，最多 64 字元
description: ...          # 最多 1024 字元，不能含 XML 標籤
---
```

### name 命名建議

使用動名詞形式（gerund），清楚表達 Skill 的動作：

- `processing-pdfs`、`analyzing-spreadsheets`（推薦）
- `pdf-processing`、`pdf-analyzer`（可接受）
- `helper`、`utils`、`tools`（避免，過於模糊）

---

## Description 撰寫規則

- 使用**第三人稱**（不用「I」或「You」）
- 說明 **Skill 做什麼** + **何時觸發**
- 包含關鍵觸發詞，讓 Claude 能正確選到此 Skill
- 描述可以稍微「積極」，避免 Claude undertriggering

**範例**
```yaml
description: Analyzes module boundaries and suggests refactoring strategies. Use when reviewing code structure, untangling dependencies, or planning a modular refactor.
```

---

## 撰寫 Skill 內容的規則

### 指導型內容

1. **只寫 Claude 不知道的內容**，不要重述 AI 的既有知識。
2. **寫概念與方法，不鎖定特定技術**。若有技術限制，放入 references。
3. **描述做什麼，不解釋是什麼**（除非 Claude 真的不會知道）。
4. 使用**祈使句**（imperative form）直接下指令，不要用敘述文。

**錯誤示範**
> ASP.NET 是一個使用 DDD 架構的 Web 框架，首先你需要...

**正確示範**
> 使用 DDD 架構，依領域切分模組邊界。

### 自由度設定

根據任務的脆弱程度選擇指令的精確度：

| 情境 | 做法 |
|------|------|
| 多種方式都可行 | 高自由度：給方向，讓 Claude 判斷 |
| 有偏好但可彈性 | 中自由度：給模板，允許調整 |
| 步驟不能出錯 | 低自由度：給精確指令，不允許變動 |

### 工具型腳本

- 將脆弱、重複的操作寫成腳本（Python/Bash/Node），讓 Claude 執行而非生成
- 腳本要有清楚的錯誤訊息，讓 Claude 能自我修正
- 明確說明是「執行」還是「參考閱讀」：
  - 執行：`Run scripts/analyze.py to extract fields`
  - 閱讀：`See scripts/analyze.py for the extraction algorithm`

---

## 結構規範

- `SKILL.md` 主體保持 500 行以內
- 細節內容拆到獨立檔案，從 `SKILL.md` **一層**連結過去（不要多層巢狀）
- 參考檔案超過 100 行時，加上目錄（Table of Contents）
- 路徑統一使用正斜線 `/`（不用 `\`）

---

## 建立流程

1. **捕捉意圖**：確認 Skill 的用途、觸發時機、期望輸出
2. **釐清細節**：詢問邊界案例、格式需求、相依套件
3. **撰寫 SKILL.md**：寫好 frontmatter 與指令內容
4. **建立測試案例**：在 `evals/evals.json` 寫 2-3 個真實測試情境
5. **測試與迭代**：觀察 Claude 實際使用行為，針對問題改進

---

## All-in-one Skills（Orchestrator）

當使用者要求製作「all in one skills」時，遵守以下概念與約束：

<all in one skills concept>
## all in one skills
- All in one skills：對外看起來是「一個完整 skill 就能把任務做完、交付結果」，但內部其實是編排器（orchestrator）——會自己拆解任務、安排步驟、整合輸出；使用者不需要知道背後有沒有多個 agent。
- 智能分配 agent 數量：根據「任務複雜度 / 風險 / 面向數 / 工作量」決定只用自己（0 個子 agent）就能做完，或啟動 1 個專家，或啟動多個專家並行；最後一定要合併成單一結論 / 單一交付物（不是丟一堆分散回覆）。
</all in one skills concept>

## 已完成的 Skills

| Skill 名稱 | 用途 |
|-----------|------|
| `security-audit-orchestrator` | All-in-one 全 repo 安全審核編排器：智能啟動 0–3 子 agent（secrets / supply-chain / appsec），最後合併成單一安全報告（不改程式碼） |
| `deep-module-refactor-advisor` | 審計成熟 codebase 的淺層模組與高耦合問題，產出 facade 封裝建議報告（不改程式碼） |
| `module-structure-optimizer` | 功能完成後的後續優化：找封裝候選、移除冗餘抽象、偵測架構/效能風險（N+1、循序 await 等），輸出重構計畫報告 |
| `unified-code-review-pro` | PR/diff 審查協調器，依變更內容自動選擇審查深度，平行執行正確性、安全性、架構、效率等多維度審查 |
| `design-patterns` | 設計模式選型指南，依情境推薦 AI 友善的模式，避免過度工程化 |
| `ai-native-design` | 語言無關的 AI-native 架構設計原則，協助規劃模組邊界、分層責任、穩定介面與測試策略 |
