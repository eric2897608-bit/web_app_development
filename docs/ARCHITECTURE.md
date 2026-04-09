# 系統架構設計 (ARCHITECTURE)：任務管理系統

## 1. 技術架構說明
本專案採用輕量級的全端單體架構 (Monolithic Architecture)，不使用前後端分離，直接由後端渲染畫面並回傳給瀏覽器。

- **後端框架：Python + Flask**
  - **選用原因**：Flask 是一個輕量、高彈性且容易上手的微框架，非常適合小型甚至是中等規模的 Web 應用開發，幫助我們快速實作 MVP。
- **模板引擎：Jinja2**
  - **選用原因**：與 Flask 緊密整合，可在 HTML 內部嵌入 Python 語法邏輯（如迴圈、條件判斷）。由伺服器端渲染 HTML 以降低前端畫面處理複雜度。
- **資料庫：SQLite**
  - **選用原因**：輕量級關聯式資料庫，它將資料儲存在單一檔案 (`database.db`) 中，不需額外架構與設定獨立的資料庫伺服器，大幅降低部署成本，對日常任務管理的需求已相當足夠。

### Flask MVC 模式說明
- **Model（模型）**：管理於 `app/models/` 之中。負責直接操作 SQLite 進行資料的存取對接與商業邏輯處理（例如新增任務、切換任務「完成」狀態等）。
- **View（視圖）**：管理於 `app/templates/` 之中。主要是充滿 Jinja2 標籤的 HTML 網頁模板，專門負責將 Model 產出的最終結果渲染並排版呈現給使用者。
- **Controller（控制器）**：管理於 `app/routes/` 之中。作為與瀏覽器的橋樑，接收 HTTP 請求 (GET, POST)，並呼叫對應的 Model 處理資料，決定最終要發送哪一個 View 模板給前端。

---

## 2. 專案資料夾結構
為了程式碼的健康度與易維護性，採用依職責分離原則所規劃的樹狀結構：

```text
web_app_development/
├── app/
│   ├── models/           ← 資料庫模型 (Models)：負責 SQLite 資料庫的存取與資料表定義
│   ├── routes/           ← 路由 (Controllers)：定義各功能 API 位址與頁面進入點 (如 task_routes.py)
│   ├── templates/        ← 視圖 (Views)：Jinja2 渲染用的 HTML 模板 (如 index.html)
│   └── static/           ← 靜態資源：放置 CSS 樣式表、純前端 JS 腳本或圖片資源
├── instance/             
│   └── database.db       ← 實體資料庫檔案 (自動產生，通常不進入版本控制)
├── docs/                 
│   ├── PRD.md            ← 產品需求文件
│   └── ARCHITECTURE.md   ← 系統架構文件 (本文件)
├── requirements.txt      ← 專案的 Python 第三方套件依賴清單
└── app.py                ← 關鍵進入點：負責啟動 Flask App 與各類初始化設定
```

---

## 3. 元件關係圖

以下展示使用者於介面操作（如新增一個任務）後，系統內部處理的資料與請求流向圖示：

```mermaid
flowchart LR
    Browser[瀏覽器 (Client)] -->|HTTP GET/POST| Router[Flask Route (Controller)]
    Router -->|1. 發起操作或查詢請求| Model[Model (資料庫層)]
    Model -->|2. 寫入或查詢資料| DB[(SQLite)]
    DB -->|3. 回傳 SQL 結果| Model
    Model -->|4. 將資料封裝回傳| Router
    Router -->|5. 傳遞變數給模板| Template[Jinja2 Template (View)]
    Template -->|6. 渲染為完整的 HTML| Router
    Router -->|7. 回傳 HTTP Response| Browser
```

---

## 4. 關鍵設計決策

1. **伺服器端渲染 (SSR)**
   - 全面採用 Server-Side Rendering (SSR)，前端不使用龐大的 SPA 框架 (如 React)。這樣做的最大的好處在於：減少初期開發的時間成本與複雜度，符合我們目前只做核心功能的產品方向。
2. **路由與模型拆分抽離**
   - 我們「不會」把所有程式碼全部堆放在 `app.py` 中。反之，利用資料夾進行分類：`routes` 專責定義 URL，而 `models` 負責管理資料庫連線與查詢。此結構設計保留了後續專案擴展功能（如擴充會員系統、增加 API 接口）而不導致程式碼難以找尋的問題。
3. **選擇關聯式資料庫（而非讀寫純文字檔案）**
   - 即便只是一個常見的 Todo List，我們仍設計一開始便引入 SQLite。透過 SQL，我們在未來很容易篩選「已完成」、「建立於三日內的任務」，且資料一致性會比自建 JSON 讀寫來得可靠許多。
