# 🚕 GoRide — A Real-Time Ride-Sharing App for Android

A rider-side clone of the Uber/Lyft experience, built from scratch to explore how the hardest parts of ride-sharing actually work under the hood: live location tracking, real-time server-driven state, and smooth map animation.

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

## Why I built this

Ride-sharing apps look simple on the surface, but a single screen has to juggle a lot at once: live location updates for both rider and driver, a map that renders routes and moving markers, a server that can push state changes at any moment, and a strict trip lifecycle (search → book → pickup → trip → drop-off) that can never fall out of sync with what the user sees.

I wanted to understand — and be able to speak to in an interview — exactly how those systems fit together. So I built GoRide end-to-end, including a simulated WebSocket backend, so the entire rider journey could be developed and tested without depending on a live production server.

---

## What it does

- Requests location permission and shows nearby cabs on a live map
- Lets the rider pick a pickup and drop-off location with Places autocomplete
- Books a cab and shows a live-animated route as the driver approaches
- Streams the driver's location and moves the car marker smoothly, instead of snapping between GPS pings
- Updates the UI as the cab is arriving, arrives, starts the trip, and completes it
- Lets the rider immediately request another ride after a trip ends
- Handles errors (no route found, directions API failure) without breaking the app's state

---

## How it's built

**Architecture: MVP (Model–View–Presenter)**

I chose MVP over a heavier framework like MVVM so that every screen's logic could be unit-tested without touching the Android framework, and so the state machine for a trip lives in one obvious place — the Presenter — rather than being spread across a ViewModel and LiveData chain.

```
        user actions                state updates
   View ───────────────▶  Presenter ───────────────▶  Model / WebSocket Layer
(map, UI)                (business logic)            (simulated real-time backend)
```

| Layer | What it owns |
|---|---|
| **View** | Renders state only — map markers, bottom sheets, buttons, marker animation |
| **Presenter** | All business logic; decides what the view shows next, with zero Android dependencies |
| **Model / WebSocket layer** | The simulated backend that emits ride events in real time |

**Real-time layer, not polling**

The app talks to a WebSocket abstraction the same way it would talk to a production dispatch server — `connect()`/`disconnect()` manage the connection lifecycle, every server push arrives as a typed event, and network errors (like a failed directions call) come through a separate error channel so they never corrupt the trip state.

**The trip lifecycle is a strict state machine**

```
Discovery → Booking → cabBooked → cabIsArriving → cabArrived → tripStart → tripPath (live) → tripEnd → ready for next ride
```

Every one of those transitions is driven by an explicit server event — never a timer, never a guess. That was the main engineering constraint I held myself to: the UI should always be a direct reflection of the last thing the server said, nothing more.

---

## Tech stack

| Category | Choice |
|---|---|
| Language | Kotlin |
| Architecture | MVP |
| Maps & location | Google Maps SDK, Directions API, Places API |
| Real-time layer | Custom-simulated WebSocket module |
| Animation | Custom marker interpolation for smooth car movement along a route |
| Build | Gradle (Kotlin DSL) |

---

## Project structure

```
app/src/main/java/.../
├── ui/            View layer — activities, fragments, map rendering, marker animation
│   ├── maps/
│   └── booking/
├── presenter/     Per-screen business logic — the state machine lives here
├── model/         Data models: Cab, Trip, location events
└── simulator/      Simulated WebSocket server + client abstraction
```

---

## Engineering details worth highlighting

- **State machine, not spaghetti**: every trip transition is driven by exactly one server event type, so the UI can never desync from what actually happened.
- **Smooth marker animation**: raw GPS pushes come in every 1–2 seconds, but the car interpolates between them so it never looks like it's teleporting — the same problem a real driver-tracking map has to solve.
- **Testable core logic**: the Presenter layer has no Android framework dependencies, so trip logic can be unit tested directly.
- **Errors don't corrupt state**: a failed directions call or "no route found" is handled on its own channel, separate from normal trip events, so a network hiccup never leaves the app in a broken state.
- **Same contract as a real backend**: the simulated server speaks the same event format a production dispatch service would, so the client code doesn't need to change to point at a real one.

---

## Try it yourself

1. Clone the repo and open it in Android Studio
2. Enable **Maps SDK for Android**, **Directions API**, and **Places API** in the [Google Cloud Console](https://console.cloud.google.com/) and generate an API key
3. Add it to `local.properties`:
   ```properties
   sdk.dir=PATH_TO_ANDROID_SDK
   apiKey=YOUR_API_KEY
   ```
4. Build and run on an emulator or device with Google Play Services

---

## Screenshots

<p align="center">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/nearby-cabs.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/pickup-drop-location.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/request-cab-button.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/cab-is-arriving.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/on-trip.png" width="200">
  <img src="https://raw.githubusercontent.com/amitshekhariitbhu/ridesharing-uber-lyft-app/master/assets/trip-end.png" width="200">
</p>

---

## What I'd build next

- A driver-side app (this is currently rider-only)
- Fare estimation before booking
- Multiple vehicle tiers (economy / premium / pool)
- Persisted trip history
- Push notifications for trip state changes
- Swapping the simulated backend for a real WebSocket server, for a full end-to-end demo

---

## License

Apache License 2.0 — see [LICENSE](http://www.apache.org/licenses/LICENSE-2.0) for details.

---

<p align="center">If this was useful or interesting, a ⭐ helps others find it.</p>
