# Copilot Instructions: MCP Integration Exercise

## Project Context

This is a **GitHub Skills learning exercise** on integrating the Model Context Protocol (MCP) with GitHub Copilot. The base application is a FastAPI backend for Mergington High School's extracurricular activities management system with a vanilla JavaScript frontend.

**Key Goal**: Demonstrate how MCP enables Copilot to bridge chat with external systems (GitHub API for issues/PRs).

## Architecture Overview

| Component | Details |
|-----------|---------|
| **Backend** | FastAPI (Python 3.13) serving REST API + static frontend |
| **Frontend** | Vanilla HTML/CSS/JS single-page app at `/static/` |
| **Data Store** | In-memory dictionary (resets on server restart) |
| **Deployment** | DevContainer with Python 3.13, utilities pre-installed |
| **MCP Config** | `.vscode/mcp.json` — GitHub MCP server integration |

**Key Files**:
- [src/app.py](src/app.py) — FastAPI application, Activity/Signup endpoints
- [src/static/app.js](src/static/app.js) — Frontend fetch logic and DOM rendering
- [.vscode/launch.json](.vscode/launch.json) — VS Code debug config (uvicorn + debugpy)
- [.vscode/mcp.json](.vscode/mcp.json) — MCP server configuration (GitHub integration point)

## Getting Started

### Prerequisites
- Dependencies auto-installed by DevContainer setup
- If manual: `pip install -r requirements.txt`

### Run the Application

**Option 1: VS Code Debug** (Recommended)
1. Open Run and Debug panel
2. Select "Start Debugging" or press F5
3. Server starts at `http://localhost:8000` with auto-reload
4. View app in Ports tab or at [http://localhost:8000](http://localhost:8000)

**Option 2: Manual Run**
```bash
uvicorn src.app:app --reload
```

### API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/activities` | Fetch all activities with participant counts |
| POST | `/activities/{name}/signup?email=...` | Sign up for activity |
| DELETE | `/activities/{name}/unregister?email=...` | Unregister from activity |
| GET | `/` | Redirect to frontend |

Swagger docs available at `/docs`; ReDoc at `/redoc`

## Development Guidelines

### Architecture Decisions

1. **In-Memory Storage**: Data persists only during server lifetime. Simplicity prioritized over persistence (appropriate for learning exercise).
2. **Shared Origin**: Frontend and API served from same FastAPI instance. No CORS issues; would break if split to different domains.
3. **Static File Mounting**: Frontend served at `/static/` via FastAPI's StaticFiles handler.
4. **Simple Query Validation**: Email passed as query parameter; loose validation (not production-grade).

### Known Limitations & Pitfalls

⚠️ **Data Loss on Restart**: All activity signups cleared when server restarts.  
⚠️ **No Capacity Check**: Signup succeeds even if activity is full (bug in current code).  
⚠️ **Email Validation**: No strict email format validation; user input accepted as-is.  
⚠️ **Port Conflicts**: If port 8000 in use, debug launch will fail. Check `lsof -i :8000`.  
⚠️ **Single-Threaded**: In-memory mutations are not transaction-safe (acceptable for exercise scope).

### Code Organization

```
src/
├── app.py              # FastAPI application, endpoints, data models
└── static/
    ├── index.html      # Frontend structure
    ├── styles.css      # Responsive styling
    └── app.js          # Event handling, fetch calls, DOM updates
```

**Frontend-Backend Interaction:**
- Frontend fetches `GET /activities` on page load
- Signup form POSTs to `/activities/{name}/signup?email=...`
- Real-time deletion via `DELETE /activities/{name}/unregister?email=...`
- No state management library; direct DOM mutations and async/await

### MCP Integration Context

The `.vscode/mcp.json` file configures the GitHub MCP server:
```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

This enables Copilot Chat to:
- Read/search GitHub issues and pull requests
- Create issues programmatically
- Access repository metadata

**Exercise Flow**: Later steps involve using MCP to create issues from chat, then implement features suggested by Copilot via MCP-enabled workflows.

## Common Tasks

### Adding a New Activity
Edit `activities` dictionary in [src/app.py](src/app.py) following the existing schema:
```python
"Activity Name": {
    "description": "...",
    "schedule": "...",
    "max_participants": 25,
    "participants": []
}
```

### Debugging
- Set breakpoints in VS Code; use Debug Console for REPL
- Check browser console (F12) for frontend errors
- View network requests in DevTools Network tab for API calls

### Static File Changes
- Edit [src/static/app.js](src/static/app.js) or `.css` directly
- Auto-reload via Uvicorn's `--reload` flag (only backend; refresh browser for frontend)

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Port 8000 already in use | `lsof -i :8000` → kill process, or change port in debug config |
| Dependencies not installed | Run `pip install -r requirements.txt` or rebuild DevContainer |
| Browser shows blank page | Check browser console for errors; verify backend running at `/docs` |
| Changes not appearing | Refresh browser (Ctrl+Shift+R for hard refresh if cached) |

## Related Documentation

- [FastAPI Docs](https://fastapi.tiangolo.com) — Framework used for backend
- [Uvicorn Guide](https://www.uvicorn.org) — ASGI server for running FastAPI
- [MCP Documentation](https://modelcontextprotocol.io) — Protocol for AI agent integrations
- [GitHub MCP Server](https://github.com/modelcontextprotocol/servers) — GitHub API integration reference
