# Todo App Demo

**重要**:
- 本專案以根目錄的 TODO.md 作為唯一真實來源（SoT）。在每個步驟結尾，必須：
  1) 回傳一個完整覆蓋的 `TODO.md` 檔案（使用多檔案區塊：```path=TODO.md ...```），
  2) 將該步的核取方塊改為 [x]，
  3) 在「進度日誌」區塊追加一行摘要。
- 如果沒有 TODO.md，請先透過使用者的 Request 和 `todo-planner` 建立 `TODO.md`

**Subagent 委派規範**
	•	Main agent 的職責：編排 & 維護 TODO.md。
	•	Subagent 的職責：僅在 implementation 階段被交付單一任務並回傳結果，且前端後端和 testing 可以同時進行（不得自行修改 TODO.md）。
	•	交付時請傳遞結構化 payload，等待 subagent 回覆，於回覆尾段會包含 RESULT: 區塊（JSON 摘要）。
	•	Main agent 需：
	•	解析 RESULT: → 合併檔案變更 → 更新 TODO.md → 移動到下一步；
	•	若 RESULT.followups 有缺口，先補齊上下文再重呼叫，不可讓 subagent 自行擴權。

**SUBAGENT_PAYLOAD 範例（Main agent 在委派時使用）**
```json
{
  "task": "一句話描述當前單一目標",
  "constraints": ["不得修改 TODO.md", "遵守 contracts/openapi.yaml", "只在指定目錄讀寫"],
  "context": {
    "paths": ["client/", "contracts/openapi.yaml"],
    "notes": "任何額外注意事項或片段",
    "accept_result_format": "必須在訊息末尾附 RESULT: JSON（files_changed, notes, followups）"
  }
}
```

## confirm_model
- 定義 `Todo` 欄位與型別：`id(int, PK)`, `title(str 1–100)`, `description?(<=2000)`, `status(enum)`, `priority(enum)`, `due_date?(datetime)`, `tags?(List[str])`, `created_at/updated_at(datetime, UTC)`.
- 設定列舉與預設：`status=('todo'|'in_progress'|'done')`, `priority=('low'|'medium'|'high')`；預設 `todo` / `medium`。
- 設計 `tags` 正規化與去重：`trim → lower → 去重`；限制總數 ≤10、單個長度 ≤20。
- 明確 `due_date >= created_at` 規則；允許 `None`。
- 定義 `toggle` 規則：`todo|in_progress → done`；`done → todo`。
- 草擬 SQLModel 類別（只列欄位與驗證思路，不輸出完整專案）。

## design_api
- 撰寫 `contracts/openapi.yaml`
- 撰寫 `contracts/shared_models.py`
- 列出所有路由與 I/O：  
  `GET /health`；`GET/POST /todos`；`GET/PATCH/DELETE /todos/{id}`；`PATCH /todos/{id}/toggle`；`PATCH /todos/bulk`。
- 規劃查詢參數：`status`、`q`（title/description LIKE, 不分大小寫）、`tag`、`page>=1`、`page_size<=100`、`sort=field:dir`。
- 定義排序白名單：`created_at|updated_at|due_date|priority`（`priority` 需映射權重比較）。
- 設計 `/bulk` payload 與策略：`{ "ids":[...], "set":{ "status" 或 "priority" } }`；採 **atomic**（任一無效 id → 400 並列出）。
- 統一錯誤回覆：422 時回 `{ "error":"validation_error", "details":[{ "field","msg" }] }`；其餘依情境回 400/404/500。
- 撰寫 1–2 組成功/錯誤 `contracts/error_schema.json` 範例以對齊契約。

## validation_rules
- 拆分 Schema：`TodoCreate`（必填/限制）、`TodoUpdate`（全部選填但各自合法）。
- 集中欄位驗證：  
  `title` 長度、`description` 長度、`tags` 數量/單長/正規化/去重、`due_date >= created_at`。
- 建立 422 轉換 handler：攔截校驗錯誤並轉成統一格式。
- 確保 `PATCH` 只更新提供的欄位，未提供的不動。
- 在 model 或 service 層補 business rules（如 `updated_at` 自動刷新）。

## server_impl
- 建立目錄結構：`server/main.py`, `db.py`, `models.py`, `schemas.py`, `crud.py`, `deps.py`, `routers/todos.py`, `tests/test_todos.py`。
- 資料庫啟動：建立 engine、Session 依賴、`on_startup` 自動建表（可選 `seed()`）。
- 實作查詢：合併 `status/tag/q` 過濾；`page/page_size` 分頁；`sort` 欄位與方向驗證＋`priority` 權重排序。
- CRUD 與業務：`POST/GET/PATCH/DELETE`；`/toggle` 依規則切換；所有變更更新 `updated_at`。
- `/bulk`：先驗證 `ids` 全存在 → 原子更新 `status/priority` → 回 `{ updated: n }` 或附 `items`。
- 防呆與相容：CORS 設定、錯誤處理（400/404/422）、SQLite 對 `ilike` 的兼容（以 `lower()` + `LIKE`）。

## client_impl
- 建立 Streamlit 結構：`client/app.py`（UI 主檔）、`api.py`（HTTP 包裝）、`widgets.py`（共用元件）。
- Sidebar 篩選器：`status`、`priority`、`tag`、`q`、`sort`（欄位+方向）、`page/page_size`。
- 列表區：表格顯示欄位；支援**勾選多筆**；點擊 `title/status/priority` 觸發就地編輯→呼叫 `PATCH`。
- 批次操作：選取多筆 → 設定 `status/priority` → 呼叫 `/bulk`，成功後刷新；失敗顯示錯誤。
- 表單：建立/編輯 Todo；顯示 422 `details[]`（逐欄位錯誤）；送出成功後清空與刷新列表。
- 輔助資訊：在頁面顯示目前查詢參數摘要、分頁資訊與總數。

## tests
- 測試環境：pytest + httpx `TestClient`；臨時 SQLite（每測重建或使用 transaction）。
- 夾具：`client`、`session`、`seed_todos(n)`。
- 編寫代表性案例：  
  - 建立成功 + 邊界：最短/最長 `title`、`tags` 去重。  
  - 列表：分頁 `page=2&page_size=5`；排序 `sort=priority:desc`。  
  - 搜尋/標籤組合：`q` + `tag` 同時作用且仍正確分頁/排序。  
  - 更新：`due_date < created_at` 觸發 422；一般欄位 `PATCH` 成功。  
  - 切換：`/toggle` 符合規則。  
  - 批次：`ids` 含不存在 → 400；全存在 → `updated == len(ids)`。  
  - 刪除：`DELETE` 後 `GET` 404。
- 確保測試互不污染、報告可讀（失敗訊息包含欄位與原因）。

## final_output
- 以**多檔案**區塊一次輸出完整專案：`server/`, `client/`, `tests/` 或 `pyproject.toml`, `README.md`。
- `README.md` 包含：安裝、啟動（`uvicorn server.main:app --reload`、`streamlit run client/app.py`）、環境變數（若有）、`pytest -q` 指令。
- 檢查清單：依賴齊全（`fastapi`, `sqlmodel`, `uvicorn`, `pydantic`, `httpx`, `pytest`, `streamlit`…）、匯入路徑正確、實際可啟動並通過最小測試。
- 建立 `start.sh`，方便使用者同時啟動前端和後端
