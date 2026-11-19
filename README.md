# 市場均衡與供需調整｜課堂互動 — 上線部署指南

高中經濟科課堂實驗平台：學生端（登記、報價、供需圖、答題）+ 教師控制台（即時看板、控場、CSV 匯出），通過伺服器事件匯流排即時同步。

## 檔案結構

```
market-equilibrium-classroom/
├── index.html          前端（學生端 + 教師控制台，單檔）
├── server.js           零依賴後端（靜態託管 + 事件儲存 + 即時推送 SSE）
├── package.json
├── render.yaml         Render 一鍵部署藍圖
└── data/               課堂資料（自動生成 events.json，不入庫）
```

## 一、免費上雲（Render，推薦，約 10 分鐘）

準備工作：註冊 GitHub 帳號（免費），註冊 [render.com](https://render.com)（免費，可用 GitHub 直接登入）。

1. **程式碼已在 GitHub：** https://github.com/woodykam0/market-equilibrium-classroom （公開倉庫，6 個檔案）
   - 修改後重新上傳：`git add -A && git commit -m "update" && git push`（或在倉庫頁點 Add file → Upload files 拖放覆蓋）
2. **Render 部署**
   - render.com 登入 → **New +** → **Blueprint** → 選擇剛才的倉庫 → Render 會讀取 `render.yaml` → **Apply**
   - （或手動：New + → **Web Service** → 選倉庫 → Build Command 留空 → Start Command 填 `node server.js` → Plan 選 **Free**）
3. 等待 1–3 分鐘部署完成，你會得到一個網址，例如 `https://market-equilibrium-classroom.onrender.com`
4. **課堂連結**（可按班別分開，資料互不干擾）：
   - 甲班：`https://你的網址/?instanceId=01GIA`
   - 乙班：`https://你的網址/?instanceId=01GIB`
   - 不加 `?instanceId=` 則所有人共用一個課室（`default`）
5. 教師控制台：開啟同一網址 →「教師控制台」→ 密碼 `@W423021k`（實際值由 Render 的 Environment 設定決定）。首次部署後請到 Render → market-equilibrium-classroom → Environment → Add Variable 設定 `TEACHER_PASSWORD=@W423021k`，然後 Manual Deploy 一次。
6. 資料互不干擾的課堂連結（可同時開多個班）：`https://你的網址/?instanceId=01GIA`、`?instanceId=01GIB`。

> **免費方案須知（重要）**
> - 免費實例 15 分鐘無人訪問會休眠，上課時學生持續使用不會休眠；若課前閒置，第一個打開頁面的人會等待約 30–60 秒喚醒，之後正常。
> - 免費方案磁碟是**臨時的**：重新部署時 `data/events.json` 會被清空。**每節課後請在教師控制台下載 CSV** 留存。資料在同一次部署期間（包括休眠再喚醒）是保留的。
> - 若需要永久保存資料庫，可在 Render 加掛付費持久磁碟，或換 Supabase/Railway 的免費資料庫（告訴我，我可以幫你改造）。

## 二、本機試跑（確認功能正常）

```powershell
cd C:\Louiskam\market-equilibrium-classroom
node server.js
```
瀏覽器打開 `http://localhost:3000` 即可。按 `Ctrl+C` 停止。

## 三、備用方案（如果雲端不想用 / 不穩定）

- **同 Wi-Fi（局域網）**：本機執行上方兩行命令，學生瀏覽器輸入「你的內網 IP:3000」（如 `192.168.1.50:3000`）。老師電腦要開機且與學生同一網路。
- **臨時公網網址**：安裝 [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/) 後執行 `cloudflared tunnel --url http://localhost:3000`，會得到一個 `*.trycloudflare.com` 網址（無需註冊，每次重啟網址會變）。

## 四、課堂流程提示

1. 學生用課堂連結登記 → 教師控制台可見名冊、買家/賣家人數自動平衡。
2.「開始新一輪 / 結束本輪（自動最小價差撮合）/ 解鎖環節 2 / 3 / 重置整個活動」即時同步全班（通常 1 秒內）。
3. 學生端右上角「立即更新」可手動刷新；正常情況下不需要，畫面會自動更新。
4. 老師每節課結束後按「下載 CSV」匯出學習數據。

## 五、安全說明

- 教師密碼只存於伺服器環境變數，頁面原始碼不含密碼；控場指令（開始/結束/重置/撮合結果）必須持有教師密碼才能寫入，學生無法偽造。
- 學生提交（報名、報價、圖表、答題）匿名寫入，請在課堂上提醒同學勿亂輸入他人姓名。
- 如需更改教師密碼：Render → market-equilibrium-classroom → Environment → 修改 `TEACHER_PASSWORD` → Save（會自動重新部署）。版本庫中一律不存密碼；未設定時教師控制功能自動鎖定。
