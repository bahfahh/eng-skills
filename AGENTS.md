# Skills 創建專案

使用者提出需求，由此專案產出對應的 skill。

---

## 如何建立 Skills

- 結構規範：[SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- 官方最佳實踐：[agent-skills/best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

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

## 撰寫指導型 Skill 的規則

1. **只寫 Claude 不知道的內容**，不要重述 AI 的既有知識。
2. **寫概念與方法，不鎖定特定技術**。若有技術限制，放入 references。
3. **描述做什麼，不解釋是什麼**（除非 Claude 真的不會知道）。

**錯誤示範**
> ASP.NET 是一個使用 DDD 架構的 Web 框架，首先你需要...

**正確示範**
> 使用 DDD 架構，依領域切分模組邊界。

---

## Description 撰寫規則

- 使用**第三人稱**（不用「I」或「You」）
- 說明 **Skill 做什麼** + **何時使用**
- 包含關鍵觸發詞（讓 Claude 能正確選到此 Skill）

**範例**
```yaml
description: Analyzes module boundaries and suggests refactoring strategies. Use when reviewing code structure, untangling dependencies, or planning a modular refactor.
```

---

## 結構規範

- `SKILL.md` 主體保持 500 行以內
- 細節內容拆到獨立檔案，從 `SKILL.md` 一層連結過去
- 不要超過兩層的參考巢狀結構

---

## 已完成的 Skills

| Skill 名稱 | 路徑 |
|-----------|------|
| deep-module-refactor-advisor | `skills/deep-module-refactor-advisor/` |
| module-structure-optimizer | `skills/module-structure-optimizer/` |
| unified-code-review-pro | `skills/unified-code-review-pro/` |
