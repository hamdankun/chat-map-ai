# Plan: LLM Maps Monorepo with Hono Backend & React Vite Frontend

Build a monorepo containing separate frontend and backend folders. Use Hono for the JavaScript backend integrated with local Ollama LLM and Google Maps API, with a React Vite frontend in a separate workspace.

## Steps

1. Initialize monorepo root with package.json (workspaces configuration for npm/yarn) or use Turbo for task orchestration
2. Create `/backend` folder: Hono project with dependencies (hono, axios, dotenv, cors)
3. Create `/frontend` folder: React Vite project with dependencies (@react-google-maps/api, axios)
4. Install and run Ollama locally with chosen model (Llama 2 or Mistral)
5. Set up Google Cloud project: create Maps API key with usage limits, billing alerts, API key restrictions
6. Implement Hono backend endpoints in `/backend`: `POST /api/search-locations` (query Ollama), `GET /api/place-details/:id` (Google Maps data), `GET /api/map-embed`
7. Integrate Ollama client in backend: connect to http://localhost:11434, parse LLM responses
8. Build React UI in `/frontend`: query form, LLM response display, interactive Google Maps component with markers and direction links
9. Implement security: environment variables per workspace (.env files), input validation, rate limiting, CORS configuration, error handling

## LLM & Google Maps Integration Architecture

**No MCP servers needed.** Use a simple, practical approach:

```
User Query (React UI)
    ↓
Hono Backend (Orchestrator)
    ├─ Step 1: Send query to Ollama (Mistral 7B)
    │  → LLM parses intent: extract location type, cuisine, city
    ├─ Step 2: Parse LLM response → structured data
    ├─ Step 3: Call Google Maps Places API with parsed data
    └─ Step 4: Return enriched results to frontend
    ↓
Frontend displays map with markers & directions
```

**Why this approach:**

- ✓ Ollama has limited MCP support; backend orchestration is simpler
- ✓ Full control over API security and rate limiting
- ✓ Backend acts as the "brain" coordinating LLM + Google Maps
- ✓ Better for assessment (shows practical architecture understanding)

## Recommended LLM Model

**Use: Mistral 7B**

- Size: 4.1GB download | RAM: 8GB minimum
- Fast responses (good for interactive use)
- Excellent at parsing location queries
- Best quality-to-size ratio

```bash
ollama pull mistral
ollama serve  # Runs on localhost:11434
```

Alternative: Llama 3.1 8B (4.7GB) - equally good, more established

## Backend Service Architecture

**`backend/src/services/ollama.js`** - Parse user queries with LLM

```javascript
async function parseLocationQuery(userQuery) {
  const response = await fetch("http://localhost:11434/api/generate", {
    method: "POST",
    body: JSON.stringify({
      model: "mistral",
      prompt: `Extract search terms from: "${userQuery}"
               Return JSON: { type: "restaurant|park|hotel", location: "city", keywords: "..." }`,
      stream: false,
      temperature: 0.7,
    }),
  });
  const result = await response.json();
  return JSON.parse(result.response);
}
```

**`backend/src/services/googleMaps.js`** - Query Google Maps API

```javascript
async function searchPlaces(query, location) {
  const response = await fetch(
    `https://maps.googleapis.com/maps/api/place/textsearch/json?query=${query}&location=${location}&key=${GOOGLE_MAPS_API_KEY}`
  );
  return response.json();
}

async function getPlaceDetails(placeId) {
  const response = await fetch(
    `https://maps.googleapis.com/maps/api/place/details/json?place_id=${placeId}&key=${GOOGLE_MAPS_API_KEY}`
  );
  return response.json();
}

