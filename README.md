# Chat Map AI

A full-stack application that integrates a local LLM (Ollama) with Google Maps API to help users discover locations. Users can query the system naturally in plain language to find places to eat, visit, or explore, and view results on an interactive map.

## Overview

This monorepo contains:

- **Backend**: Hono.js API server that orchestrates Ollama LLM queries and Google Maps API calls
- **Frontend**: React + Vite web UI with interactive map display

## Architecture

```
User Query (React UI)
    ↓
Frontend (Vite) → POST /api/search-locations → Backend (Hono)
    ↓
Ollama LLM (localhost:11434) processes query
    ↓
Parse response → Google Maps API calls
    ↓
Return results → Display on interactive map
```

## Tech Stack

| Component | Technology                              |
| --------- | --------------------------------------- |
| Backend   | Node.js, Hono, JavaScript               |
| Frontend  | React, Vite, JavaScript                 |
| LLM       | Ollama (local, self-hosted)             |
| Maps      | Google Maps API, @react-google-maps/api |
| Monorepo  | npm workspaces                          |

## Quick Start

### Prerequisites

- Node.js 18+
- npm or yarn
- Ollama installed and running locally
- Google Maps API key (with Places, Embed, and Directions APIs enabled)

### 1. Install Ollama and Download Model

```bash
# Install Ollama from https://ollama.ai
# Then run:
ollama serve

# In another terminal, download a model (e.g., Mistral or Llama 2)
ollama pull mistral
```

Ollama will be available at `http://localhost:11434`

### 2. Set Up Google Cloud Project

1. Create a Google Cloud project at [console.cloud.google.com](https://console.cloud.google.com)
2. Enable these APIs:
   - Google Maps Platform
   - Places API
   - Embed API
   - Directions API
3. Create an API key with HTTP referrer restrictions (localhost:3001, localhost:5173)
4. Set up billing alerts to monitor usage

### 3. Install Dependencies and Configure

```bash
# From root directory
npm install

# Create backend .env file
cp backend/.env.example backend/.env
# Edit backend/.env and add your GOOGLE_MAPS_API_KEY and OLLAMA_URL

# Create frontend .env file
cp frontend/.env.example frontend/.env
# Edit frontend/.env and set VITE_API_BASE_URL=http://localhost:3001
```

### 4. Start Development Servers

```bash
# Terminal 1: Backend (runs on port 3001)
cd backend
npm run dev

# Terminal 2: Frontend (runs on port 5173)
cd frontend
npm run dev
```

Then open `http://localhost:5173` in your browser.

## Project Structure

```
.
├── backend/
│   ├── src/
│   │   ├── index.js                    # Hono app entry
│   │   ├── routes/
│   │   │   └── api.js                  # REST endpoints
│   │   ├── services/
│   │   │   ├── ollama.js               # Ollama LLM client
│   │   │   └── googleMaps.js           # Google Maps API client
│   │   ├── middleware/
│   │   │   ├── rateLimiter.js          # Rate limiting
│   │   │   └── errorHandler.js         # Error handling
│   │   └── utils/
│   │       └── locationParser.js       # Parse LLM responses
│   ├── package.json
│   ├── .env.example
│   └── .env
├── frontend/
│   ├── src/
│   │   ├── main.jsx
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   ├── SearchForm.jsx          # Query input
│   │   │   ├── MapDisplay.jsx          # Google Maps component
│   │   │   └── ResultsList.jsx         # Location results
│   │   ├── services/
│   │   │   └── api.js                  # Backend API client
│   │   ├── pages/
│   │   └── styles/
│   ├── package.json
│   ├── vite.config.js
│   ├── .env.example
│   └── .env
├── package.json                        # Root workspaces config
├── .github/
│   ├── copilot-instructions.md         # AI agent guidelines
│   └── prompts/
│       └── plan-llmMapsMonorepo.prompt.md
├── README.md
```

## API Endpoints

### Search Locations

```
POST /api/search-locations
Content-Type: application/json

Request:
{
  "query": "best restaurants in downtown Seattle"
}

Response:
{
  "success": true,
  "data": {
    "llmResponse": "I found several great restaurants...",
    "locations": [
      {
        "name": "Restaurant Name",
        "address": "123 Main St",
        "placeId": "ChIJ...",
        "lat": 47.6062,
        "lng": -122.3321,
        "rating": 4.5,
        "types": ["restaurant", "food"]
      }
    ]
  }
}
```

### Get Place Details

```
GET /api/place-details/:placeId

Response:
{
  "success": true,
  "data": {
    "name": "Restaurant Name",
    "address": "123 Main St",
    "phoneNumber": "+1-206-555-0123",
    "website": "https://example.com",
    "rating": 4.5,
    "reviews": 245,
    "openNow": true,
    "hours": [...]
  }
}
```

### Get Map Embed

```
GET /api/map-embed/:placeId

Response:
{
  "success": true,
  "data": {
    "embedUrl": "https://www.google.com/maps/embed?pb=...",
    "directionsUrl": "https://www.google.com/maps/dir/?api=1&destination=..."
  }
}
```

## Environment Variables

### Backend (.env)

```
OLLAMA_URL=http://localhost:11434
GOOGLE_MAPS_API_KEY=your_google_maps_api_key
RATE_LIMIT_REQUESTS=5
RATE_LIMIT_WINDOW_MS=60000
CORS_ORIGIN=http://localhost:5173
PORT=3001
```

### Frontend (.env)

```
VITE_API_BASE_URL=http://localhost:3001
```

## Development

### Run Linter

```bash
npm run lint
```

### Run Tests (when implemented)

```bash
npm run test
```

### Build for Production

```bash
# Backend
cd backend && npm run build

# Frontend
cd frontend && npm run build
```

## Security Considerations

- **API Keys**: Always use environment variables, never commit secrets
- **Google Maps API**: Configure API key restrictions in Google Cloud Console
  - HTTP referrer: Restrict to your domain
  - API restrictions: Limit to required APIs only
- **Rate Limiting**: Backend implements per-IP rate limiting (5 requests/min default)
- **CORS**: Configured for localhost development, update for production
- **Input Validation**: All user inputs validated before processing
- **LLM Response Parsing**: Responses validated and sanitized before use

## Common Issues

### Ollama Connection Failed

- Ensure `ollama serve` is running in a terminal
- Check `OLLAMA_URL` in backend `.env` (default: http://localhost:11434)
- Verify Ollama model is pulled: `ollama pull mistral`

### Google Maps API Error

- Verify API key has required permissions (Places, Embed, Directions)
- Check HTTP referrer restrictions allow localhost:3001 and localhost:5173
- Ensure billing is enabled on Google Cloud project

### CORS Errors

- Verify `CORS_ORIGIN` in backend `.env` matches frontend URL
- For development, should be `http://localhost:5173`

### Frontend Can't Connect to Backend

- Ensure backend is running on port 3001
- Check `VITE_API_BASE_URL` in frontend `.env`
- Verify no firewall blocking localhost:3001

## Next Steps

1. Set up Google Cloud project and obtain API key
2. Install Ollama and download a model
3. Configure environment variables
4. Run development servers
5. Test end-to-end query flow
6. Implement additional features as needed

## References

- [Ollama Documentation](https://ollama.ai)
- [Google Maps API Documentation](https://developers.google.com/maps)
- [Hono Framework](https://hono.dev)
- [React Documentation](https://react.dev)
- [Vite Documentation](https://vitejs.dev)
