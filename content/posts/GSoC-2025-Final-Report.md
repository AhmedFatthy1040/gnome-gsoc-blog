+++
date = '2025-08-31T18:00:00+03:00'
draft = false
title = 'GSoC 2025 Final Report: Backend Isolation Prototype for GNOME Papers'
slug = 'gsoc-2025-final-report'
tags = ['GSoC', 'GNOME', 'Papers', 'open-source', 'D-Bus', 'IPC', 'sandboxing']
categories = ['GSoC Journey']
summary = 'Wrapping up my GSoC 2025 project: a working per-document sidecar (process isolation) prototype for GNOME Papers with D-Bus page objects and shared-memory rendering.'
+++

**Project**: Papers: Proof of Concept for Backend Isolation (Per-Document Sidecar Architecture)  
**Organization**: GNOME  
**Mentors**: Pablo Correa Gómez, Lucas Baudin, Markus Gollnitz, Qiu Wenbo  
**Student**: Ahmed Fatthi Al-Khateeb

---

## 1. Goal Recap (What We Set Out To Do)
The mission was to prototype per-document process isolation for Papers (GNOME’s next‑gen document viewer). The intent: each opened document runs in its own helper (“sidecar”) process so crashes or exploits in parsing/rendering code don’t take down the UI or leak other open documents. Success criteria were exploratory: map feasibility, design clean IPC, and surface risks shipping production quality was a stretch goal, not a requirement.

Security benefit: exploit scope reduction.  
Reliability benefit: unlock safe multi‑tab / multi‑document UX.

---

## 2. What I Actually Built
**High-level outcome**: A working prototype where the main process spawns a dedicated `document-process` per file. Each sidecar:
- Loads exactly one document.
- Exposes a hierarchical D‑Bus object model: one Document object plus individual Page objects.
- Renders pages on demand and returns pixel data via shared memory (memfd + fd passing).

On the client side there is:
- A high-level async API (factory + page wrapper) that hides D‑Bus details.
- A simple proxy-based CLI test client.
- A GTK4 viewer (`simple-viewer`) demonstrating opening a PDF, navigating pages, and rendering them without UI stalls.

### Architecture Snapshot
| Layer | Purpose |
|-------|---------|
| `document-process.c` | Sidecar service: loads the document, exports Document + Page skeletons, handles rendering + annotations test call. |
| `pps-sandboxed-document-factory.[ch]` | Spawns sidecars, retries connection, returns a ready document proxy (async). |
| `pps-sandboxed-page.[ch]` | Page-level async operations (render, size) returning Cairo surfaces. |
| `pps-shared-memory.[ch]` | memfd-backed Cairo surface creation / validation / cleanup. |
| `simple-viewer.c` | GTK4 demo to validate async flow and rendering pipeline. |
| `document-client.c` | Lower-level proxy test harness. |

### D‑Bus Object Model
- Document: `/org/gnome/Papers/DocN`  
  - Methods: `GetNPages`, `GetPage`, `GetAnnotations` (legacy compatibility)
- Pages: `/org/gnome/Papers/DocN/PageM`  
  - Methods: `Render(width,height)` → (fd + metadata), `GetPageSize()`

### Rendering / Data Path
Cairo render in sidecar → memfd with structured header → fd passed via D‑Bus → client maps and wraps into Cairo surface → optional conversion to texture for GTK.

### High-Level API Features
- Safe async creation with retry loop (up to 10 attempts, 500ms interval) before declaring failure.
- Automatic bus name + object path generation (`org.gnome.Papers[.Devel].DocN`, `/org/gnome/Papers/DocN`).
- Separation of concerns: factory = lifecycle, page object = graphics operations.
- All error paths return meaningful `GError` domains/codes.

### Demo Viewer Validated
- Non-blocking UI during page transitions.
- Multiple sequential renders create expected debug artifacts (`papers-render-*.png`).
- Correct aspect ratio scaling and centering logic.

---

## 3. Representative Run (Condensed)
Excerpt of a typical debug trace (simplified):
```
Spawned sidecar PID 11
Document loaded successfully: 16 pages
Exported page objects: /Doc1/Page0..Page15
GetNPages → 16
GetPage(0) → /Doc1/Page0
Page.Render(0, 1200x1600) → memfd stride=4800 size=7.3MB
Page.Render(1 ...) ...
```
This confirmed stable sequencing: spawn → connect → enumerate → page proxy → render.

---

