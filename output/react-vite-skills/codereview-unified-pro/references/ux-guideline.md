---
title: UX Review Guideline (Code Review Gate)
scope: code-review skill
version: 1
---

# UX Review Guideline (Code Review Gate)

目的：把 UX 檢查變成「可重複、可執行」的 Code Review Gate，避免出現 unit test 抓不到的問題：
- 按鈕點了沒反應 / 沒回饋
- Double submit / 重複送出
- 靜默失敗（只有 logger）
- 無限 loading / 無法復原
- 手機體驗破碎（點擊區域、鍵盤遮擋、捲動）

## 必讀文件（Mandatory）

Review UI/流程相關 PR 時，必須先讀並套用：
- `.kiro/steering/ui/interaction-contract.md`（互動契約：Button/Dialog/Toast/Loading/Navigation）
- `.kiro/steering/ui/ui-guidelines.md`（視覺與基本 UI token，含 mobile touch target）
- `docs/memory/ux/nav-button-instant-feedback.md`（導航即時回饋模式）

## Review 輸出要求（必須產出）

只要 PR 有改到 UI/流程，你的 review 需要包含一段 **UX Gate**，至少回答：
1) 這個 action pending 時會發生什麼？能不能連點？
2) 成功後使用者怎麼知道成功了？
3) 失敗後使用者看到什麼？下一步是什麼？
4) 有沒有可能卡在無限 loading？fallback 是什麼？
5) 手機上是否可點、可捲動、鍵盤不遮擋？

只要 PR 有改到**頁面佈局 / 卡片 / 列表密度 / 按鈕群 / 標籤區**（任何容易 RWD 爆版的區域），review 必須加上一段 **RWD Gate**：
1) 是否附上 375px（mobile）與 1440px（desktop）截圖？
2) 是否做過 overflow 壓測（長標題 / 多 tags / 日期很長 / 按鈕很多）？
3) 是否避免手機橫向捲動（no horizontal scroll）？

## 高優先級問題（看到就要擋）

1) Dead click（點了沒有任何回饋）
- 沒有 disabled/pending UI
- 沒有 toast/inline feedback

2) Double submit（連點或重複提交）
- pending 時仍可點、或沒有 single-flight 保護

3) Silent failure（失敗被吞）
- catch 後只 logger、不顯示 UI
- API error 沒映射成使用者可理解的訊息

4) Error mapping 缺失（API 狀態碼無對應 UI 行為）
- 401/403/422/500 等錯誤都顯示同一種訊息或直接吞掉
- 使用者不知道下一步（重新登入/檢查權限/修正輸入/稍後重試）

4) Infinite loading（無限 spinner）
- loading 沒 timeout、沒 fallback（重試/回登入/重新整理/離線提示）

5) Destructive action 無 confirm
- 刪除/停用/覆蓋沒有 dialog confirm 或文案不清楚

6) RWD Gate 缺失（mobile-first 未驗收）
- 沒附 375px + 1440px 截圖
- 沒做 overflow 壓測（長標題/多 tags/按鈕群）
- 手機上出現橫向捲動或重要 CTA 被擠出畫面

## 中優先級問題（建議修）

- Loading 顯示位置不對（例如 nav button 上做 loading，而頁面內容沒 skeleton）
- Toast 使用不一致（同類成功/失敗訊息不同位置/語氣）
- 表單驗證只用 toast（欄位錯誤應 inline）
- 按鈕/狀態缺乏視覺回饋（顏色、disabled、spinner、文字變化都不明顯）
- 文案像工程語言，使用者看不懂（技術詞直接露出）
- Mobile 觸控目標 < 44px 或按鈕太貼邊（含 safe-area）
- 路由切換體感延遲（active 狀態不即時）
- 大量資料仍用直向長列表，造成選擇困難/捲動過長（應考慮 grid/分群/搜尋/虛擬化）

## Review Checklist（逐項勾）

### Button / Form Submit
- 是否有 pending 狀態（disable + 視覺提示）？
- 是否防 double submit？
- 成功是否有 feedback（toast 或 UI state）？
- 失敗是否有可理解的訊息 + 下一步（重試/回上一頁/重新登入）？

### Dialog Confirm
- 是否只對「不可逆/高風險」操作出現？
- 文案是否具體（不要只寫「確定」）？
- 預設 focus 是否避免落在 destructive button？

### Toast / Error UI
- 是否避免把表單驗證錯誤只做 toast？
- 是否避免大量重複 toast（需節流/合併）？
- 是否有 API error mapping（依 status code 顯示不同且可行動的訊息）？

### Loading / Offline
- 是否有 timeout + fallback？
- 是否有 offline 提示（或至少不會永遠 spinner）？

### Navigation
- active 狀態是否即時回饋？

### Content / Copy
- UI 文案是否「使用者可理解」？是否避免技術詞（token/RLS/PGRST...）直接出現在 UI？
- 錯誤訊息是否包含下一步（重試/重新登入/檢查網路/回上一頁）？

### List / Selection at Scale
- 若資料量大（例如學生很多），是否避免純直向長列表？
- 是否提供搜尋/分群/常用置頂？必要時是否考慮 grid 或虛擬化？

### RWD / Overflow（必做壓測）
- 375px：長標題、很多 tags、日期很長、按鈕很多 → 不爆版、不遮擋 CTA、不產生橫向捲動
- 1440px：資訊密度合理、卡片 action 區不破版（可換行/縮排/截斷策略需一致）

## 對作者的追問（當你資訊不足時）

1) 這個按鈕在慢網路或 500/403 時 UI 會怎樣？
2) 這個操作是否可能被連點？是否會產生重複資料？
3) 這個錯誤訊息是給使用者看的，還是只有工程師看得懂？
4) 在手機上，鍵盤打開時提交按鈕是否仍可見？
5) 資料量很大時（例如學生 100+），這個選擇器/列表還好用嗎？有搜尋或分群嗎？
