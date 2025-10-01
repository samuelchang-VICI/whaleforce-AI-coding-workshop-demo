# Todo App Demo

> 此檔為專案進度的唯一真實來源（Source of Truth）。任何變更必須先反映於此。

## 1) confirm_model
- [ ] 定義欄位與型別：id, title(1–100), description(<=2000), status(enum), priority(enum), due_date(>=created_at), tags(List[str], 去重<=10<=20), created_at/updated_at(UTC)
- [ ] 決定列舉與預設：status(todo|in_progress|done, default=todo), priority(low|medium|high, default=medium)
- [ ] 規範 tags 正規化流程：trim → lower → 去重
- [ ] 定義 toggle 規則：todo|in_progress → done；done → todo

## 2) design_api
- [ ] 路由與 I/O：GET /health；GET/POST /todos；GET/PATCH/DELETE /todos/{id}；PATCH /todos/{id}/toggle；PATCH /todos/bulk
- [ ] 查詢參數：status, q(不分大小寫), tag, page>=1, page_size<=100, sort=created_at|updated_at|due_date|priority : asc|desc
- [ ] /bulk 策略：atomic（任一 id 無效 → 400，列出無效 id）

## 3) validation_rules
- [ ] Schema 拆分：TodoCreate（必填）、TodoUpdate（選填且各自合法）
- [ ] 422 統一格式：{ error:"validation_error", details:[{field,msg}] }
- [ ] due_date 與 created_at 關係校驗；tags 數量/長度/去重

## 4) server_impl
- [ ] 專案骨架：main/db/models/schemas/crud/deps/routers/tests
- [ ] 分頁/排序/搜尋/tag 過濾；priority 權重排序；updated_at 自動刷新
- [ ] /toggle 與 /bulk(atomic) 實作；CORS；啟動建表

## 5) client_impl
- [ ] Streamlit：Sidebar 篩選、表格就地編輯（title/status/priority）、勾選批次 → /bulk
- [ ] 表單顯示 422 details；API 包裝（list/create/update/delete/toggle/bulk）

## 6) tests
- [ ] pytest + httpx：CRUD、分頁、排序、q+tag、422、toggle、bulk 邊界
- [ ] 臨時 SQLite、夾具、seed

## 7) final_output
- [ ] 一次性輸出所有檔案（多檔案區塊），含 README 啟動與測試指令

---

**重要**: 在每一步完成時，必須回到 `TODO.md` 打勾完成進度，才能繼續往下進行!!!
