# Gasoil Around Me — project instructions

## Product goal

Build a French Android application that helps a user find nearby fuel stations and compare prices. The first release is a free, ad-free prototype for France only.

The primary screen should be inspired by the two Essence&CO reference screenshots supplied by the user:

- an interactive map in the background;
- station markers synchronized with the station list;
- fuel and sorting controls above the list;
- a draggable Material 3 bottom sheet showing station, city, straight-line distance, selected-fuel price, and price freshness;
- a station detail view with all available prices, address, hours/services when available, and a navigation action.

Use the screenshots as UX inspiration, not as assets to copy. Avoid intrusive advertising and unnecessary visual clutter.

## Confirmed MVP scope

- Android only, written in Kotlin with Jetpack Compose and Material 3.
- Minimum SDK 26; preserve the existing application ID `com.francotte.gasoilaroundme`.
- Google Maps SDK for Android for map rendering.
- Foreground location only. Never request background location.
- If location is unavailable or refused, offer city/address search.
- Fetch fuel stations and prices directly from the official French government API; no backend in the prototype.
- Calculate distances locally as straight-line distances (Haversine). Do not call a route matrix for list items.
- Open an installed navigation app through an Android intent for directions to a selected station.
- No login, account, analytics, advertising, tracking, payment, or push notifications in the MVP.

## Fuel taxonomy

Use these user-facing choices and map them explicitly to the official data fields:

- Gazole
- SP95-E5
- E10
- SP98-E5
- E85
- GPL

Do not collapse SP95 and SP98 into a generic `E5` choice. Persist the selected fuel locally if doing so does not require adding unnecessary infrastructure.

## Authoritative external services

### Fuel stations and prices

Use the official dataset `prix-des-carburants-en-france-flux-instantane-v2` from `data.economie.gouv.fr`, through the Opendatasoft Explore API v2.1.

Dataset information:

`https://data.economie.gouv.fr/explore/dataset/prix-des-carburants-en-france-flux-instantane-v2/`

Records endpoint base:

`https://data.economie.gouv.fr/api/explore/v2.1/catalog/datasets/prix-des-carburants-en-france-flux-instantane-v2/records`

Treat the upstream schema as untrusted network input:

- tolerate missing/null prices, coordinates, update timestamps, hours, and services;
- exclude invalid coordinates;
- preserve the station's official identifier;
- represent temporary/permanent fuel shortages explicitly;
- display price freshness and never imply that an old price is current;
- show a sensible station label based on available official data rather than inventing a brand.

Do not scrape commercial fuel-price sites and do not add another station-price source without an explicit product decision.

### Address and city search

Use the current official French Geoplateforme/IGN geocoding service. Do not use the retired `api-adresse.data.gouv.fr` endpoint, which was scheduled for decommissioning in January 2026.

Debounce typed searches, require a useful minimum query length, limit results to France, and avoid a network request for every keystroke.

### Map

Use only `Maps SDK for Android` for this prototype. Do not enable or integrate Google Places, Routes, Directions, Distance Matrix, or Google Geocoding unless the user explicitly expands the scope.

## Secrets and Google Maps security

The user stores the development key in root `local.properties` as:

`MAPS_API_KEY=...`

Requirements:

- never read, print, log, copy, expose, or commit the key;
- never place the literal key in Kotlin, XML, Gradle files, tests, screenshots, or documentation;
- inject it into the manifest with the Google Maps Platform Secrets Gradle Plugin;
- use the manifest metadata name `com.google.android.geo.API_KEY`;
- keep `local.properties` ignored;
- assume the Cloud key is restricted to package `com.francotte.gasoilaroundme`, the relevant signing SHA-1, and Maps SDK for Android only.

Use separate properly restricted credentials for release signing when publishing. The debug SHA-1 must not be treated as the Play production signing certificate.

## Location and privacy behavior

