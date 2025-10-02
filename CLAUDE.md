# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python demo project using uv for package management. The project is minimal with a single entry point.

## Development Commands

### Running the application
```bash
python main.py
```

### Package management
This project uses uv. Install dependencies with:
```bash
uv sync
```

Run with uv:
```bash
uv run python main.py
```

## 專案規格

### 套件

fastapi
uvicorn[standard]        # 本地開發：熱重載/uvloop/httptools
sqlmodel                 # 內含 SQLAlchemy；用 SQLite 免額外驅動
pydantic                 # v2 系列，SQLModel/ FastAPI 共同依賴
httpx                    # 前端/測試端呼叫 API
streamlit                # Demo 前端

### 程式需求

資料模型 (Todo)
	•	id: int, PK
	•	title: str (1–100)
	•	description?: str (<=2000)
	•	status: enum(“todo”, “in_progress”, “done”) (default “todo”)
	•	priority: enum(“low”, “medium”, “high”) (default “medium”)
	•	due_date?: datetime (>= created_at)
	•	tags?: List[str] (<=10, 單個<=20, 去重)
	•	created_at: datetime
	•	updated_at: datetime

後端 (FastAPI)
	•	GET /health
	•	GET /todos (支援 status, q, tag, page, page_size, sort=due_date:asc)
	•	POST /todos
	•	GET /todos/{id}
	•	PATCH /todos/{id}
	•	DELETE /todos/{id}
	•	PATCH /todos/{id}/toggle
	•	PATCH /todos/bulk

前端 (Streamlit)
	•	左邊 sidebar：篩選器 (status, priority, tag, 搜尋, 排序)
	•	中間：Todo 列表 (可 inline 編輯 title/status/priority, 可勾選批次更新)
	•	右邊：建立/編輯表單 (帶驗證)
	•	上方 Tab：Baseline / TODO / Subagent 三種版本

### **Subagent 委派規範**
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
```
