# 🚕 GoRide — Real-Time Ride-Sharing Android App (Uber/Lyft Architecture Clone)

**A production-grade Android application replicating the core Uber/Lyft rider experience** — built to showcase senior-level Android engineering: clean layered architecture, live map/location systems, WebSocket-driven real-time state, and buttery-smooth trip animations.

<p align="center">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/banner-ridesharing-uber-lyft-app.jpg">
</p>

<p align="center">
  <img alt="Kotlin" src="https://img.shields.io/badge/Kotlin-100%25-7F52FF?logo=kotlin&logoColor=white">
  <img alt="Architecture" src="https://img.shields.io/badge/Architecture-MVP-blue">
  <img alt="Min SDK" src="https://img.shields.io/badge/minSdk-23-green">
  <img alt="License" src="https://img.shields.io/badge/License-Apache%202.0-lightgrey">
</p>

---

## 📌 Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [System Overview](#2-system-overview)
3. [Architecture](#3-architecture)
4. [Real-Time Communication Layer](#4-real-time-communication-layer)
5. [State Machine — Trip Lifecycle](#5-state-machine--trip-lifecycle)
6. [Tech Stack](#6-tech-stack)
7. [Project Structure](#7-project-structure)
8. [Functional Requirements](#8-functional-requirements)
9. [Non-Functional Requirements](#9-non-functional-requirements)
10. [Getting Started](#10-getting-started)
11. [WebSocket API Reference](#11-websocket-api-reference)
12. [Engineering Highlights](#12-engineering-highlights-what-makes-this-non-trivial)
13. [Screenshots](#13-screenshots)
14. [Roadmap](#14-roadmap)
15. [License](#15-license)

---

## 1. Problem Statement

Ride-sharing is one of the hardest consumer mobile categories to build correctly, because a single screen has to simultaneously handle:

- **Live location streaming** for both rider and driver
- **Map rendering** with route polylines, camera control, and marker animation
- **Asynchronous, bidirectional networking** (server can push state changes at any time)
- **A strict multi-stage state machine** (search → book → pickup → arrival → trip → drop-off) that must never desync from what the user sees on screen
- **Failure handling** for network drops, no-route-found, and reconnection — without corrupting trip state

GoRide reconstructs this entire flow end-to-end, including a **simulated WebSocket backend**, so the full rider journey can be built, tested, and demoed without depending on a live production server.

---

## 2. System Overview

```
┌──────────────┐        WebSocket (JSON events)        ┌───────────────────┐
│   Android     │ ─────────────────────────────────────▶ │  Simulated Server │
│   Client      │ ◀───────────────────────────────────── │  (simulator mod.) │
└──────────────┘                                        └───────────────────┘
       │
       ├── Google Maps SDK  (map rendering, markers, camera)
       ├── Directions API   (route polylines)
       └── Places API       (pickup / drop-off search)
```

The client never assumes server state — every UI transition is **driven by an event from the WebSocket layer**, making the app resilient to timing issues and easy to reason about.

---

## 3. Architecture

GoRide uses a deliberately simple, highly-readable **MVP (Model–View–Presenter)** pattern instead of a heavier framework — prioritizing clarity of each feature so the codebase stays approachable for review, extension, and interview walkthroughs.

```
┌─────────────────────┐      user actions      ┌─────────────────────┐
│        View          │ ───────────────────▶  │     Presenter        │
│ (Activity/Fragment)  │                        │  (business logic)    │
│                       │ ◀───────────────────  │                       │
└─────────────────────┘     state updates      └──────────┬──────────┘
                                                            │
                                                 ┌──────────▼──────────┐
                                                 │   Model Layer        │
                                                 │  WebSocket client +  │
                                                 │  simulated backend   │
                                                 └───────────────────────┘
```

| Layer | Responsibility | Depends On |
|---|---|---|
| **View** | Renders UI state only — map markers, bottom sheets, buttons, marker animation | Presenter (via interface) |
| **Presenter** | Owns all business logic; decides what the View shows next based on events | Model layer (via interface), never Android framework directly |
| **Model / WebSocket Layer** | Simulated real-time backend; emits ride lifecycle events over a WebSocket abstraction | Nothing (pure data layer) |

**Why MVP over MVVM here:** the View–Presenter contract is defined through plain interfaces, which keeps every screen's logic unit-testable without Android instrumentation, and keeps the mental model of "one presenter owns one trip's state machine" explicit rather than implicit in a ViewModel + LiveData chain.

---

## 4. Real-Time Communication Layer

Rather than polling, GoRide's `simulator` module exposes a **WebSocket abstraction** so the client is built exactly the way it would be against a production ride-sharing backend:

- `connect()` / `disconnect()` manage the connection lifecycle explicitly, so reconnection and cleanup are first-class, not an afterthought
- All server events arrive as `onMessage(data: String)` JSON payloads, decoded into typed sealed events on the client
- `onError(error: String)` is a distinct channel from normal state events, so network failures (`directionApiFailed`, `routesNotAvailable`) are handled without corrupting the trip state machine

This mirrors how a real rider app talks to services like a location/dispatch server — location pushes, trip-state pushes, and error channels are all separate concerns.

---

## 5. State Machine — Trip Lifecycle

```
   ┌───────────┐   nearByCabs   ┌───────────┐  requestCab   ┌────────────┐
   │ Discovery │ ─────────────▶ │  Booking  │ ─────────────▶│ cabBooked  │
   └───────────┘                └───────────┘               └─────┬──────┘
                                                                   │ pickUpPath / location
                                                                   ▼
                                                          ┌──────────────────┐
                                                          │ cabIsArriving /  │
                                                          │   cabArrived     │
                                                          └────────┬─────────┘
                                                                   │ tripStart
                                                                   ▼
                                                          ┌──────────────────┐
                                                          │  tripPath /      │
                                                          │  location (live) │
                                                          └────────┬─────────┘
                                                                   │ tripEnd
                                                                   ▼
                                                          ┌──────────────────┐
                                                          │  Take Next Ride  │
                                                          └──────────────────┘
```

Every transition above corresponds 1:1 to a server-pushed event — the client never infers state from a timer or an assumption, only from an explicit message.

---

## 6. Tech Stack

| Category | Choice |
|---|---|
| Language | Kotlin |
| Architecture | MVP (View ↔ Presenter ↔ Model) |
| Maps & Location | Google Maps SDK for Android, Directions API, Places API |
| Real-time layer | Custom-simulated WebSocket module (`simulator`) |
| Animation | Custom marker interpolation for smooth, Uber-style car movement along polylines |
| Build | Gradle (Kotlin DSL) |

---

## 7. Project Structure

```
ridesharing-uber-lyft-app/
├── app/
│   └── src/main/java/.../
│       ├── ui/                 # Activities/Fragments — View layer
│       │   ├── maps/           # Map rendering, marker animation
│       │   └── booking/        # Pickup/drop-off selection UI
│       ├── presenter/          # Presenter layer — per-screen business logic
│       ├── model/              # Data models (Cab, Trip, LatLng events, etc.)
│       └── simulator/          # Simulated WebSocket server + client abstraction
├── assets/                     # Screenshots / demo media
├── local.properties            # SDK path + API key (not committed)
└── build.gradle.kts
```

---

## 8. Functional Requirements

- [ ] Request and display runtime **location permission**
- [ ] Fetch and render **nearby cabs** on Google Maps
- [ ] Let the rider select **pickup and drop-off** locations via Places autocomplete
- [ ] Send a **cab request** and receive a booking confirmation
- [ ] Draw and animate the **pickup route** as the driver approaches
- [ ] Stream the **driver's live location** and animate the car marker smoothly (not jumpy per-update snapping)
- [ ] Reflect **cab-is-arriving → cab-arrived** states in the UI
- [ ] Start a **trip**, draw the trip route, and stream live location during the trip
- [ ] Handle **trip completion** and allow the rider to immediately request another ride
- [ ] Gracefully surface **errors** (no route found, directions API failure) without crashing the state machine

## 9. Non-Functional Requirements

| Requirement | Target |
|---|---|
| **Location update smoothness** | Marker interpolation, not raw snapping, even at 1–2s update intervals |
| **State consistency** | UI state is always derived from the latest server event — no client-side guessing |
| **Resilience** | WebSocket disconnects are recoverable without losing trip context |
| **Testability** | Presenter layer is unit-testable independent of Android framework classes |
| **Cold start** | App reaches an interactive map within a few seconds on a mid-tier device |
| **Extensibility** | New trip states / events can be added without touching unrelated screens |

---

### Step 2 — Get a Google API Key
This app needs **Maps SDK for Android**, **Directions API**, and **Places API** enabled.
1. Open the [Google Cloud Console](https://console.cloud.google.com/)
2. Enable the three APIs above
3. Generate a key ([official guide](https://developers.google.com/maps/documentation/directions/get-api-key))

### Step 3 — Configure `local.properties`
```properties
sdk.dir=PATH_TO_ANDROID_SDK_ON_YOUR_LOCAL_MACHINE
apiKey=YOUR_API_KEY
```

### Step 4 — Build incrementally (recommended if studying the code)
1. Base MVP architecture scaffold
2. Runtime location permission handling
3. Nearby cabs via the `simulator` WebSocket
4. Pickup & drop-off location selection (Places)
5. Book-a-cab flow
6. Pickup path drawing + animation
7. Live driver location streaming during pickup
8. `cabIsArriving` → `cabArrived` handling
9. Smooth car marker animation (Uber-style interpolation)
10. Trip path drawing + animation
11. `tripStart` → ongoing → `tripEnd` handling
12. "Take next ride" loop

### Step 5 — Run
Build and run on an emulator or physical device with Google Play Services installed.

---

## 11. WebSocket API Reference

### Connection Methods
| Method | Purpose |
|---|---|
| `connect()` | Opens the connection to the server |
| `sendMessage(data: String)` | Sends a JSON payload to the server |
| `disconnect()` | Closes the connection |

### Listener Callbacks
| Callback | Fires When |
|---|---|
| `onConnect()` | Connection established |
| `onMessage(data: String)` | Server pushes an event |
| `onDisconnect()` | Connection closed |
| `onError(error: String)` | Something fails server-side |

### Client → Server

**Request nearby cabs**
```json
{ "type": "nearByCabs", "lat": 28.438147, "lng": 77.0994446 }
```

**Request a cab**
```json
{
  "type": "requestCab",
  "pickUpLat": 28.4369353,
  "pickUpLng": 77.1125599,
  "dropLat": -25.274398,
  "dropLng": 133.775136
}
```

### Server → Client

| Event Type | Payload Highlights |
|---|---|
| `nearByCabs` | Array of `{lat, lng}` cab locations |
| `cabBooked` | Booking confirmation |
| `pickUpPath` | Array of `{lat, lng}` route points |
| `location` | Live `{lat, lng}` of the cab |
| `cabIsArriving` | Cab is close to pickup point |
| `cabArrived` | Cab has arrived |
| `tripStart` | Trip has begun |
| `tripPath` | Array of `{lat, lng}` route points |
| `tripEnd` | Trip is complete |

### Error Events (`onError`)

| Error Type | Meaning |
|---|---|
| `directionApiFailed` | Directions API call failed (network/DNS issue) |
| `routesNotAvailable` | No route found between the two points |

---

## 12. Engineering Highlights (what makes this non-trivial)

- **Deterministic state machine** driven entirely by server events — no polling, no client-side timers guessing trip phase.
- **Smooth marker interpolation** between raw location pushes, avoiding the "jumping car" problem common in naive implementations.
- **Decoupled Presenter layer** that has zero Android framework dependencies, enabling fast unit tests of trip logic.
- **Explicit error channel** separate from state events, so transient network failures don't corrupt the UI's understanding of trip state.
- **Simulated backend** that mirrors a real dispatch server's contract, so the exact same client code is ready to point at a production WebSocket service.

---

## 13. Screenshots

<p align="center">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/nearby-cabs.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/pickup-drop-location.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/pickup-drop-location-both-filled.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/request-cab-button.png" width="200">
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/cab-is-booked.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/cab-is-arriving.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/on-trip.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/trip-end.png" width="200">
</p>

---

## 14. Roadmap

- [ ] Driver-side app (currently rider-only)
- [ ] Fare estimation before booking
- [ ] Multiple vehicle tiers (economy / premium / pool)
- [ ] Persisted trip history
- [ ] Push notifications for trip state changes
- [ ] Migrate simulated backend to a real WebSocket server for a full-stack demo

---

## 15. License

```
Copyright (C) 2024 DK

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

---

<p align="center">If this project helped you, consider giving it a ⭐ — it helps others find it too.</p>
