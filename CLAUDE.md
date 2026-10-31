# group-dining-decider — 聚餐決定器（聚餐選擇困難症助手）

單檔靜態網頁（`index.html`＋`mrt-stations-data.js`＋`manual.html`＋PWA）。整合人數、預算、飲食限制、交通 → 公平集合點＋推薦聚餐類型 → 轉盤／投票拍板 → LINE 公告。2026-10-31 建立。

- 本機預覽 port **8845**（`.claude/launch.json` 的 `group-dining-decider`）。8843 已被 ecommerce-competitor-analyzer 使用。
- **尚未上線**：部署 GitHub Pages 前先問使用者（新工具部署前確認的偏好）。visitor-badge 已預埋 `m255525.groupdiningdecider`。
- 範圍台北／新北：`mrt-stations-data.js` 直接複製自 `小工具/food-finder/`（149 站，不含機場捷運；資料缺「行天宮」站）。food-finder 補站時這裡也要同步。
- 與 food-finder 分工：food-finder＝一個人附近吃什麼（純 LLM）；本工具＝一群人條件取交集（規則式為主，AI 選用）。

## 架構重點
- `HUBS`：39 個餐廳多的集合候選站，旗標 `park／late／biz／room／family`。`CUISINES`：30 種聚餐類型，`p` 人均價位、`s` 適合人數、`d` 飲食限制（`x` 排除／`c` 需問店家）、`room/kids/elder/biz/drink/bday` 0–2、`t` 時段 LDN、`needLate`（夜市）。
- `travelMin`：直線距離（haversine）× 交通方式係數（捷運再乘 1.15）＋基本時間；開車加停車時間（park 站較少）；**捷運出發站與集合站沒有共同路線 +7 分轉乘**（沒加這個時淡水＋新店＋南港會跑出「三重」）。
- `rankHubs`：cost＝0.6×最遠＋0.4×平均（min-max 公平）。
- `evalCuisine`：硬排除（預算起價 >最低預算×1.15、時段不符、人數 >上限×1.6、任何人飲食 `x`）；計分 預算25＋飲食20（每位需問店家 −5）＋人數10＋場合15＋需求15＋想吃15；最近 3 筆紀錄吃過 −8。
- `decide`：每個類型在前 6 個公平站中挑 `-cost + hubBonus×2` 最佳的站；**排序以契合度 total 為主**，hubBonus 只當小數點級的 tie-break（之前直接加分導致顯示分數不單調）。
- 分享：成員＋條件 base64url 編碼進 `#s=`，載入後自動 `go()` 並清掉 hash。
- 儲存鍵：`gddState`、`gddSettings`、`gddApi`（BYOK 金鑰）、`gddHistory`（最多 50）、`gddWarnSeen`（首訪警語）、`gddMarquee`。
- AI（選用）：`callLLM` 沿用 fridge-recipe-notes 格式，prompt 不送成員暱稱；回傳 JSON 陣列店家，標「AI 建議・請查證」。
- 標準配備：跑馬燈（工作區公告 GAS、滑鼠移入暫停）、⚙ 常用設定（含「關於本工具」）、警語彈窗（首訪自動）、manual.html#creator 完整版、PWA、visitor-badge。
- 測試掛鉤 `window._gdd`（state、decide、go、applyPreset、rankHubs、travelMin、shareUrl、openDecide…）。

## 驗證紀錄（2026-10-31，Python Playwright）
- 淡水＋新店＋南港 → 雙連／中山／民權西路／台北車站；素食＋海鮮過敏＋預算 300 → 迴轉壽司因海鮮過敏排除、高價類型全因預算排除。
- 4 組範例皆有結果；換一批、轉盤、拍板公告、存紀錄（重整後仍在）、投票、分享連結還原（375px 無水平捲動）、首訪警語只跳一次、無金鑰按 AI 會開設定、SW 註冊成功、無 console 錯誤。AI 真實金鑰未測。