- Ask for location permission in context, not immediately without explanation.
- Support both approximate and precise foreground location.
- The app must remain useful after denial by presenting manual search.
- Do not repeatedly prompt after denial and handle permanent denial with a settings action only when useful.
- Do not send the user's location anywhere except as coordinates required for the direct station query; do not persist or profile location history.
- Do not request contacts, phone state, advertising ID, background location, or unrelated permissions.
- Provide clear loading, empty, offline, permission-denied, and upstream-error states.

## Architecture guidelines

Keep the prototype simple but maintain clear boundaries so a backend can replace the direct API later:

- `data`: API DTOs, remote data sources, parsing, repository implementations;
- `domain`: station/fuel models, repository contracts, Haversine calculation, filtering and sorting;
- `ui`: screens/components, immutable UI state, ViewModels;
- `location`: a small platform location abstraction;
- `navigation`: external navigation-intent helper.

Prefer unidirectional data flow. Composables should render state and emit events; networking, parsing, filtering, location work, and sorting do not belong directly in composables.

Use coroutines with structured concurrency. Cancel obsolete address searches and station loads. Avoid unnecessary abstractions, dependency-injection frameworks, databases, and modules until the prototype needs them. Dependencies should be centralized in `gradle/libs.versions.toml`.

The repository boundary should expose a stable app model rather than leaking Opendatasoft DTOs into the UI. This boundary is the future seam for a backend migration.

## UI and interaction requirements

- Design for edge-to-edge layouts and system insets.
- Keep map attribution and Google branding unobscured.
- Ensure the bottom sheet does not permanently cover essential map controls.
- Fuel selection and sort selection must be obvious and accessible.
- Default sort: lowest valid price for the selected fuel.
- Alternative sort: nearest by straight-line distance.
- Clearly distinguish missing price, reported shortage, stale price, and network failure.
- Selecting a marker should reveal/select its station row; selecting a row should focus its marker.
- Do not require or fabricate commercial station logos for MVP.
- Use French copy and French number formatting, for example `1,789 €/L` and `3,2 km`.
- Provide content descriptions and touch targets suitable for accessibility.

## Networking and resilience

- Declare only the normal Internet permission plus required foreground location permissions.
- Use HTTPS exclusively.
- Apply reasonable connect/read timeouts.
- Limit API fields and records where the official API permits it.
- Do not load all of France on every camera movement.
- Debounce map-driven refreshes and avoid duplicate concurrent requests.
- Preserve the last successful in-memory result during transient refresh failures when practical.
- API failures must produce a recoverable UI with a retry action, not a crash.

## Testing expectations

At minimum, add deterministic unit tests for:

- official fuel-field mapping;
- nullable/malformed API parsing;
- Haversine distance calculation;
- radius filtering;
- price and distance sorting;
- shortage and missing-price handling;
- stale request cancellation or result replacement where relevant.

Add focused Compose/UI tests for important state transitions when practical. Do not make unit tests depend on live government, IGN, or Google services. Keep parsing fixtures small and representative.

Before handing off changes, run the narrowest relevant checks and then, when possible on Windows:

```powershell
.\gradlew testDebugUnitTest
.\gradlew assembleDebug
```

Never print `local.properties` as part of diagnostics or test output.

## Definition of done for the first usable prototype

The prototype is usable when a fresh install can:

1. explain and request foreground location;
2. center the map from an accepted approximate or precise location;
3. fall back to a French city/address search after refusal or location failure;
4. retrieve nearby official stations for the chosen point;
5. switch among all six fuel choices;
6. show markers and a synchronized bottom-sheet list;
7. sort valid results by price or straight-line distance;
8. communicate missing, stale, or unavailable prices honestly;
9. open external navigation for a station;
10. recover gracefully from offline/API errors without exposing secrets.

## Out of scope unless explicitly requested

- backend, scheduled ingestion, database, or station-data aggregation;
- authentication and user accounts;
- favorites synchronized across devices;
- price alerts or notifications;
- historical price charts;
- crowdsourced price verification;
- ads, subscriptions, purchases, or analytics;
- background location or trip monitoring;
- route duration/distance for every station;
- iOS, web, or markets outside France.

If a future request conflicts with these constraints, call out the product or cost implication before changing the architecture.
