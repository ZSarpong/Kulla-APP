# Kulla

Kulla is an independent, hyperlocal product-discovery MVP for Accra Central's physical markets. This repository contains a FastAPI/PostgreSQL API and a native Kotlin/Compose Android client. Every seeded shop and contact is explicitly fictional development data.

## What works

The implemented vertical slice is `search → result → product → vendor → map/contact/directions`. The API provides ranked exact, partial, alias, category, vendor-name, and typo-tolerant search with market, price, availability, verification, distance, and radius filters. Android uses a repository, Retrofit, Room caching, immutable ViewModel state, and safe dial/WhatsApp/geo intents. It never sends a message automatically.

Vendor registration, inventory upsert, verification submission, saves, search outcomes, and incorrect-information reports also have functional endpoints. The broader vendor, agent, and admin surfaces remain roadmap work.

## Local backend

Requirements: Python 3.12+ and optionally Docker Desktop.

```powershell
Copy-Item .env.example .env
docker compose up -d db
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r backend/requirements.txt
Set-Location backend
alembic upgrade head
python -m app.scripts.seed
uvicorn app.main:app --reload
```

Open `http://localhost:8000/docs` for OpenAPI or `http://localhost:8000/health` for health. For a dependency-free demo, omit `KULLA_DATABASE_URL` to use SQLite and run the seed module before Uvicorn.

Tests and quality checks:

```powershell
pytest
ruff check backend
ruff format --check backend
```

## Android

Open `android/` in Android Studio, use JDK 17, install Android SDK 35, and sync Gradle. Start the backend first; the emulator reaches it at `10.0.2.2:8000`. Then run the `app` debug configuration.

```powershell
Set-Location android
.\gradlew.bat assembleDebug
.\gradlew.bat testDebugUnitTest
.\gradlew.bat lintDebug
```

The client distinguishes cached search results, keeps saves/sync operations in Room, and falls back to Android geo intents for routing. The development phone `+233000000000` is deliberately invalid.

## Data, maps, and attribution

Reference points are supplied product coordinates, not surveyed boundaries. Initial market polygons are provisional rectangles stored as editable GeoJSON; administrators must replace these with verified shapes. Any production map must display attribution for OpenStreetMap contributors and the chosen tile provider. Do not use public OSM raster tiles as an unbounded production tile service.

PostgreSQL migrations enable PostGIS, pgvector, and `pg_trgm`. The MVP stores portable latitude/longitude columns for its executable SQLite test path; production follow-up should add generated PostGIS `geography(Point,4326)` columns and GiST indexes before scale testing.

The deterministic image provider in `backend/app/search/service.py` produces a stable 16-value test vector. Replace it by implementing `ImageEmbeddingProvider`, persisting the model/version with each vector, backfilling embeddings, and switching ranking behind configuration. No search caller needs to change.

## Security assumptions and limitations

The `X-Kulla-Role` and `X-Buyer-Id` headers are development-only identity seams, not production authentication. Before deployment add password hashing, signed short-lived access tokens, refresh rotation, ownership-scoped vendor authorization, upload validation/storage, rate limiting, secret management, and encrypted Android token storage. Server-side role checks already demonstrate the authorization boundary; vendors cannot submit writes with a buyer role.

Current known limitations: no live image upload, map rendering is a placeholder surface with MapLibre dependency prepared, no production route service, no audited verification-event tables yet, and no full vendor/agent/admin Android flows. Offline verification capture and conflict resolution are represented by the Room sync queue but not fully surfaced.

## Roadmap

1. Finish normalized verification/audit models and ownership-scoped authentication.
2. Add PostGIS geography/polygon columns, point-in-polygon and bounding-box queries.
3. Render MapLibre zones, clustering, markers, and offline tile-region management.
4. Complete vendor registration and field-agent offline capture/sync screens.
5. Add CameraX/photo picker, object storage, embedding workers, and pgvector ranking.
6. Add integration/Compose tests and deploy observability.

See [docs/architecture.md](docs/architecture.md) for boundaries and tradeoffs.