## 4. Key Technical Decisions
| Decision | Why | Trade-off |
|----------|-----|-----------|
| Per-page D‑Bus objects | Extensibility (future: links, text, forms) | More objects on the bus |
| memfd + fd passing | Efficient large bitmap transfer | Linux-specific (for now) |
| Async GLib API (`GTask`) | Integrates with GNOME stack | Verbose vs. coroutine style |
| Simple counter for IDs | Deterministic, minimal | No reuse / crash detection yet |
| Keep annotations on Document temporarily | Transitional simplicity | Needs relocation later |
| Session bus prototype | Fast to iterate | Not optimal for perf / isolation |

---

## 5. Challenges & How I Approached Them
- **Race between spawn and bus readiness**: Added retry loop instead of blocking sleep.
- **Surface reconstruction correctness**: Embedded a header (width/height/stride/format) before pixel data; validated on map.
- **Aspect ratio rendering**: Compute scale by min(width_ratio, height_ratio), center in target surface, preserve whitespace.
- **File descriptor hygiene**: Close local fd copies immediately after adding to `GUnixFDList`; attach cleanup via Cairo user data.
- **GTK pipeline stability**: Converted Cairo → Pixbuf → Texture for simplicity; kept logging around pixel formats.
- **Observability**: Uniform `g_debug()` markers for every step (spawn, export, page fetch, render) to speed diagnosis.

---

## 6. What’s Missing / Not Finished (By Design for Prototype Stage)
- **Meson integration gap**: Current `libdocument/sandbox/meson.build` omits `gdbus-codegen` steps and doesn’t compile the high-level factory/page sources needs wiring so downstream code can consume the API properly.
- **No sidecar supervision**: If a sidecar crashes mid-session, there’s no automatic detection or restart signaling to the UI.
- **No memory/page caching policy**: All renders are cold; no LRU or reuse of previously rendered sizes.
- **Security hardening not re-enabled**: bubblewrap / tighter sandbox profiles deferred due to dev iteration friction.
- **Transport still via session bus**: Eventually should move to peer-to-peer (Unix stream pair) to reduce overhead and remove global name coordination.
- **Lifecycle & invalidation signals**: No signals for “page changed”, “document closing”, or crash events.
- **Automated tests**: No CI harness for spawning N sidecars, stress rendering, or fuzzing D‑Bus entrypoints.
- **Cross-platform abstraction**: memfd approach is Linux-first; portability layer would be needed later.

---

## 7. Lessons Learned
- Start with **end-to-end vertical slices** early: rendering + transport + consumption beats isolated micro-benchmarks.
- Hierarchical IPC objects reduce future API churn. Pages as first-class entities unlock cleaner feature layering.
- A tiny investment in structured logging repays itself once multiple processes are talking.
- memfd + Cairo is a pragmatic sweet spot for binary pixel transport far simpler than inventing a custom image protocol.
- Keeping the prototype honest (real viewer, not just synthetic tests) flushed out timing and sequencing issues quickly.

---

## 8. How To Run the Prototype (Recap)
Inside Flatpak dev environment:
```bash
export PPS_ENABLE_SIDECAR_LOADING=1
export G_MESSAGES_DEBUG=PapersExamples   # optional
flatpak run --command=simple-viewer org.gnome.Papers.Devel
```
Manual D-Bus probing (replace bus name appropriately):
```bash
gdbus call --session --dest org.gnome.Papers.Devel.Doc1 \
  --object-path /org/gnome/Papers/Doc1 \
  --method org.gnome.Papers.Devel.Document.GetNPages
```
(Note: `Render` returns an fd better tested through the viewer or the proxy client.)

---

## 9. Impact & Outcome
Even as a prototype, the work:
- Demonstrates concrete feasibility of per-document isolation with modest architectural intrusion.
- Establishes patterns (page objects, shared memory surfaces, async factory) that can scale.
- Surfaces the *real* integration blockers early: supervision, build wiring, and security profiling.
- Provides a runnable, visually verifiable artifact lowering the barrier for future contributors to iterate.

It shifts future work from “Can this even work?” to “How do we productionize it safely and ergonomically?”

---

## 10. Personal Reflection
I deliberately started by setting up D-Bus code generation and thinking about how the eventual developer-facing API should feel; later I parked parts of that integration so I could focus on proving isolation + rendering + transport correctness. That ordering helped keep the design coherent even when I temporarily deferred polishing the Meson integration. I’m happy that the hard questions (object model, rendering transport, async ergonomics) were answered early enough that the remaining tasks feel like engineering follow-through rather than research.

---

## 11. Work Reference & Next Step
Here is the link of my work: https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598

In September we are going to squash a few commits and push the branch to something like `poc/sandbox` in the main repository.

## 12. Closing
This prototype gives Papers a credible path toward safer, crash-resilient multi-document viewing. There’s still supervision, sandbox tightening, and build integration to finish but the architectural spine is in place. I’d be glad to continue refining it or help onboard the next contributor picking it up.

_Thanks to the mentors and GNOME community for guidance and review._
