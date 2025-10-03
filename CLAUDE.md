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

### 重要
please continue without asking
