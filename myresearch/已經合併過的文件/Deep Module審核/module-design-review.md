# Module Design Review — Merged

> **此文件已合併進 `ai-first-code-review`。**
> 請直接使用該 skill，不需要單獨執行本文件的規則。

---

## 合併說明

原本的 D/T/N 規則（模組深度、測試策略、命名與介面設計）已整合進：

**`ai-first-code-review/references/judgment-axes.md`**

| 原規則 | 現在的位置 |
|--------|-----------|
| D-1 單一業務操作一次呼叫 | R1 — Intent Cohesion，Module depth fail pattern |
| D-2 同一 inputs 反覆出現 | R3 — AI-Rewrite Safety，Shallow module fail pattern |
| D-3 區分自由工具 vs 固定序列 | R3 — AI-Rewrite Safety，Shallow module fail pattern |
| D-4 介面不應比實作複雜 | R3 — AI-Rewrite Safety，Interface depth fail pattern |
| T-1 主要測試對象是 facade | R4 — Test Derivability，Wrong test target fail pattern |
| T-3 不測試「有沒有呼叫到特定 util」 | R4 — Test Derivability，Test coupling fail pattern |
| N-1 函數名反映業務行為 | R5 — Domain Visibility，Naming fail pattern |
| N-2 Facade 應有獨立命名 | R5 — Domain Visibility，Naming fail pattern |
| N-3 介面參數用業務語言 | R5 — Domain Visibility，Naming fail pattern |

## 原 PASS/WARN/SUGGEST 語氣的對應

原本的 `WARN` 現在對應 `refactor_recommended`。
原本的 `SUGGEST` 現在對應 `accept_with_risk` + `suggested_action`。
原本的 `PASS` 現在對應 `accept`。
