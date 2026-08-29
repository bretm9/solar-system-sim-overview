# N-Body Spaceflight Sim — real orbital mechanics in the browser

> A Kerbal-Space-Program-style sandbox that flies a craft through the **real solar system with full n-body gravitation**: 31 bodies seeded from JPL Horizons state vectors, a symplectic integrator running in a Web Worker, maneuver nodes, SAS, a transfer-window planner, and enough time warp to fly Earth → Mars → Phobos.

🔗 **Live demo:** not yet hosted publicly — a browser build is planned for [bretmerritt.com](https://www.bretmerritt.com).

This repo is an overview of a closed-source project. The source is private — I'm happy to walk through it in an interview.

<p align="center">
  <img src="screenshots/map-view.png" width="900" alt="Map view — orbit around Earth">
</p>

---

## Overview

No ship building, no career mode, no science — just a pre-built craft and a physically honest solar system. Every massive body pulls on every other body every step (Principia-style), rather than the patched-conic approximation most games use, and the simulation runs at a fixed rate in a worker so the main thread never blocks even at 100,000× time warp.

## Screenshots

| Flight HUD & navball | Maneuver node |
|---|---|
| ![Flight](screenshots/flight-hud.png) | ![Maneuver node](screenshots/maneuver-node.png) |

| Transfer window planner (Earth → Mars) |
|---|
| ![Transfer planner](screenshots/transfer-planner.png) |

## Features

- **Real solar system** — 31 bodies (Sun, planets, major moons) with real masses, radii, and rotation, seeded from JPL Horizons J2000 state vectors.
- **N-body physics** — a Blanes & Moan symplectic Runge–Kutta–Nyström integrator on Float64 typed arrays, in a **Web Worker** (Comlink RPC); an RK4 fallback for verification and an adaptive integrator for trajectory prediction.
- **Flight** — attitude, throttle, RCS translation, and a full SAS suite (stability, prograde/retrograde, normal/anti-normal, radial in/out, target/anti-target, maneuver hold) with KSP-standard keyboard and gamepad bindings.
- **Navball and map view** — Three.js rendering with a **floating origin and logarithmic depth buffer** so float32 precision holds across a 10¹³ m range; orbit lines, periapsis/apoapsis markers, and a live trajectory prediction.
- **Maneuver nodes** — create, drag-edit (prograde/normal/radial gizmos), and execute; burn-duration estimates from the craft's real thrust.
- **Transfer-window planner** — simple and advanced modes: porkchop-style departure scrubber, Hohmann and Izzo–Lambert solutions, LEO ejection Δv, and "warp to transfer".
- **Dynamic spheres of influence** — the craft's dominant body switches live as it crosses SOIs; the HUD, map, and planner all follow.
- **Time warp** 1× – 100,000×, quicksave/quickload and named saves (IndexedDB, compressed with pako, versioned schema with migrators), a pause menu, touch controls, and an in-game performance HUD.

## Technologies

TypeScript (strict) · Vite · Three.js · Web Workers + Comlink · gl-matrix · Zustand · IndexedDB + pako · Vitest

## Engineering notes

- **Physics in a worker, rendering on the main thread.** The integrator needs tens of thousands of steps per second at high warp; keeping it off the main thread is what keeps input lag and frame drops away.
- **Prediction is its own worker** and its polyline reuses one persistent GPU buffer across updates instead of reallocating ~50 kB of geometry per refresh.
- **Precision at solar-system scale.** A floating origin keeps the craft near (0,0,0) in the renderer while the simulation stays in the solar-system barycentric frame.
- **Tests as the safety net for math.** ~390 unit tests cover the integrators, Kepler/Lambert solvers, SOI transitions, save migrations, and input routing — a physics bug that produced silent NaN positions was caught by adding a test, not by staring at the screen.
- The project was run from a written spec with phased acceptance criteria and a deferred-items backlog; every phase shipped with type-check + tests green.

## Status

Built April 2026 (118 commits, ~23k lines of TypeScript). Earlier iterations explored a compressed "mini" scale before committing to real SI units; a sibling project ("Orbital") uses patched conics with a Lambert autopilot that flies Earth → Mars end-to-end, and a Unity/C# prototype ("Artemis Sim") explored the same design natively.

## Process

Built solo with Claude Code as a pair-programmer, working from a spec I wrote with explicit acceptance criteria per phase.

---

*Bret Merritt · [GitHub](https://github.com/bretm9) · [LinkedIn](https://www.linkedin.com/in/bret-merritt) · merrittbret9@gmail.com*
