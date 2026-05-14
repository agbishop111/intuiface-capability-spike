# Intuiface capability spike (HTML)

**Status:** Technical feasibility spike — not production software.

## Purpose

Validate that a kiosk-style HTML5 experience can implement **in-session personalization** and the **in-kiosk portion** of a **post-visit takeaway** flow using only plain HTML, CSS, and JavaScript—without a proprietary authoring stack, build pipeline, or backend—so external systems can be added later with minimal rework.

## What was validated

| Area | Demonstrated |
|------|----------------|
| **Capability 2 — Personalization** | Sample “AI capability” cards with star selection, ordered in-memory state, Focus Map summary with **Primary** (first two picks) vs **Supporting**, empty state, view switching without losing selection order. |
| **Capability 3 — In-kiosk only** | Stable **session ID** at page load (human-readable prefix + digits), **QR** encoding a placeholder takeaway URL (`https://example.invalid/session?sessionId=…`), **versioned handoff payload** (`buildHandoffPayload()`), on-screen JSON preview, and console logging when opening the Focus Map. |

Capability 1 (read-only explore) is represented by the card catalog; the spike does not separate it as a distinct product module.

## Runtime constraints (design envelope)

- **Single artifact:** `index.html` only — embedded CSS and JavaScript.
- **Client-only:** no backend, no API calls, no `localStorage`; state is in-memory for the page lifetime.
- **No toolchain:** no npm, `package.json`, bundlers, or TypeScript.
- **No frameworks:** vanilla DOM, show/hide views (no client router).
- **Kiosk-oriented:** larger touch targets, simple navigation, readable structure for handoff to Intuiface or similar hosts.

## Architectural approach

- **Views:** Two panels (Explore / Focus Map) toggled with CSS (`display` / `is-active`).
- **State:** `selectedCapabilities` as an ordered array of capability IDs; `sessionId` and `sessionCreatedAt` set once at load.
- **QR:** [qrcodejs](https://github.com/davidshimjs/qrcodejs) (MIT) **inlined** in the same file; application code uses a small adapter (`mountQrCodeWithLibrary`) so the encoder can be swapped without rewriting UI logic.
- **Handoff:** `buildHandoffPayload()` returns a versioned object (`schemaVersion: 1`, `handoffStatus: "in-kiosk-demo-only"`) suitable as a contract sketch for a future POST body or message payload.

## Intentionally out of scope

- Cross-device persistence, session lookup after leaving the kiosk, or real personalized assets.
- Database, APIs, auth, or server-rendered takeaway pages.
- End-to-end QR resolution to a live service (placeholder URL only).
- Production hardening (analytics, error monitoring, accessibility audit, WebView matrix sign-off, etc.).

## How to run

1. Open `index.html` in a desktop or mobile browser (double-click, or **File → Open**).
2. Optionally serve the directory with any static file server; behavior does not depend on it.
3. Select cards on **Explore**, open **Focus map** to see summary, session, QR, and **Developer handoff preview**. Open DevTools **Console** before clicking **View focus map** to see the logged payload.

## Offline compatibility notes

- **No network is required** for core behavior once the file is loaded: QR generation uses the inlined library, not a CDN.
- **`file://`:** Supported for this spike; QR uses the library’s canvas path. If a specific embedded WebView blocks canvas `toDataURL` or related paths, retest in that runtime and adjust (e.g. table/SVG rendering) — not exhaustively validated here.

## Future requirements for full Capability 3

A complete post-kiosk experience would still need, at minimum:

- **Durable session storage** and correlation with the identifier encoded in the QR.
- **Backend/API** to accept or merge handoff payloads and drive personalization.
- **Public or authenticated takeaway surface** (web or document) reachable from the encoded URL.
- **Operational concerns:** TLS, abuse controls, retention policy, PII classification, and Intuiface/WebView compatibility testing on target hardware.

## Key takeaways

- **Feasibility:** Capability-style UX (select → summarize → show session + QR + structured payload) is achievable in one static HTML file with clear seams for later infrastructure.
- **Contract clarity:** A small, versioned payload shape and a stable session identifier give downstream teams a concrete integration target without committing to stack choices prematurely.
- **Limits:** This spike proves **packaging and client-side behavior**, not security, scale, or real-world session continuity. Treat findings as input to a proper product design and WebView test plan—not as a shipping baseline.
