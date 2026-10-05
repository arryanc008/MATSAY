# MATSAY — Product Overview

*Intelligent Maritime Fleet Command Center*

## The problem

Fleet operations teams deal with a lot of moving parts at once — vessel positions, weather windows, port schedules, fuel burn, cargo commitments — and in most setups, each of these lives in its own tool. AIS tracking is one screen. Weather forecasts are another. Port information gets checked separately. Fuel and cost numbers often end up in a spreadsheet that's updated by hand.

That fragmentation has a few predictable consequences. Situational awareness stays low because nobody has the full picture in one view. Route decisions get made manually, which is slow and leans heavily on individual experience rather than data. Alerts that matter get buried among ones that don't. And perhaps most importantly, raw data rarely gets turned into something a commander can act on quickly — there's a persistent gap between having data and being able to use it.

## The approach

MATSAY exists to close that gap. Rather than adding another dashboard to the pile, it pulls the relevant feeds — AIS/vessel data, weather and ocean conditions, port information, historical operational logs — into a single platform, normalizes them, and builds a decision layer on top.

That decision layer does three things: it predicts (fuel consumption, ETA, weather exposure, risk), it optimizes (route, fleet allocation and cargo assignment across multiple objectives simultaneously), and it lets a commander simulate and compare alternatives before anything is finalized. The actual decision — and the accountability that comes with it — stays with a qualified person. MATSAY's job is to make that decision faster and better informed, not to replace it.

## How it works

The platform runs on a repeating cycle:

**Observe.** Real-time data comes in from AIS feeds, weather and ocean APIs, port systems, satellite/geospatial sources and historical logs.

**Predict.** Based on that data, the system forecasts fuel consumption, arrival times, weather exposure and operational risk for each vessel and route under consideration.

**Optimize.** An optimization engine searches for better route, fleet and cargo allocation choices, balancing fuel use, cost, emissions and schedule constraints against each other rather than optimizing for just one metric.

**Simulate.** Instead of handing over a single "best" answer, MATSAY lays out comparable scenarios side by side, so a commander can see the trade-offs directly.

**Approve.** The commander reviews the recommendation, checks the reasoning behind it, and makes the final call. Everything downstream — the actual voyage, the fleet adjustment — only happens after that approval.

This loop runs continuously, so the picture a commander is working from stays current rather than going stale between manual check-ins.

## What a user actually sees

The command center itself is organized around a live GIS map showing vessel positions, routes, ports and conditions, paired with a fleet dashboard, a route-comparison view, fuel and emissions analytics, cargo and fleet allocation tools, weather and ocean data overlays, and an alerts panel for risks, route deviations and geofence events.

Sitting alongside all of that is MATSAY OPS — an assistant a commander can ask direct operational questions, like which vessel is burning more fuel than expected and why. It answers using live data pulled from the platform itself, rather than producing a plausible-sounding guess, and it's meant to be a shortcut into the data, not a replacement for the dashboard.

## Why it holds together technically

The system is split into four layers that each do one job: data ingestion and normalization, geospatial intelligence (GIS mapping, spatial queries, geofencing), the intelligence and optimization layer (fuel estimation, route optimization, risk analysis), and the decision-support/visualization layer that the commander actually interacts with. Separating these means a change to how weather data gets normalized, for instance, doesn't require touching the optimization logic or the dashboard — each piece can be developed, tested and extended on its own.

On the stack side, the frontend is built with Next.js, React, TypeScript and Tailwind, with Mapbox GL JS/Leaflet handling the GIS layer. The backend runs on FastAPI with PostgreSQL and PostGIS for spatial data, and Redis for caching and real-time state. Data flows in from AIS feeds, weather/marine APIs and port data sources over HTTPS, with provider credentials kept server-side.

## Who it's for, and what changes for them

**Fleet commanders** get faster decisions, a centralized view of the fleet, and recommendations they can actually interrogate rather than take on faith.

**Shipping and fleet operators** get better route planning and vessel utilization, without needing to manually reconcile five different data sources before making a call.

**Maritime operations teams** get centralized alerts and far less manual cross-checking between systems that don't talk to each other.

**Management and oversight functions** get fleet-level analytics and historical insight that supports planning decisions beyond any single voyage.

Taken together, the practical impact is fewer manual steps between "something changed" and "someone made an informed decision about it" — plus a route/fuel-aware planning process that supports lower unnecessary distance, time and fuel burn where the model's recommendations are followed.

## Where this goes next

The current build is a functioning prototype with real architecture behind it — not a mockup. The natural progression from here is splitting data ingestion, intelligence processing and notification handling into independent services as the data volume and fleet size grow, widening the set of data providers integrated, and tuning the optimization engine's weighting based on feedback from people who actually run fleets for a living. The modular structure was chosen specifically so that path doesn't require starting over.
