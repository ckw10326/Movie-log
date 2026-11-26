# 電影紀錄網站（Movie-log）

- 這是什麼：個人電影片單、評價與觀後心得資料庫，附網頁查詢介面。
- 用途：新增、查詢看過的電影與心得。
- 網址：https://ckw10326.github.io/Movie-log/
- 公開或私有：公開（Public）。

# 電影紀錄與分析（Movie-log）

這是小華的個人電影評價查詢、觀後心得與片單資料庫，並透過 GitHub Pages 提供個人網頁版瀏覽介面：
- **GitHub 倉庫**：https://github.com/ckw10326/Movie-log （Public，以免費啟用 GitHub Pages）
- **個人電影牆網頁**：https://ckw10326.github.io/Movie-log/

---

## 🤖 AI 操作與省 Token 規則（必讀）

1. **嚴禁整份讀取 `movies.jsonl`**：
   - 除非使用者明確要求匯出全部資料，否則**禁止**用讀檔工具一次載入整個 `movies.jsonl`。
2. **查重與瀏覽片單**：
   - 有終端機工具的 AI：使用 `rg` / `grep` 或 Python 腳本只搜尋並回傳「目標電影那 1 行」。
   - 無終端機工具的 AI：請先只讀取輕量索引 [`titles.txt`](./titles.txt)。
3. **新增、校對與暫存機制**：
   - **先收資料、確認再寫入**：若小華先分批提供劇情整理或資料，並指示「先不要寫入」，AI 應先暫存於對話中，待心得補充完整並確認後再一次寫入並推送。
   - **檢查重複**：新增前先檢查是否已存在相同電影，避免重複建檔。
   - **新增電影**：用腳本直接附加（append）到 [`movies.jsonl`](./movies.jsonl) 最後一行，並同步更新 [`titles.txt`](./titles.txt)。
   - **順手抓取海報（`poster`）**：新增電影時，應根據片名／年份／導演抓取可用的公開海報圖片網址（優先選用支援跨站顯示、無防盜鏈阻擋的來源，如 NeoDB、Wikimedia 等，並驗證 HTTP 200），寫入 `poster` 欄位。
   - **更新心得／資料**：用腳本只修改對應 `id` 的那一行，並更新 `updated_at`。
   - **保留原話與校對**：若片名來自語音輸入（可能有錯字）或劇情描述模糊，先查證核對後再標記 `status: "已確認"`（未確定前標為 `"待確認"`），並保留使用者的原始心得。
4. **網頁（`index.html`）省 Token 與更新規則**：
   - **純前端即時讀取（無需 `build.py`）**：[`index.html`](./index.html) 會直接在瀏覽器透過 `fetch('./movies.jsonl')` 渲染最新資料。**日常新增或修改電影時，AI 嚴禁重寫或修改 `index.html`**（不需執行任何生成 HTML 的腳本，網頁更新的額外 Token 消耗為 0）。
   - **主動詢問與確認**：當小華提供完整資料並寫入 `movies.jsonl` 之後，AI 應**主動詢問小華是否需要調整或更新 `index.html` 呈現方式**（若無新欄位或版面調整需求，則直接提醒 GitHub Pages 已隨 `movies.jsonl` 推送自動更新即可）。
   - **網頁版面設計共識**：
     - 主畫面維持**緊湊小字表格式**，**不顯示海報縮圖與長篇內容**。
     - **海報與詳細內容（含個人推薦度、主演、平台評價、心得、討論重點、跨片連結）僅在點擊該列彈出詳細視窗（Modal）時才載入顯示**。
     - 結構化故事整理（`plot_outline`）預設摺疊收合，點擊後才展開。

---

## 🗂️ `movies.jsonl` 資料欄位規格（Schema）

每行 1 個 JSON 物件，支援以下欄位（選填欄位若無資料可省略或設為 `null`）：

- `id`（number）：流水編號
- `title`（string）：中文片名
- `original_title`（string | null）：原文片名
- `year`（number | null）：年份
- `director`（string | null）：導演
- `cast`（string | null，選填）：主演陣容
- `status`（string）：`"已確認"` 或 `"待確認"`
- `watched`（boolean）：是否已看過
- `rating`（string | null，選填）：個人評分／推薦度（例如 `"超級喜歡（極力推薦）"`）
- `douban`（object | null）：`{"score": 8.0, "votes": 140000, "dist": "..."}`
- `other_scores`（string | null）：其他平台分數對照
- `user_notes`（string | null）：小華的個人觀後感／原始筆記
- `summary`（string | null）：討論重點與評價總結
- `related_movies`（array | null，選填）：跨片連結／延伸片單
- `plot_outline`（string | null，選填）：結構化故事整理（於網頁彈窗預設摺疊）
- `poster`（string | null，選填）：電影海報圖片網址（僅於彈窗開啟時載入）
- `updated_at`（string）：最後更新日期（`YYYY-MM-DD`）

---

## 🎬 電影分析與討論標準

1. **查評價（防洗分查證模式）**：
   - 先確認原文片名、年份、導演、各地上映時間，避免同名作品混淆。
   - **優先查中國豆瓣**：列出總分、評分人數、短評數，並附上 **1～5 星各星級比例與約略人數**，分析是否有洗分／灌票跡象。
   - **補充其他平台對照**：如 IMDb、Letterboxd、Filmarks、映画.com、爛番茄、PTT 電影板等，並註明票數參考價值。
   - 歸納好評與負評重點（附來源連結）與簡短觀看建議。
2. **觀後感討論與影評練習**（詳見 [`docs/discussion-agent.md`](./docs/discussion-agent.md)）：
   - 涉及結局或關鍵劇情時，先標註「以下有劇透」或先確認是否已看過。
   - 回應時承接使用者的個人觀點，補充網友解讀、導演／原著訪談與跨片主題連結。
   - 協助影評寫作時，注重結構（先下結論、用具體畫面撐觀點、少用抽象形容詞、一段只講一件事）。
   - **深度討論視窗分工**：專注於陪聊討論與影評練習，不直接寫入資料庫；討論完整收尾時，輸出結構化、可複製的文字段落供小華貼到另一個專案進行紀錄。
3. **溝通風格**：繁體中文（台灣習慣用語），親切自然、條理分明。

---

## 📁 檔案說明

- `README.md`：本說明書（AI 交接規則、網頁省 Token 規範與資料欄位規格）。
- `docs/discussion-agent.md`：深度討論視窗的 AI 工作定位與結尾可複製輸出範本。
- `titles.txt`：極簡片單索引（格式：`id | 片名 | 年份 | 是否看過 | 豆瓣分數`）。
- `movies.jsonl`：完整電影資料庫（每行 1 部電影的 JSON 物件）。
- `index.html`：GitHub Pages 個人電影牆前端頁面（自動讀取 `movies.jsonl`，緊湊表格＋彈窗詳情）。