async function getEmbedUrl(placeId) {
  return `https://www.google.com/maps/embed?pb=...&q=place_id:${placeId}`;
}
```

**`backend/src/utils/locationParser.js`** - Parse LLM responses

```javascript
function extractPlaceData(llmResponse) {
  // Handle edge cases: invalid responses, no locations found, ambiguous input
  // Extract type, location, keywords from unstructured LLM text
  // Return structured query for Google Maps API
}
```

## API Endpoints

**POST /api/search-locations**

```javascript
// Request: { query: "Find pizza restaurants in downtown Seattle" }
// Response:
{
  "success": true,
  "data": {
    "llmResponse": "I found several great pizza places...",
    "locations": [
      {
        "placeId": "ChIJ...",
        "name": "Pizzeria Name",
        "address": "123 Main St",
        "rating": 4.5,
        "types": ["restaurant", "food"]
      }
    ]
  }
}
```

**GET /api/place-details/:placeId**

```javascript
// Response:
{
  "success": true,
  "data": {
    "name": "Pizzeria Name",
    "address": "123 Main St, Seattle, WA",
    "phoneNumber": "+1-206-555-0123",
    "website": "https://example.com",
    "rating": 4.5,
    "reviews": 245,
    "openNow": true,
    "hours": ["9 AM – 10 PM", ...]
  }
}
```

**GET /api/map-embed/:placeId**

```javascript
// Response:
{
  "success": true,
  "data": {
    "embedUrl": "https://www.google.com/maps/embed?pb=...",
    "directionsUrl": "https://www.google.com/maps/dir/?api=1&destination=..."
  }
}
```

## Security Best Practices

1. **Google Maps API Key**

   - Store in `.env` (never hardcode)
   - Enable API key restrictions in Google Cloud Console:
     - HTTP referrer: `localhost:3001`, `localhost:5173` (dev)
     - API restrictions: Limit to Places, Embed, Directions APIs only
   - Set up billing alerts and daily usage limits

2. **Rate Limiting** (implement in backend middleware)

   - 5 requests per minute per IP (configurable)
   - Prevents abuse of Google Maps quota
   - Return 429 (Too Many Requests) when exceeded

3. **Input Validation**

   - Validate all user queries before sending to Ollama/Google Maps
   - Sanitize LLM responses before using in API calls
   - Handle malformed Google Maps responses gracefully

4. **Error Handling**
   - Try-catch all async operations
   - Log errors with context (function name, input, error details)
   - Return user-friendly error messages (not stack traces)
   - Handle Ollama timeouts and connection failures

## Further Decisions

1. **Monorepo Tool**: Use npm/yarn workspaces (simple, sufficient for this project)

2. **Frontend State Management**: Use React Context + hooks (simple) or Redux (if complex)

3. **Caching**: Add response caching for repeated queries (Redis, or in-memory for dev)

4. **Development Workflow**: Single root `npm run dev` to start backend + frontend concurrently

## Technology Stack

- **Monorepo**: npm/yarn workspaces
- **Backend**: Node.js, Hono framework, JavaScript
- **Frontend**: React, Vite, JavaScript
- **LLM**: Ollama (local, self-hosted)
- **Maps**: Google Maps API, @react-google-maps/api
- **Security**: dotenv, input validation, rate limiting
- **Package Manager**: npm or yarn

## Project Structure

```
chat-map-ai/
├── package.json (root workspaces config)
├── backend/
│   ├── package.json
│   ├── src/
│   │   ├── index.js (Hono app)
│   │   ├── routes/
│   │   ├── services/
│   │   ├── middleware/
│   │   └── utils/
│   ├── .env
│   └── .env.example
├── frontend/
│   ├── package.json
│   ├── src/
│   │   ├── main.jsx
│   │   ├── components/
│   │   ├── pages/
│   │   └── services/
│   ├── .env
│   └── .env.example
└── Tests.pdf
```

## Assessment Requirements Coverage

- ✓ Run local LLM (Ollama)
- ✓ Google Maps API integration
- ✓ Backend API (Hono, JavaScript)
- ✓ Best practices for Google Maps security (API key restrictions, rate limiting)
- ✓ Map embedding and direction links
- ✓ Code quality and workflow assessment focus
