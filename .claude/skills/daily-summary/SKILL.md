---
name: daily-summary
description: 每日下班摘要 — 在每天台北時間 18:30 執行,彙整使用者今天的工作活動(Google Calendar、Gmail、Slack、GitHub),產生簡短 summary 與視覺化儀表板,然後傳「下班了唷!」訊息給使用者。由每日 Routine 自動觸發,也可手動執行 /daily-summary。
---

# 每日下班摘要(Daily Work Summary)

你要為使用者(Fenny,b123105@gmail.com)產生今天的下班摘要。全程使用繁體中文。
「今天」指台北時間(Asia/Taipei, UTC+8)的今天,先用 `TZ=Asia/Taipei date` 確認日期。

## 步驟 1:蒐集今天的活動

用 ToolSearch 以關鍵字載入需要的工具(MCP 工具的前綴每個 session 可能不同,不要寫死名稱,用關鍵字搜尋,例如 "calendar list events"、"gmail search threads"、"slack search send message"、"github list commits"),然後平行從以下來源蒐集(某個來源的連接器不可用時直接跳過,不要中斷流程,最後在訊息中註明該來源今天沒有資料):

1. **Google Calendar**(list events):今天 00:00–23:59(台北時間)的行程與會議。
2. **Gmail**(search threads):今天寄出的信件,查詢語法 `from:me after:YYYY/MM/DD`。
3. **Slack**(search):今天使用者發過的訊息與參與的討論串,查詢 `from:me after:YYYY-MM-DD` 之類的語法。
4. **GitHub**(list commits / search pull requests):使用者今天在可存取 repo 的 commits、PR、issue 活動。

同時也蒐集**最近 7 天**的每日數量(會議數、寄信數、Slack 訊息數、commits 數),供儀表板畫趨勢圖。儀表板是每天從來源重新計算的,不依賴本地狀態檔。

## 步驟 2:寫 summary

用蒐集到的資料寫一份簡短摘要(繁體中文、口語、友善):

- **今天完成的事**:3–8 條列點,合併同主題的瑣碎項目,寫「做了什麼事」而不是流水帳。
- **開了哪些會**:會議名稱與時間。
- **待追蹤**:明顯還沒收尾的事(未回的重要信、還開著的 PR 等),沒有就略過。

## 步驟 3:更新視覺化儀表板

1. 先讀 `dataviz` skill(用 Skill 工具)取得圖表設計規範。
2. 產生一個自包含的 HTML 儀表板(檔案放 scratchpad 目錄),內容:
   - 標題「Fenny 的工作儀表板」與今天日期
   - 今日完成事項清單(勾選樣式)
   - 最近 7 天活動趨勢圖(會議/信件/Slack/commits,分組長條或折線)
   - 今日活動來源佔比
   - 淺色/深色主題都要支援
3. 用 Artifact 工具發佈。**網址必須保持不變**:先用 `Artifact` 的 `action: "list"` 找標題為「Fenny 的工作儀表板」的既有 artifact,找到就帶 `url` 參數原地更新;找不到才建新的。favicon 固定用 `📊`,`<title>` 固定為「Fenny 的工作儀表板」。

## 步驟 4:傳訊息給使用者

1. **Slack DM**(主要管道):用 Slack 的 search users 工具以 email b123105@gmail.com 找到使用者,再用 send message 工具發 DM,格式:

   > 下班了唷!🎉
   >
   > 今天的小結:
   > (summary 內容)
   >
   > 📊 進度儀表板:(artifact 網址)

2. **PushNotification**(輔助):一行短訊,例如「下班了唷!今天完成 N 件事,摘要已送到 Slack 📊」,status 用 `proactive`。
3. Slack 不可用時的備援:改用 PushNotification 帶重點,並在本對話中輸出完整 summary 與儀表板連結。

## 注意事項

- 任一來源失敗都不要讓整個流程失敗,盡力用拿得到的資料完成。
- 不要建立 PR、不要動 repo 的網站內容;這個 skill 只讀取資料與發訊息。
- 假日或今天完全沒有活動時,訊息改為輕鬆的「今天沒偵測到工作活動,好好休息!」並照常附上儀表板。
