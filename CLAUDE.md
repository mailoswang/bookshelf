# 我的書櫃（個人藏書紀錄 Web App）

單人使用的藏書紀錄網頁（漫畫／小說）。掃描 ISBN 入庫、自動歸類系列、跨裝置同步。部署在 GitHub Pages，從 iPhone「加入主畫面」使用。回覆請用繁體中文。

## 結構
- `index.html`：整個前端（單檔，原生 JS／HTML／CSS，無建置步驟）。hash 路由：`#/`、`#/series/<key>`、`#/book/<id>`、`#/edit/new`、`#/add`、`#/settings`。
- `worker/worker.js`：Cloudflare Worker。`?code=ISBN` 查博客來（書名、作者、定價、封面轉 base64）；`/sync`、`/cover/<id>` 同步（KV 綁定 `BOOKS`、密鑰 `SYNC_TOKEN`）。
- `manifest.webmanifest`、`icon-*.png`、`apple-touch-icon.png`、`splash-*.png`：主畫面 App 所需。

## 資料
- localStorage：`bookshelf.v1`（書）、`bookshelf.tombs`（刪除紀錄）、`bookshelf.sync`、`bookshelf.sort`、`bookshelf.theme`、`bookshelf.quick`。
- 書的欄位：id、isbn、title、series、cat（系列內分類）、vol、author、price、cover、note、date、updatedAt、coverAt、`_up`、`pending`。
- 同步：以 `updatedAt` 較新者為準；封面是 data URI 時另外走 `/cover/<id>`。

## 慣例
- 保持單檔、不引入建置工具或框架。
- 介面是藏藍色調的 Liquid Glass 近似（CSS 變數在 `:root` 與 `:root[data-theme="dark"]`）。
- 全頁沒有上方橫幅，所有導覽與操作都在底部懸浮列（`setDock()`）。
- 書名用 `clean()` 去掉「（電子書）」後，再用 `guessFromTitle()` 判斷系列／集數／分類。

## iPhone 主畫面模式的坑（已踩過）
- 網頁本身不能被拖動：`body` 固定、只有 `main` 捲動，並用 touchmove 攔截。
- 主畫面模式偶爾可視高度比螢幕矮一截（少一個狀態列高度）：`vpCheck()` 偵測後加 `.short`，底部淡出改成實色。設定頁有「畫面診斷」可看數字。
- 鍵盤高度只在輸入框聚焦時才套用（`--kb`），否則 Safari 工具列會誤觸發。
- 已移除離線／Service Worker；`index.html` 會清掉舊的 SW 與快取。

## 部署
推到 `main`，GitHub Pages 自動發布。第一次改啟動畫面或圖示後，iPhone 要刪掉主畫面圖示再重新加入。Worker 用 `npx wrangler deploy`（或貼到 Cloudflare 後台），密鑰用 `npx wrangler secret put SYNC_TOKEN`，不要把密鑰寫進程式碼或提交。
