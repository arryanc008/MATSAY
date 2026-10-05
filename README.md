# MATSAY

**Intelligent Maritime Fleet Command Center**
*Navigate Smarter. Optimize Every Voyage.*

Live demo: [matsay.vercel.app](https://matsay.vercel.app)

---

## About

Most shipping operations still run on scattered information. Vessel positions come from one feed, weather from another, port schedules live in a different system entirely, and fuel logs often sit in a spreadsheet somewhere. By the time all of that gets pulled together manually, the window to act on it has usually already closed.

MATSAY brings these sources into one place. It combines live vessel data, weather and ocean conditions, port information, and historical operations data, then runs prediction and optimization on top of that, so a fleet commander can actually compare options before committing to a route or an allocation decision — instead of relying on gut feeling and a dozen open browser tabs.

It's built as a command center, not a chatbot bolted onto a dashboard. The map and the fleet view are the core experience; the AI layer (MATSAY OPS) is there to answer questions and surface patterns, backed by real data from the platform rather than guesses.

## How it works

MATSAY follows a fairly simple loop, repeated continuously for every vessel in the fleet:

1. **Observe** — real-time maritime and environmental data is ingested: AIS positions, weather and ocean conditions, port status, historical logs.
2. **Predict** — fuel consumption, ETA, weather exposure and operational risk are forecast for the current route and vessel state.
3. **Optimize** — the engine searches for better route, fleet and cargo allocation options against multiple objectives at once (fuel, cost, emissions, schedule).
4. **Simulate** — alternative scenarios are laid out side by side so they can be compared before anything is locked in.
5. **Approve** — the final call stays with the commander. MATSAY recommends; it doesn't act on its own.

That last point matters more than it might seem — the system is built around human-in-the-loop decisions, not automation that overrides the person in charge.

## What's in the platform

- **Fleet Dashboard** — live overview of every vessel, its status and key metrics
- **Live GIS Map** — vessel positions, routes, ports and conditions tracked in real time
- **Route Optimization** — compares route options against fuel, time, cost and weather exposure
- **Fuel & Emission Analytics** — consumption tracking against predicted baselines, with the factors behind any deviation
- **Cargo & Fleet Allocation** — matches cargo and routes to the right vessels
- **Weather & Ocean Integration** — wind, wave, current and sea-state data folded directly into planning
- **Risk & Alerts** — geofencing, route-deviation and weather alerts, surfaced before they become a problem
- **Scenario Simulation** — "what-if" comparisons across different operational choices
- **MATSAY OPS** — an operational assistant that answers fleet questions by pulling real data from the platform, not by making it up

## Tech stack

**Frontend**
Next.js · React · TypeScript · Tailwind CSS · Mapbox GL JS / Leaflet for GIS visualization

**Backend & core services**
FastAPI · PostgreSQL with PostGIS (spatial/geospatial queries) · Redis (caching, real-time data)

**Data & intelligence**
AIS/vessel data ingestion · weather & marine data APIs · port data · an LLM-backed assistant with tool calling for MATSAY OPS · optimization and ML services for route, fuel and risk analysis

**Integration**
HTTPS throughout, server-side secret management for provider API keys, structured geospatial data (zones, boundaries, restricted areas)

## Architecture, in brief

The system is organized in four layers, each one doing a specific job rather than one monolith trying to do everything:

1. **Data ingestion & normalization** — AIS data, weather/marine feeds, port data and geospatial data are cleaned, merged and standardized as they come in.
2. **Geospatial intelligence** — GIS mapping, spatial queries, geofencing and route intelligence sit on top of the normalized data.
3. **Intelligence & optimization** — fleet monitoring, fuel estimation, route optimization and risk/alert analysis happen here.
4. **Decision support & visualization** — MATSAY OPS, explainable recommendations, the live dashboard, GIS map and route comparison tools are what the commander actually interacts with.

Keeping data acquisition separate from decision intelligence makes the system easier to extend — new data sources or new optimization logic can be added without touching the layers above them.

## Getting started

```bash
git clone https://github.com/<your-username>/matsay.git
cd matsay
```

**Frontend**
```bash
cd frontend
npm install
npm run dev
```

**Backend**
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
```

You'll need a PostgreSQL instance with the PostGIS extension enabled, a Redis instance, and API keys for your weather/AIS data providers. Copy `.env.example` to `.env` in both the frontend and backend folders and fill in the relevant values before running either service.

## Live demo

[matsay.vercel.app](https://matsay.vercel.app)

## Roadmap

The current build is a working prototype covering the core command-center experience. Natural next steps include splitting data ingestion, intelligence and notification concerns into separate services as fleet size and data volume grow, broadening the data provider integrations, and refining the optimization engine's objective weighting based on real operator feedback.

## License

MIT
