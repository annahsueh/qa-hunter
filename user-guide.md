# QA Hunter 使用說明書 🎯
> 給第一次使用的人看的完整指南

---

## QA Hunter 是什麼？

QA Hunter 是一個 Claude skill，讓你只要提供三樣東西：
- Figma 設計稿連結
- Notion 規格文件連結
- 要測試的網頁網址

Claude 就會自動：
1. 讀懂規格和設計
2. 列出完整測試項目
3. 用瀏覽器逐項驗收
4. 需要 DB 資料時給你 SQL
5. 把結果寫回 Notion

---

## 使用前準備

### 必要條件

**1. Claude in Chrome 擴充套件**

用來讓 Claude 控制你的瀏覽器進行測試。

安裝步驟：
1. 打開 Chrome，前往 [Chrome 線上應用程式商店](https://chromewebstore.google.com/)
2. 搜尋「Claude in Chrome」
3. 點「加到 Chrome」→「新增擴充功能」
4. 點右上角拼圖圖示 → 找到 Claude → 用你的 **Claude 訂閱帳號**登入

> ⚠️ 重要：擴充套件要用你的 Claude 付費帳號登入，不需要跟 Chrome 設定檔的帳號一樣。

**2. Sequel Ace（如果需要調整 DB）**

部分測試情境需要調整資料庫資料，建議安裝 [Sequel Ace](https://sequel-ace.com/)（免費的 MySQL GUI）。

**3. 確認測試環境**
- 測試網址使用 UAT 或 Staging 環境，不要用正式站
- 準備好測試帳號的帳密

---

## 如何啟動

在 Claude Code 聊天框輸入：

```
幫我 QA 驗收這個功能：
- Figma：https://www.figma.com/design/...
- 規格：https://www.notion.so/...
- 網址：https://web-uat.yoursite.com/feature
```

或者更簡單：

```
qa-hunter
Figma：[連結]
Notion：[連結]
測試網址：[連結]
```

---

## 完整測試流程說明

### Phase 1：讀懂規格與設計（約 2-3 分鐘）

Claude 會自動：
- 讀取 Notion 規格，摘取功能邏輯、UI 文案、數字邊界
- 從 Figma 截取各狀態畫面
- 產出結構化測試清單，分成：
  - A 系列：文案 / 資訊顯示
  - B 系列：UI 狀態變化
  - C 系列：核心功能邏輯
  - D 系列：按鈕 / 互動行為
  - F 系列：迴歸測試

你可以在這個階段確認測試範圍是否正確。

---

### Phase 2：瀏覽器測試（主要測試階段）

Claude 會連接你的 Chrome 瀏覽器，逐項驗收。

**你需要做的事：**

1. 確保 Chrome 已開啟，且 Claude in Chrome 擴充套件已登入
2. 把要測試的頁面開在 Chrome 裡
3. 看著 Claude 操作，隨時可以打斷並說「這個有問題」

**遇到需要 DB 資料時：**

Claude 會暫停測試，說：「這個測試需要調整 DB，請執行以下 SQL：」

你只需要：
1. 打開 Sequel Ace
2. 複製貼上 Claude 給的 SQL
3. 執行
4. 截圖或回報結果
5. Claude 繼續測試

---

### Phase 3：寫回 Notion 報告（約 1-2 分鐘）

測試完成後，Claude 會把結果寫入 Notion 的「測試回報」區塊：

```
測試日期：2026-05-21｜測試環境：UAT｜PASS：8｜FAIL：2｜待確認：1

## ▊ 🌐 Web / 後端問題回報列表
### [功能]
- [ ] [❌ FAIL] B-3：里程碑 3 天達標後，圖示未變色 — 預期橘色，實際灰色
- [x] [✅ PASS] D-1：按鈕文案「前往 APP 領取上週獎勵」正確
```

---

## DB 操作常識

### 為什麼需要改 DB？

有些測試情境無法在畫面上操作（例如：上週累積簽到 7 天、帳號有特定金額點數），只能直接修改資料庫來製造測試資料。

### 安全守則

| 做 | 不做 |
|----|------|
| 每個 SQL 都加 WHERE 條件 | 不下沒有 WHERE 的 UPDATE/DELETE |
| 修改前先 SELECT 確認 | 不在正式站執行 |
| 用 UPDATE 不用 DELETE | 不亂改別人的帳號資料 |

### 常用 SQL 範本

**查看某會員本週簽到紀錄：**
```sql
SELECT id, times, checkin_date, checkin_type
FROM `daily_checkin`
WHERE `member_id` = '[member_id]'
  AND `checkin_date` BETWEEN '[週一]' AND '[週日]'
ORDER BY `checkin_date`;
```

**移動簽到日期（釋放今天讓你可以再簽）：**
```sql
UPDATE `daily_checkin`
SET `checkin_date` = '[新日期]'
WHERE `member_id` = '[member_id]'
  AND `checkin_date` = '[今天]';
```

**清除本週紀錄（重置測試）：**
```sql
DELETE FROM `daily_checkin`
WHERE `member_id` = '[member_id]'
  AND `checkin_date` BETWEEN '[週一]' AND '[週日]';
```

**設定跨週獎勵（模擬上週達標）：**
```sql
UPDATE `checkin_award`
SET `ca_id` = '[新UUID]',        -- 一定要換新 ca_id，否則領獎會報錯
    `checkin_times` = '[天數]',   -- 3、5、或 7
    `points` = '[點數]',          -- 50、100、或 300
    `receive_time` = NULL,        -- NULL = 尚未領取 = 按鈕會出現
    `expire_time` = '[週日 23:59:59]'
WHERE `member_id` = '[member_id]'
  AND `data_type` = 'week'
  AND `week` = '[YYYYWW]';       -- 上週的週次，例如 202620
```

> ⚠️ `checkin_award` 的 (member_id + data_type + week) 是唯一索引，同一週只能有一筆，所以只能 UPDATE，不能 INSERT 新的。

---

## 常見問題

### Q：Claude 說「無法連接瀏覽器」？
確認 Claude in Chrome 擴充套件已安裝且登入。登入後重新整理 Claude Code 頁面再試一次。

### Q：瀏覽器連上了但 Claude 說「無法截圖」？
在 Chrome 的擴充套件頁面，確認 Claude in Chrome 已取得「在所有網站上讀取及變更你的所有資料」的權限，或手動允許測試網站的存取。

### Q：SQL 執行後回到網頁還是沒變化？
強制重新整理頁面（Cmd+Shift+R），確保瀏覽器沒有快取舊資料。

### Q：`checkin_award` UPDATE 後領取還是報錯？
確認 `ca_id` 有換成新的值。舊的 `ca_id` 可能已被系統標記為「已處理」，必須換新才有效。

### Q：想重置帳號重頭測試怎麼做？
```sql
-- 刪除本週所有簽到
DELETE FROM `daily_checkin`
WHERE `member_id` = '[member_id]'
  AND `checkin_date` BETWEEN '[週一]' AND '[週日]';

-- 重置跨週獎勵（設為未領取）
UPDATE `checkin_award`
SET `receive_time` = NULL
WHERE `member_id` = '[member_id]'
  AND `week` = '[YYYYWW]';
```

---

## 週次計算速查

| 概念 | 說明 | 範例 |
|------|------|------|
| 週次格式 | YYYYWW（ISO week number） | 2026年第21週 = `202621` |
| 本週範圍 | 週一 ~ 週日 | 5/18（一）~ 5/24（日） |
| 查本週週次 | `SELECT YEARWEEK(NOW(), 1)` | 返回 `202621` |
| 查上週週次 | `SELECT YEARWEEK(NOW() - INTERVAL 7 DAY, 1)` | 返回 `202620` |

---

## 測試結果代號說明

| 代號 | 意思 | 後續動作 |
|------|------|---------|
| ✅ PASS | 符合規格與設計 | 無需處理 |
| ❌ FAIL | 與規格或設計不符 | 開 bug ticket，等修復後複測 |
| ⚠️ 待確認 | 不確定是 bug 還是規格問題 | 詢問 PM / 工程師釐清 |
| ⏸️ 未測 | 需要 DB 設定或特殊條件，本次跳過 | 排入下次測試 |

---

*QA Hunter v1.0 — 讓找 bug 變成一種樂趣 🎯*
