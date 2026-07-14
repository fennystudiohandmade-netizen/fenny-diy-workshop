---
name: daily-summary
description: 每日下班摘要 — 在每天台北時間 18:30 執行,彙整使用者今天的工作活動(Google Calendar、Gmail),產生簡短 summary 與視覺化儀表板截圖,然後透過 Telegram 傳「下班了唷!」訊息給使用者。由每日 Routine 自動觸發,也可手動執行 /daily-summary。
---

# 每日下班摘要(Daily Work Summary)

你要為使用者(Fenny,b123105@gmail.com)產生今天的下班摘要。全程使用繁體中文。
「今天」指台北時間(Asia/Taipei, UTC+8)的今天,先用 `TZ=Asia/Taipei date` 確認日期。

## 步驟 0:讀取 Telegram 憑證

憑證存在 repo 根目錄的 `.secrets/telegram.json`(已加入 `.gitignore`,不會被 commit),格式:

```json
{ "bot_token": "123456789:AA...", "chat_id": "123456789" }
```

- 檔案不存在或缺欄位:中止流程,改用 PushNotification 告知使用者「Telegram 憑證未設定,請提供 bot token」,不要繼續往下執行。
- 讀到後,後續所有 Telegram API 呼叫都用 `https://api.telegram.org/bot<bot_token>/<method>`。

## 步驟 1:蒐集今天的活動

用 ToolSearch 以關鍵字載入需要的工具(MCP 工具的前綴每個 session 可能不同,不要寫死名稱,用關鍵字搜尋,例如 "calendar list events"、"gmail search threads"),然後平行從以下來源蒐集(某個來源的連接器不可用時直接跳過,不要中斷流程,最後在訊息中註明該來源今天沒有資料):

1. **Google Calendar**(list events):今天 00:00–23:59(台北時間)的行程與會議。
2. **Gmail**(search threads):今天寄出的信件,查詢語法 `from:me after:YYYY/MM/DD`。

同時也蒐集**最近 7 天**每天的會議數與寄信數(不是只看今天單一個數字),供儀表板畫出一條趨勢線,方便看出今天是比平常忙還是比較閒。儀表板每天都是從來源即時重新計算,不依賴本地狀態檔。

## 步驟 2:寫 summary

用蒐集到的資料寫一份簡短摘要(繁體中文、口語、友善):

- **今天完成的事**:3–8 條列點,合併同主題的瑣碎項目,寫「做了什麼事」而不是流水帳。
- **開了哪些會**:會議名稱與時間。
- **待追蹤**:明顯還沒收尾的事(未回的重要信等),沒有就略過。

## 步驟 3:產生視覺化儀表板截圖

1. 先讀 `dataviz` skill(用 Skill 工具)取得圖表設計規範。
2. 產生一個自包含的 HTML 儀表板檔案(存到 scratchpad 目錄,例如 `dashboard.html`),內容:
   - 標題「Fenny 的工作儀表板」與今天日期
   - 今日完成事項清單(勾選樣式)
   - 最近 7 天活動趨勢圖(會議數/寄信數,折線或長條)
   - 固定寬度版面(例如 900px),不要依賴瀏覽器 viewport 自適應,因為要截圖成手機看的圖片
   - 淺色/深色主題都要支援,但截圖用淺色(deviceScaleFactor 用預設,背景不要透明)
3. 用容器內建的無頭 Chromium 把這個 HTML 截圖成 PNG(存到同一個 scratchpad 目錄,例如 `dashboard.png`):

   ```bash
   /opt/pw-browsers/chromium --headless --disable-gpu --no-sandbox \
     --screenshot=/path/to/dashboard.png --window-size=900,1200 \
     "file:///path/to/dashboard.html"
   ```

   視內容長度調整 `--window-size` 的高度,避免截圖被裁切。

## 步驟 4:傳訊息給使用者(Telegram)

1. 傳送截圖(multipart 上傳,`chat_id` 用步驟 0 讀到的值):

   ```bash
   curl -s -F chat_id="<chat_id>" -F photo=@/path/to/dashboard.png \
     -F caption="下班了唷!🎉" \
     "https://api.telegram.org/bot<bot_token>/sendPhoto"
   ```

2. 再傳一則文字訊息帶完整 summary(`sendMessage`,`parse_mode` 用 `Markdown`):

   ```bash
   curl -s -X POST "https://api.telegram.org/bot<bot_token>/sendMessage" \
     -d chat_id="<chat_id>" \
     -d parse_mode="Markdown" \
     --data-urlencode text="今天的小結:
   (summary 內容)"
   ```

3. 兩次呼叫都要檢查回傳的 `"ok":true`;若 `ok:false`,把錯誤訊息記錄下來並改用 PushNotification 通知使用者傳送失敗與原因。

## 注意事項

- 任一資料來源失敗都不要讓整個流程失敗,盡力用拿得到的資料完成,並在 summary 裡註明哪個來源今天沒資料。
- 不要建立 PR、不要動 repo 的網站內容;這個 skill 只讀取資料、產生截圖、發送 Telegram 訊息。
- 假日或今天完全沒有活動時,文字訊息改為輕鬆的「今天沒偵測到工作活動,好好休息!」,但截圖照常附上。
- Bot token 是密鑰,絕對不要把它寫進 commit、log 摘要或任何會進入 git 歷史的檔案。
