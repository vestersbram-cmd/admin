# FastAPI Style Guide

Simple, efficient FastAPI code.

## File Structure
```
project/
├── app.py                  # Main FastAPI app
├── routers/                # APIRoute handlers
│   ├── __init__.py
│   └── items.py
├── models.py               # Pydantic models
└── utils.py               # Utilities (optional)
```

## Minimal App Pattern

```python
# app.py
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: init db, connections, etc.
    yield
    # Shutdown: close connections

app = FastAPI(
    title="My API",
    description="Description",
    version="1.0.0",
    lifespan=lifespan,
)

@app.get("/health")
async def health():
    return {"status": "ok"}
```

## Router Pattern

```python
# routers/items.py
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter(prefix="/items", tags=["items"])

class Item(BaseModel):
    name: str
    quantity: int = 1

class ItemResponse(BaseModel):
    id: int
    name: str
    quantity: int

# In-memory store
items_db: dict[int, dict] = {}
next_id = 1

@router.get("")
async def list_items():
    return list(items_db.values())

@router.post("", response_model=ItemResponse)
async def create_item(item: Item):
    global next_id
    id = next_id
    next_id += 1
    items_db[id] = {"id": id, **item.model_dump()}
    return items_db[id]

@router.get("/{item_id}")
async def get_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return items_db[item_id]

@router.delete("/{item_id}")
async def delete_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    del items_db[item_id]
    return {"deleted": True}
```

Include in main app:
```python
from routers.items import router as items_router
app.include_router(items_router)
```

## Pydantic Models

### Request Model
```python
class CreateItemRequest(BaseModel):
    name: str
    quantity: int = 1
    optional_field: str | None = None
```

### Response Model
```python
class ItemResponse(BaseModel):
    id: int
    name: str
    quantity: int
```

## CORS Middleware

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## State Management

```python
# Set state in lifespan
@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.cache = {}
    app.state.repo = None
    yield
    
app = FastAPI(lifespan=lifespan)

# Access in routes
@router.get("/data")
async def get_data(request: Request):
    return request.app.state.cache
```

## HTTPException Error Handling

```python
from fastapi import HTTPException, status

@router.get("/item/{item_id}")
async def get_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found"
        )
    return items_db[item_id]
```

## Query Parameters

```python
@router.get("/search")
async def search(q: str = "", limit: int = 10, offset: int = 0):
    return {"q": q, "limit": limit, "offset": offset}
```

## Path Parameters

```python
@router.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"id": user_id}
```

## Request Body

```python
@router.post("/create")
async def create(data: Item):
    return data
```

## Background Tasks

```python
from fastapi import BackgroundTasks

def cleanup():
    # Cleanup task
    pass

@router.post("/task")
async def run_task(background_tasks: BackgroundTasks):
    background_tasks.add_task(cleanup)
    return {"started": True}
```

## Dependency Injection

```python
from fastapi import Depends

async def get_db():
    db = connect()
    try:
        yield db
    finally:
        db.close()

@router.get("/data")
async def get_data(db = Depends(get_db)):
    return db.fetch()
```

## Responses

### JSONResponse
```python
from fastapi.responses import JSONResponse

@router.get("/json")
async def json():
    return JSONResponse({"key": "value"})
```

### PlainTextResponse
```python
from fastapi.responses import PlainTextResponse

@router.get("/text", response_class=PlainTextResponse)
async def text():
    return "Plain text"
```

### StreamingResponse
```python
from fastapi.responses import StreamingResponse

@router.get("/stream")
async def stream():
    async def gen():
        for i in range(10):
            yield f"data: {i}\n\n"
    return StreamingResponse(gen(), media_type="text/event-stream")
```

### FileResponse
```python
from fastapi.responses import FileResponse

@router.get("/download")
async def download():
    return FileResponse("path/to/file.txt", filename="file.txt")
```

## Static Files

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
```

### SPA (Single Page App)
```python
from fastapi.responses import HTMLResponse
from pathlib import Path

STATIC_DIR = Path(__file__).parent / "static"

@app.get("/{full_path:path}")
async def serve_spa(full_path: str):
    # Serve index.html for non-API routes
    return HTMLResponse((STATIC_DIR / "index.html").read_text())
```

## Logging

```python
import logging

logger = logging.getLogger(__name__)

@router.post("/action")
async def action():
    logger.info("Action performed")
    return {"ok": True}
```

## Global Exception Handler

```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"Unhandled exception: {exc}", exc_info=True)
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"}
    )
```

## Health Check

```python
@app.get("/health")
async def health():
    return {"status": "healthy"}

# Or with details
@app.get("/health")
async def health():
    return {
        "status": "healthy",
        "service": "myapp",
        "version": "1.0.0"
    }
```

## Database Integration

### SQL with asyncpg
```python
# In lifespan
app.state.pool = await asyncpg.create_pool(dsn)

# In router
@router.get("/users")
async def list_users(request: Request):
    pool = request.app.state.pool
    async with pool.acquire() as conn:
        users = await conn.fetch("SELECT * FROM users")
    return users
```

## Running

```python
import uvicorn

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

CLI:
```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

## Best Practices

1. Use `APIRouter` for grouping endpoints
2. Use Pydantic for request/response validation
3. Use `HTTPException` for errors
4. Use app.state for shared state
5. Return direct dicts when no validation needed
6. Keep handlers simple - one per route
7. Use lifespan for startup/shutdown
8. No unnecessary abstraction layers
9. Simple is better than complex
10. No auth unless needed
11. No middleware unless used
12. Health check is essential

## Common Patterns Summary

| Pattern | Code |
|:---------|:-----|
| Minimal app | `app = FastAPI()` |
| Lifespan | `@asynccontextmanager async def lifespan(app)` |
| Router | `router = APIRouter(prefix="/items")` |
| Include | `app.include_router(router)` |
| CORS | `app.add_middleware(CORSMiddleware, ...)` |
| State | `app.state.cache = {}` |
| Health | `@app.get("/health")` |
| Error | `raise HTTPException(status_code=404, detail="...")` |
| Query | `async def search(q: str = "")` |
| Body | `async def create(item: Item)` |
| Background | `background_tasks.add_task(cleanup)` |
| Response | `return JSONResponse({...})` |
| Static | `app.mount("/static", StaticFiles(...))` |
| Logging | `logger = logging.getLogger(__name__)` |
| Exception handler | `@app.exception_handler(Exception)` |