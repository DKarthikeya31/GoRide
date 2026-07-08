# GoRide — Android Ride-Sharing App (Uber/Lyft Clone)

An end-to-end, production-style ride-sharing Android application that replicates the core rider experience of apps like **Uber** and **Lyft** — built to demonstrate real-world Android engineering: clean architecture, live map interactions, WebSocket-driven real-time updates, and smooth trip-state animations.

<p align="center">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/banner-ridesharing-uber-lyft-app.jpg">
</p>

---

## 1. Why This Project Matters

Ride-sharing apps are one of the hardest consumer product categories to get right on mobile: they combine live location tracking, map rendering, asynchronous networking, and a multi-stage state machine (search → book → pickup → trip → drop-off) that all has to feel instant and reliable to the user.

This project rebuilds that entire flow from scratch, including a **simulated backend and WebSocket server**, so the full rider journey can be developed, tested, and demoed without depending on a live production backend.

---

## 2. What the App Does

| Stage | What Happens |
|---|---|
| **Discovery** | Rider opens the app, location permission is requested, nearby cabs are fetched and rendered on Google Maps |
| **Booking** | Rider sets pickup and drop-off locations, requests a cab |
| **Pickup** | Driver's live location streams in, pickup path is drawn and animated on the map |
| **Arrival** | App reflects "cab is arriving" → "cab arrived" states |
| **Trip** | Trip starts, live driver location + trip path animate turn-by-turn |
| **Completion** | Trip ends; rider can immediately request another ride |

---

## 3. Architecture

The app is intentionally built on a **simple, readable MVP (Model–View–Presenter)** architecture — prioritizing clarity of each feature over framework complexity, so the code stays approachable for anyone studying it.

```
View (Activity/Fragment)  ←→  Presenter  ←→  Model / WebSocket Layer
```

- **View** — renders UI state (map markers, bottom sheets, buttons, animations)
- **Presenter** — owns business logic, decides what the View should show next
- **Model / WebSocket Layer** — simulated real-time backend that mimics an actual ride-sharing server

---

## 4. Tech Stack

- **Language:** Kotlin
- **Maps & Location:** Google Maps SDK, Directions API, Places API
- **Real-time layer:** Custom-simulated WebSocket module (`simulator`)
- **Architecture:** MVP
- **Animation:** Custom marker interpolation for smooth car movement (Uber-style)

---

## 5. Screenshots

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

## 6. Getting Started — Step by Step

### Step 1: Clone the repository
```bash
git clone https://github.com/amitshekhariitbhu/ridesharing-uber-lyft-app.git
```
The `master` branch always has the latest, working code.

### Step 2: Get a Google API Key
This app needs Maps, Directions, and Places APIs enabled.
1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Enable **Maps SDK for Android**, **Directions API**, and **Places API**
3. Generate an API key ([official guide](https://developers.google.com/maps/documentation/directions/get-api-key))

### Step 3: Configure `local.properties`
Add the SDK path and your API key:
```
sdk.dir=PATH_TO_ANDROID_SDK_ON_YOUR_LOCAL_MACHINE
apiKey=YOUR_API_KEY
```

### Step 4: Build the project incrementally
If you're using this as a learning reference, build it feature-by-feature rather than all at once:

1. Start a new Android project
2. Set up the base MVP architecture
3. Implement runtime location permission handling
4. Implement **nearby cabs** using the `simulator` module's WebSocket
5. Implement pickup & drop-off location selection
6. Implement the **book a cab** flow
7. Draw and animate the **pickup path** on the map
8. Stream and render the **driver's live location** during pickup
9. Handle **cab is arriving** → **cab arrived** states
10. Add Uber-style **smooth car marker animation**
11. Draw and animate the **trip path**
12. Handle **trip start** → **trip on-going** → **trip end**
13. Implement **"take next ride"** to loop the flow

### Step 5: Run it
Build and run on an emulator or physical device with Google Play Services installed.

---

## 7. WebSocket API Reference

The app talks to a **simulated backend** over a WebSocket abstraction, so the real-time behavior of a production ride-sharing service can be developed against locally.

### Core WebSocket Methods
| Method | Purpose |
|---|---|
| `connect()` | Opens the connection to the server |
| `sendMessage(data: String)` | Sends a JSON payload to the server |
| `disconnect()` | Closes the connection |

### Core WebSocket Listener Callbacks
| Callback | Fires When |
|---|---|
| `onConnect()` | Connection is established |
| `onMessage(data: String)` | Server pushes an event |
| `onDisconnect()` | Connection is closed |
| `onError(error: String)` | Something goes wrong server-side |

### Client → Server Messages

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

### Server → Client Messages

| Event Type | Payload Highlights |
|---|---|
| `nearByCabs` | Array of `{lat, lng}` cab locations |
| `cabBooked` | Confirms booking |
| `pickUpPath` | Array of `{lat, lng}` points for the pickup route |
| `location` | Live `{lat, lng}` of the cab during pickup/trip |
| `cabIsArriving` | Cab is close to pickup point |
| `cabArrived` | Cab has arrived |
| `tripStart` | Trip has begun |
| `tripPath` | Array of `{lat, lng}` points for the trip route |
| `tripEnd` | Trip is complete |

### Error Events (via `onError`)

| Error Type | Meaning |
|---|---|
| `directionApiFailed` | Directions API call failed (e.g., network/DNS issue) |
| `routesNotAvailable` | No route could be found between the two points |

---

## 8. Support This Project

If this project helped you learn something, consider giving it a ⭐ on GitHub — it helps others discover it too.

---

## 9. License

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
