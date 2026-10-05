# MATSAY — Intelligent Maritime Fleet Command Center

Smart India Hackathon 2026 prototype by **Team Aqua-Agents**.

| | |
|---|---|
| Problem Statement ID | SIH26138 |
| Title | Quantum-Inspired Fuel Consumption Prediction and Green Fleet Optimization |
| Theme / Category | Smart Vehicles / Software |
| Team ID | 139377 |
| Live prototype | https://matsay.vercel.app |

MATSAY is a web command center for a shipping fleet. It puts vessel positions, weather, ports, alerts and fuel estimates on one screen, and helps a fleet commander compare route and speed options before approving them.

The commander always makes the final call. The system only recommends, and every approve / modify / reject action is written to an audit log.

> **Prototype note.** This is a hackathon prototype. Some parts run on live data and some run on a demo dataset. The section [What is real and what is simulated](#what-is-real-and-what-is-simulated) lists exactly which is which, so please read it before judging the numbers on screen.

---

## The problem

Fleet operators work with data that lives in too many places: AIS feeds, weather services, port information, and old operational logs. Because of this:

- nobody has a single live picture of fleet, routes and risks
- route and speed decisions are made by hand, and slowly
- important alerts get lost between routine ones
- fuel and emission impact is rarely compared before a decision is taken

## What MATSAY does

The workflow follows five steps:

1. **Observe**: collect vessel, weather and port data in one place
2. **Predict**: estimate fuel burn, CO2 and ETA for a voyage
3. **Optimize**: weigh fuel, cost, CO2, time, utilization and reliability
4. **Simulate**: compare "what if" scenarios (fuel price jump, speed limit, demand change)
5. **Approve**: the commander approves, modifies or rejects each recommendation

## Features

The app has 14 screens, all reachable from the sidebar.

| Screen | What it shows |
|---|---|
| Command Center | KPI cards, fleet summary, pending recommendations |
| Fleet Map | Leaflet map with vessels, ports, restricted zones, storm systems and OpenSeaMap seamarks |
| Vessels | Per-vessel details: specs, cargo, telemetry, fuel and CO2 |
| Routes | Planned route vs optimized route, with a short explanation of why the route changed |
| Optimization | Objective weights, constraints, run progress, Pareto-style option cards, recommendations |
| Fuel Intelligence | Voyage fuel / cost / CO2 prediction from speed, load, wind, waves and current |
| Weather & Ocean | Wind, waves, current and sea temperature near the selected vessel |
| Analytics | Fleet-level charts and comparisons |
| Cargo & Allocation | Cargo utilization and vessel allocation |
| Alerts | Critical / warning / info alerts, with acknowledge |
| Communications | Operational messages between commander and vessels / ports |
| Scenarios | Sliders for fuel price, cargo demand and speed restriction |
| Sustainability | Emission-focused view of the fleet |
| MATSAY Ops | Chat assistant for fleet questions (uses Gemini through the server) |

Human-in-the-loop approval is handled by a modal that records the commander's decision (`APPROVED`, `MODIFIED` or `REJECTED`) with notes into the audit log table.

## Architecture

```mermaid
flowchart LR
    subgraph Client[React + TypeScript, Vite]
      UI[Command Center UI]
      MAP[Leaflet GIS map]
    end

    subgraph Server[Express server.ts]
      API[REST API /api/*]
      AIS[AIS stream manager]
      MET[MetOcean service]
      FUEL[Fuel model]
    end

    DB[(PostgreSQL via Drizzle ORM)]
    OM[Open-Meteo Marine API]
    AISS[AISStream.io WebSocket]
    GEM[Google Gemini]

    UI --> API
    MAP --> API
    API --> DB
    MET --> OM
    MET --> DB
    AIS --> AISS
    API --> GEM
    UI --> FUEL
```

On Vercel the Express app is exported from `api/index.ts` and runs as a serverless function. The built frontend (`dist/`) is served as static files. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for more detail.

## Tech stack

| Layer | Used in this prototype |
|---|---|
| Frontend | React 19, TypeScript, Vite, Tailwind CSS 4, lucide-react, motion |
| Map | Leaflet with OpenStreetMap + OpenSeaMap tiles |
| Backend | Node.js, Express 4 |
| Database | PostgreSQL with Drizzle ORM (optional, see below) |
| Live data | Open-Meteo Marine API (weather/ocean), AISStream.io (AIS, needs a key) |
| AI assistant | Google Gemini through `@google/genai`, called only from the server |
| Auth | Firebase Auth packages are set up; the login screen currently uses a demo session (see limitations) |
| Hosting | Vercel |

## Project structure

```
.
├── api/index.ts            # Vercel entry, re-exports the Express app
├── server.ts               # Express app and all /api routes
├── vercel.json             # Rewrites /api/* to the function, everything else to the SPA
├── src/
│   ├── App.tsx             # Simple path-based routing
│   ├── context/            # MaritimeContext: app state, backend sync, login, approvals
│   ├── pages/              # One file per screen
│   ├── components/         # common/ (header, sidebar, modal), map/, vessels/, command/, ops/
│   ├── services/
│   │   ├── fuelModel.ts            # Fuel / CO2 prediction
│   │   ├── optimizationEngine.ts   # Optimization run (see limitations)
│   │   ├── metoceanService.ts      # Open-Meteo fetch + DB cache
│   │   ├── aisStreamService.ts     # AISStream WebSocket client
│   │   └── vesselDatabaseService.ts# DB reads/writes and first-run seeding
│   ├── data/mockMaritimeData.ts    # Demo fleet, ports, zones, storms, alerts
│   ├── db/                 # Drizzle schema and connection
│   ├── middleware/auth.ts  # Firebase token check (not applied to routes yet)
│   ├── lib/                # Firebase client / admin setup
│   └── types/maritime.ts
└── docs/                   # Architecture, API, fuel model, demo guide, roadmap
```

## Getting started

**You need:** Node.js 20 or newer. PostgreSQL is optional.

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
npm install

cp .env.example .env     # fill in only what you have, everything is optional
npm run dev
```

Open http://localhost:3000.

In development, `server.ts` starts Express and mounts Vite as middleware, so one command runs both frontend and backend.

### Environment variables

All variables are optional. The app starts without any of them.

| Variable | What it enables | If missing |
|---|---|---|
| `DATABASE_URL` | PostgreSQL storage for vessels, ports, alerts, messages, audit logs, weather cache | Server uses the built-in demo dataset; writes (acks, messages, audit entries) are not persisted |
| `AISSTREAM_API_KEY` | Live AIS positions from AISStream.io | `/api/ais` reports "unavailable"; the app keeps using the demo fleet |
| `GEMINI_API_KEY` | MATSAY Ops chat answers from Gemini | Assistant falls back to a built-in canned reply |
| `SQL_HOST`, `SQL_DB_NAME`, `SQL_USER`, `SQL_PASSWORD` | Alternative to `DATABASE_URL` (Cloud SQL style) | Ignored |
| `SQL_ADMIN_USER`, `SQL_ADMIN_PASSWORD` | Only used by `drizzle-kit` to create tables | Not needed to run the app |
| `PORT` | Server port | `3000` |

Never commit your `.env`. It is already in `.gitignore`.

### Database setup (optional)

Create an empty PostgreSQL database first. The app itself connects with `DATABASE_URL`, but `drizzle-kit` (used to create the tables) reads the separate `SQL_*` variables, so set both in `.env`:

```bash
DATABASE_URL=postgres://user:pass@localhost:5432/matsay

SQL_HOST=localhost
SQL_DB_NAME=matsay
SQL_ADMIN_USER=user
SQL_ADMIN_PASSWORD=pass
```

Then create the tables and start the app:

```bash
npx drizzle-kit push --config src/db/drizzle.config.ts
npm run dev          # demo rows are inserted on first start if the vessels table is empty
```

### Build and deploy

```bash
npm run build        # outputs dist/
```

The repo is set up for Vercel (`vercel.json`). Add the environment variables in the Vercel project settings.

## Demo login

The login screen has a **Demo Commander Session** button. It fills in:

- Email: `commander.mercer@matsay.maritime`
- Password: `Admiralty#2026`

The login is a front-end demo. Any email with a password of 4+ characters is accepted. See limitations.

A suggested walkthrough for judges is in [docs/DEMO_GUIDE.md](docs/DEMO_GUIDE.md).

## Fuel and emission model

`src/services/fuelModel.ts` is a physics-based estimate, not a lookup table. In short:

1. Calm-water shaft power from the Admiralty formula: displacement^(2/3) x speed^3 / coefficient
2. Extra resistance added for headwind and wave height
3. Speed over ground adjusted for ocean current
4. Fuel per day = power x SFOC x 24 h
5. CO2 = fuel x fuel-specific emission factor (VLSFO, MGO, LNG, methanol, ammonia)

The formulas and constants are written out in [docs/FUEL_MODEL.md](docs/FUEL_MODEL.md). The coefficients are generic values and have not been calibrated against real vessel noon reports.

## What is real and what is simulated

| Part | Status |
|---|---|
| Weather / wave / wind / current | **Live** from Open-Meteo, cached in Postgres for 30 minutes when a database is configured. If the API call fails, the service returns default values, so check the `isLive` flag in the response |
| AIS positions | **Live only if** `AISSTREAM_API_KEY` is set. Otherwise the demo fleet is used |
| Fleet (6 vessels), ports (7), restricted zones (4), storms (2), alerts, messages | **Demo dataset** in `src/data/mockMaritimeData.ts` |
| Fuel / CO2 prediction | **Computed** by the formula model above |
| Optimization run | **Simulated.** See next section |
| Route comparison, recommendations | **Pre-written demo scenarios** tied to the demo fleet |
| Alerts acknowledge, messages, commander decisions | **Saved** to PostgreSQL when `DATABASE_URL` is set |
| MATSAY Ops chat | **Live Gemini** when `GEMINI_API_KEY` is set, otherwise a fallback reply |
| Login | **Demo session** stored in browser localStorage |

### About the "quantum-inspired" optimizer

The problem statement asks for a quantum-inspired approach. In this prototype the Optimization screen demonstrates the workflow and the multi-objective inputs (six weights and eight constraints), but the run itself is **not yet a real quantum-inspired search**. The progress phases are timed steps for the demo, and the result is derived from the chosen weights together with fixed example solutions. Savings numbers shown there are illustrative and are not measured results.

The fuel model above is real and is the piece a proper optimizer would call. Building the actual quantum-inspired evolutionary search on top of it is the main item on the [roadmap](docs/ROADMAP.md).

## Limitations

- Demo data for the fleet; live AIS needs an API key and a subscription tier that covers the region
- Optimization is simulated (see above)
- Login is front-end only. Firebase is configured in the project, but the UI does not use it yet, and the API routes are not protected by `requireAuth`
- No automated tests yet
- Route geometry is hand-defined waypoints, not computed by a routing algorithm
- Single-user: no roles, no multi-tenant separation

## Roadmap

Short version, full list in [docs/ROADMAP.md](docs/ROADMAP.md):

1. Real quantum-inspired optimizer (Q-bit encoding + rotation-gate updates) using the fuel model as the objective
2. Weather-aware routing on a sea-lane graph
3. Wire Firebase Auth into login and protect write endpoints
4. Move spatial queries to PostGIS, add Redis for live-data caching
5. Calibrate the fuel model with real noon-report data
6. Tests for the fuel model and API

## Team Aqua-Agents

| Name | Role |
|---|---|
| _Name 1_ | _Role_ |
| _Name 2_ | _Role_ |
| _Name 3_ | _Role_ |
| _Name 4_ | _Role_ |
| _Name 5_ | _Role_ |
| _Name 6_ | _Role_ |

## Data sources and credits

- Weather and marine data: [Open-Meteo](https://open-meteo.com/)
- AIS: [AISStream.io](https://aisstream.io/)
- Map tiles: [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors, [OpenSeaMap](https://www.openseamap.org/)
- Map library: [Leaflet](https://leafletjs.com/)

## License

Add a license of your choice here (MIT is common for hackathon projects).
