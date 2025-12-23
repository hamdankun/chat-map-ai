# Copilot Instructions for Chat Map AI LLM Maps Project

## Project Overview

This is a **monorepo interview assessment** building a location discovery system powered by a local LLM (Ollama) and Google Maps API integration. The architecture separates concerns into a **Hono backend** (Node.js) and a **React Vite frontend**, connected via REST API.

**Key Assessment Focus**: Code quality, security best practices for API key management, LLM integration patterns, and API design.

## Architecture & Data Flow

```
User Query (React UI)
    ↓
Frontend (Vite) → POST /api/search-locations → Backend (Hono)
    ↓
Hono Router → Ollama Service (http://localhost:11434)
    ↓
Parse LLM Response → Extract locations → Google Maps API calls
    ↓
Return Place Details + Map Data → Frontend renders @react-google-maps/api
```

**Key Integration Points**:

- `/backend/src/services/ollama.js` - Ollama client connecting to local LLM instance
- `/backend/src/services/googleMaps.js` - Google Maps API client with security patterns
- `/frontend/src/components/MapDisplay.jsx` - Interactive map rendering component
- `/backend/src/middleware/` - Rate limiting, CORS, request validation

## Critical Workflows

### Local Development Setup

```bash
# Root workspace (npm workspaces configured)
npm install                    # Install all workspace dependencies

# Terminal 1: Start Ollama LLM
ollama serve                   # Runs on localhost:11434 (required)
ollama pull mistral           # Download model once

# Terminal 2: Start backend
cd backend && npm run dev      # Port 3001 (or configured in .env)

# Terminal 3: Start frontend
cd frontend && npm run dev     # Port 5173 (Vite default)
```

### Environment Variables

- **Backend** (`.env`): `OLLAMA_URL`, `GOOGLE_MAPS_API_KEY`, `RATE_LIMIT_REQUESTS`, `RATE_LIMIT_WINDOW_MS`
- **Frontend** (`.env`): `VITE_API_BASE_URL` (e.g., `http://localhost:3001`)
- **Never** commit `.env` files; use `.env.example` as template

### Building for Assessment

- Run linter: `npm run lint` (eslint configured at root)
- Run tests (if implemented): `npm run test`
- Check both backend and frontend work end-to-end with real API calls

## Project-Specific Patterns

### 1. Hono Route Structure

Routes in `/backend/src/routes/` follow RESTful naming:

- `POST /api/search-locations` - Query Ollama with user input, return parsed locations
- `GET /api/place-details/:id` - Fetch single place details from Google Maps
- `GET /api/map-embed/:id` - Generate embedded map markup or directions link

**Pattern**: Each route calls relevant service (OllamaService, GoogleMapsService) with error handling.

### 2. LLM Response Parsing

Ollama returns unstructured text. Backend must parse location names/addresses from responses.

- Store parsing logic in `/backend/src/utils/locationParser.js`
- Example: Extract "restaurants in downtown Seattle" → Array of place queries
- Handle edge cases: invalid responses, no locations found, ambiguous input

### 3. Google Maps API Security

**Critical for assessment**:

- Store `GOOGLE_MAPS_API_KEY` in environment variables (never hardcode)
- Implement **API key restrictions** in Google Cloud Console:
  - HTTP referrer: `localhost:3001` and `localhost:5173` (dev)
  - Restrict to Maps APIs only (Embed, Places, Directions)
- Implement **rate limiting** at backend: max 5 requests/minute per IP (configurable)
- Use `.env.example` to document required secrets

### 4. API Request/Response Contract

Define consistent response structure for all endpoints:

```javascript
// Success
{ success: true, data: { /* endpoint-specific data */ } }

// Error
{ success: false, error: "User-friendly message", code: "ERROR_CODE" }
```

### 5. Frontend-Backend Communication

- Frontend uses Axios instance in `/frontend/src/services/api.js`
- Base URL from `VITE_API_BASE_URL` environment variable
- Error handling: retry logic for transient failures, user-friendly error messages

## Conventions & Patterns to Follow

**File Naming**:

- Components: PascalCase (`MapDisplay.jsx`, `SearchForm.jsx`)
- Services: camelCase (`ollamaService.js`, `googleMapsService.js`)
- Utilities: camelCase (`locationParser.js`, `rateLimiter.js`)

**Code Style**:

- JavaScript (not TypeScript) - simpler setup for assessment
- ESLint configured at root; run before commits
- Async/await for all async operations (avoid `.then()` chains)

**Error Handling**:

- All API endpoints wrap logic in try-catch
- Log errors with context (function name, input data)
- Return meaningful error codes to frontend (not generic 500)

**Testing Approach** (if implementing):

- Backend: Jest for unit tests on services (Ollama parsing, API calls)
- Frontend: Vitest for component tests (MapDisplay, SearchForm)

## Key Files & Directories

| Path                                     | Purpose                                             |
| ---------------------------------------- | --------------------------------------------------- |
| `backend/src/index.js`                   | Hono app entry, middleware setup (CORS, rate limit) |
| `backend/src/services/ollama.js`         | Ollama client, text generation                      |
| `backend/src/services/googleMaps.js`     | Google Maps API client, places/details queries      |
| `backend/src/routes/`                    | REST endpoint handlers                              |
| `backend/src/middleware/`                | Rate limiting, validation, error handling           |
| `frontend/src/components/MapDisplay.jsx` | @react-google-maps/api component                    |
| `frontend/src/services/api.js`           | Axios instance for backend calls                    |
| `.env.example`                           | Document required environment variables             |

## Assessment Evaluation Checklist

When implementing or reviewing code:

- [ ] Google Maps API key properly secured (env var, API restrictions, rate limiting)
- [ ] Ollama integration handles connection failures gracefully
- [ ] LLM response parsing is robust (handles edge cases, ambiguous input)
- [ ] Rate limiting prevents API quota abuse
- [ ] CORS correctly configured for dev (localhost:3001/5173) and future deployment
- [ ] Error messages are user-friendly, not internal stack traces
- [ ] Code follows consistent patterns across backend/frontend
- [ ] `.env.example` documents all required secrets

## Common Gotchas

1. **Ollama Not Running**: Backend will hang/timeout if Ollama not listening on localhost:11434. Check `ollama serve` is running.
2. **Google Maps API Key**: Must have correct permissions in Google Cloud Console; "Embed API" differs from "JavaScript API".
3. **CORS Errors**: Ensure backend CORS middleware allows frontend origin (localhost:5173 in dev).
4. **Rate Limiting**: Implement at backend, not frontend; prevent abuse of external APIs (Ollama, Google Maps).
5. **Environment Variables**: Frontend uses `VITE_` prefix for Vite to inject at build time.

## What to Avoid

- ❌ Hardcoding API keys or secrets
- ❌ Making Google Maps API calls directly from frontend (exposes key)
- ❌ Trusting LLM output format; always validate/parse
- ❌ Unhandled promise rejections in async operations
- ❌ Generic error messages ("Error" instead of "Failed to fetch locations: timeout")
