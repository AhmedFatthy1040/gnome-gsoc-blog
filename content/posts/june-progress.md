+++ 
date = '2025-06-28T18:00:00+03:00'
draft = false
title = 'GSoC 2025: June Progress Report'
tags = ['GSoC', 'GNOME', 'Papers', 'open-source', 'D-Bus', 'IPC']
categories = ['GSoC Journey']
+++

June has been a month of deep technical work and architectural progress on my GSoC project with GNOME Papers. Here’s a summary of the key milestones, challenges, and decisions from the month.

---

## �️ Architecture Overview

To better illustrate the changes, here are diagrams of the current (unsandboxed) and the new (sandboxed) architectures for GNOME Papers:

**Current Architecture (Unsandboxed):**

![Current unsandboxed architecture](/Unsandboxed_current_arch.png)

**Target Architecture (Sandboxed):**

![Target sandboxed architecture](/Sandboxed.png)

---



## ️ Early June: Prototyping, Research & First Meeting

*Note: D-Bus is a system that lets different programs on your computer talk to each other, even if they are running in separate processes.*

The month started with a meeting on **June 5th** where I presented my research on Glycin and discussed architectural ideas for backend isolation in GNOME Papers. I also demoed my first prototype, which was a basic proof-of-concept for D-Bus communication within a single process. The prototype used a hardcoded D-Bus interface:

```c
/* D-Bus interface XML */
static const gchar introspection_xml[] =
  "<node>"
  "  <interface name='org.gnome.Papers.Test'>"
  "    <method name='GetAnnotations'>"
  "      <arg type='i' name='page_num' direction='in'/>"
  "      <arg type='as' name='annotations' direction='out'/>"
  "    </method>"
  "    <method name='RenderPage'>"
  "      <arg type='i' name='page_num' direction='in'/>"
  "      <arg type='s' name='result' direction='out'/>"
  "    </method>"
  "  </interface>"
  "</node>";
```

I implemented handlers for the interface methods (`handle_get_annotations`, `handle_render_page`) and set up the necessary callbacks. This allowed the process to call itself using D-Bus, which was a good starting point for understanding the communication model. [See the commit for details.](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=773d062bf86c62403443c3b32074ca329c939ec2)

After the meeting, I was tasked with making the main Papers app talk to the prototype: specifically, to have `PpsJobLoad` start a sandboxed process and load the document in both the main and “sidecar” processes. This allowed us to test document duplication and focus on designing robust inter-process communication.

---

## 🔧 Mid-June: Refactoring and Improvements

Between **June 5th and 16th**, I worked on several improvements based on maintainer feedback:

- Fixed document loading by adding a missing `pps_document_load()` call.
- Added `pps_document_setup_cache()` for proper page rendering.
- Replaced manual D-Bus XML handling with generated skeleton/proxy code:
  - Created `org.gnome.Papers.Test.xml` interface definition.
  - Updated build system to use `gnome.gdbus_codegen()`.
  - Refactored code to use generated skeletons and type-safe completion functions.
  - Modernized async callback signatures and reduced boilerplate.
- Added process management utilities and environment variable controls for daemon and sidecar modes.
- Implemented sandboxing logic (using bubblewrap, with fallback to direct execution), though I encountered issues with Flatpak environments. Markus suggested using `flatpak-spawn` for better compatibility, which I plan to try soon.

Relevant commits:
- [Refactor to generated D-Bus code](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=585996b3ffb798dfb007629e16f22d83ce6066ce)
- [Process management improvements](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=c09adf9742efd7f27008a5a2d0170c40f2c8f792)
- [Sandboxing and environment controls](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=8b6109f17dc452127ad245737fd136f68193f45d)

---

## 🧩 June 16th Meeting: Designing the Daemon and Process Communication

In our **June 16th** meeting, we reviewed the latest work and discussed the architecture for communication between the main process (daemon) and document processes. The key design points:

- The daemon maintains a registry (e.g., a HashMap) of all document processes.
- The main process can call document processes (e.g., to render pages, load annotations).
- Document processes can emit signals (e.g., file changes, rendering completion).
- Large objects (like rendered textures) should be shared via shared memory, inspired by Glycin.

---

## 🚀 Late June: Daemon Registry and Shared Memory

Following the meeting, I implemented:

- A `DocumentProcessInfo` structure for tracking sidecar processes.
- Daemon-side process tracking with URI-to-D-Bus path mapping.
- Lifecycle management for document processes.
- Daemon-assigned D-Bus paths via file-based IPC.
- Sidecar process discovery and service availability checks.
- Periodic status reporting and debugging tools.

The daemon now maintains a comprehensive registry of all sidecar processes, including their D-Bus paths, PIDs, and status. Sidecar processes request D-Bus paths from the daemon before spawning, enabling robust coordination.

[Commit: Daemon registry and process management](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=9f2722e909dde6ebe89778e7c87604f8b157179d)

I also replaced string-based image transfer with efficient shared memory using `memfd`, allowing high-performance image data exchange between processes:

- Added `shared-memory.{c,h}` for memfd-based Cairo surface sharing.
- Updated D-Bus interface: `RenderPage` now returns a file descriptor and metadata.
- Implemented real PDF rendering with scaling.
- Added Unix FD list support for secure file descriptor passing.

[Commit: Shared memory for rendering](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=6cdd1b91712558d6663655dc0cca13b462a8cfa0)

---

## 📝 In-Depth: Early June, Research & Prototyping

The month started with a meeting on **June 5th** where I presented my research on Glycin and discussed architectural ideas for backend isolation in GNOME Papers. I also demoed my first prototype, which was a basic proof-of-concept for D-Bus communication within a single process. The prototype used a hardcoded D-Bus interface:

```c
/* D-Bus interface XML */
static const gchar introspection_xml[] =
  "<node>"
  "  <interface name='org.gnome.Papers.Test'>"
  "    <method name='GetAnnotations'>"
  "      <arg type='i' name='page_num' direction='in'/>"
  "      <arg type='as' name='annotations' direction='out'/>"
  "    </method>"
  "    <method name='RenderPage'>"
  "      <arg type='i' name='page_num' direction='in'/>"
  "      <arg type='s' name='result' direction='out'/>"
  "    </method>"
  "  </interface>"
  "</node>";
```

I implemented handlers for the interface methods (`handle_get_annotations`, `handle_render_page`) and set up the necessary callbacks. This allowed the process to call itself using D-Bus, which was a good starting point for understanding the communication model. [See the commit for details.](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=773d062bf86c62403443c3b32074ca329c939ec2)

After the meeting, I was tasked with making the main Papers app talk to the prototype: specifically, to have `PpsJobLoad` start a sandboxed process and load the document in both the main and “sidecar” processes. This would allow us to test document duplication and focus on designing robust inter-process communication.

---

## 🛠️ Mid-June: Refactoring, Automation, and Sandboxing

Between **June 5th and 16th**, I worked on improvements suggested by maintainers. These included:

- Fixing document loading by adding a missing `pps_document_load()` call.
- Adding `pps_document_setup_cache()` for proper page rendering.
- Replacing manual D-Bus XML handling with generated skeleton/proxy code:
  - Created `org.gnome.Papers.Test.xml` interface definition.
  - Updated `examples/meson.build` to use `gnome.gdbus_codegen()`.
  - Refactored `dbus-test.c` to use the generated `PapersTestTest` skeleton.
  - Replaced manual `GDBusInterfaceVTable` with signal-based handlers.
  - Removed embedded XML strings and manual method dispatching.
  - Used type-safe completion functions (`papers_test_test_complete_*`).
  - Modernized async callback signatures for `GAsyncReadyCallback`.

**Benefits:**
- Type safety at compile time
- Reduced boilerplate code
- Standard GNOME/GLib D-Bus patterns
- Interface compatibility guarantee between client/server

I also added:
- `_pps_spawn_dbus_daemon()` in `pps-init.c` for daemon mode
- `_pps_spawn_sidecar_process()` in `pps-jobs.c` for per-document sidecar mode
- Comprehensive search logic for the `dbus-test` executable across Flatpak and standard environments
- Bubblewrap sandboxing with fallback to direct execution
- Environment variable controls: `PPS_ENABLE_DBUS_DAEMON` and `PPS_ENABLE_SIDECAR_LOADING`
- Included `unistd.h` in `dbus-test.c` for the `daemon()` function

However, I ran into issues using bubblewrap with Flatpak, so I commented out that code for now. Markus suggested using `flatpak-spawn` for better compatibility, which I haven’t tried yet but plan to in the future.

Relevant commits:
- [Refactor to generated D-Bus code](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=585996b3ffb798dfb007629e16f22d83ce6066ce)
- [Process management improvements](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=c09adf9742efd7f27008a5a2d0170c40f2c8f792)
- [Sandboxing and environment controls](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=8b6109f17dc452127ad245737fd136f68193f45d)

---

## 🧠 June 16th Meeting: Daemon Design & Communication Model

In our **June 16th** meeting, we reviewed the latest work and discussed how the communication between document processes and the daemon (which manages document processes) should be designed. We decided that:

- The daemon should maintain a HashMap (or similar) containing every document process.
- The main process calls the document processes (e.g., `render_page`, `load_annotations`).
- Document processes may emit signals to the main process (e.g., file changed, rendering finished).
- Shared memory should be used to share large objects (like rendered textures), inspired by Glycin.

---

## 🏗️ Late June: Daemon Registry, Shared Memory, and More

After the meeting, I implemented several major features:

- Added a `DocumentProcessInfo` structure for tracking sidecar processes.
- Implemented daemon process tracking with document URI to D-Bus path mapping.
- Added process lifecycle management functions (register, cleanup, monitoring).
- Implemented a daemon-assigned D-Bus path system via file-based IPC.
- Added sidecar process discovery and service availability checking.
- Extended sidecar spawning to use daemon-assigned paths.
- Added periodic status reporting and debugging capabilities.

The daemon now maintains a registry of all sidecar processes with their D-Bus paths, PIDs, and service availability status. Sidecar processes request D-Bus paths from the daemon before spawning, enabling proper process coordination and tracking.

[Commit: Daemon registry and process management](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=9f2722e909dde6ebe89778e7c87604f8b157179d)

I also replaced string-based `RenderPage` with efficient `memfd` shared memory transfer, enabling high-performance image data exchange between sidecar processes:

- Added `shared-memory.{c,h}` for memfd-based Cairo surface sharing
- Updated D-Bus interface: `RenderPage` now returns fd + metadata instead of string
- Implemented real PDF rendering with scaling in `handle_render_page()`
- Added Unix FD list support for secure file descriptor passing
- Updated `meson.build` to include `shared-memory.c`

This eliminates large image data serialization through D-Bus buffers, providing the foundation for efficient inter-process rendering.

[Commit: Shared memory for rendering](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=6cdd1b91712558d6663655dc0cca13b462a8cfa0)

---

## 🧭 June 23rd Meeting & Next Steps

In our **June 23rd** meeting, we discussed two possible directions for moving forward:

1. Build a simple app that spawns a document process and shows rendered pages (probably easier).
2. Gradually use a `PpsDocumentSkeleton` instead of a `PpsDocument` in the main process (see diagram), and update the architecture to add page objects and various D-Bus objects.

**Tasks moving forward:**
- Write this GSoC update post.
- Remove the daemon part from the prototype.
- Update the architecture diagram to add pages (`PpsDocumentSkeleton::get_pages()` → a list of `PpsPageSkeleton`?).
- Implement various D-Bus objects for pages and documents.

---

## 📚 Final Summary: Current State of the Prototype

### D-Bus Interface: `org.gnome.Papers.DocumentProcessor`
Currently, the D-Bus interface defines the following methods:
- `GetAnnotations`
- `RenderPage` (currently in the process of being removed)

**My tasks:**
- Remove the `RenderPage` method.
- Add two new methods:
  - `GetPage(int n)` → returns the object path for a specific page.
  - `GetNPages()` → returns the total number of pages in the document.

#### D-Bus Interface Introspection XML

Here is the D-Bus interface after the last code split and shared memory implementation:

```xml
<!DOCTYPE node PUBLIC "-//freedesktop//DTD D-BUS Object Introspection 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/introspect.dtd">
<node>
  <interface name="org.gnome.Papers.DocumentProcessor">
    <method name="GetAnnotations">
      <arg type="i" name="page_num" direction="in"/>
      <arg type="as" name="annotations" direction="out"/>
    </method>
    <method name="RenderPage">
      <arg type="i" name="page_num" direction="in"/>
      <arg type="i" name="target_width" direction="in"/>
      <arg type="i" name="target_height" direction="in"/>
      <arg type="h" name="memfd" direction="out"/>
      <arg type="i" name="width" direction="out"/>
      <arg type="i" name="height" direction="out"/>
      <arg type="i" name="stride" direction="out"/>
      <arg type="i" name="format" direction="out"/>
      <arg type="t" name="data_size" direction="out"/>
    </method>
  </interface>
</node>
```

### New Interface: `org.gnome.Papers.Page`
I am also working on a new interface called `org.gnome.Papers.Page`, which will expose:
- `Render(int width, int height)` → renders the page into shared memory.
- `GetSize()` → returns the size (width, height) of the page.

The `Render` method uses a shared memory approach via `memfd`, allowing the document process to send a file descriptor to the main process for efficient page image transfer.

### Shared Memory Utilities (`shared-memory.c`)

*Note: Shared memory is a way for different programs to quickly exchange large amounts of data (like images) by sharing a part of the computer's memory, instead of copying data back and forth.*
This file provides helper functions for working with shared memory:
- `create_shared_memory_from_surface(surface)`
  - Used in the document process to create a memory file descriptor (memfd) and write the rendered surface to it.
    *A memfd is a special kind of temporary file stored in memory, which makes sharing data between programs fast and efficient.*
- `read_surface_from_shared_memory(fd, ...)`
  - Used in the main process to map the file descriptor and reconstruct the rendered surface.
This mechanism allows zero-copy image transfer between the two processes over D-Bus.

### Document Process Logic (`document-process.c`)
This file implements the server-side logic for the D-Bus interface. It currently handles:
- `handle_get_annotations`
- `handle_render_page` (to be removed)
- Document loading and lifecycle management

#### Example: handle_render_page with Shared Memory

Below is the updated `handle_render_page` function, which demonstrates how a rendered page is transferred efficiently using shared memory and Unix file descriptors:

```c
static gboolean
handle_render_page (PapersDocumentProcessorDocumentProcessor *interface,
                    GDBusMethodInvocation *invocation,
                    gint page_num,
                    gint target_width,
                    gint target_height)
{
    // ...existing code...
    /* Create shared memory from rendered surface */
    gint width, height, stride, format;
    guint64 data_size;
    gint memfd = create_shared_memory_from_surface (surface, &width, &height, &stride, &format, &data_size);

    // ...add fd to GUnixFDList, return via D-Bus...
    GVariant *result = g_variant_new ("(hiiiit)", fd_index, width, height, stride, format, data_size);
    g_dbus_method_invocation_return_value_with_unix_fd_list (invocation, result, fd_list);
    // ...existing code...
}
```

**Current work:**
- Extending `document-process.c` to support page objects:
  - Adding a `page_skeletons` GHashTable: `{ "/org/gnome/Papers/Doc1/Page1" → PpsPageSkeleton* }`
  - Implementing cleanup logic for this table and the corresponding skeletons.
  - Writing D-Bus handlers for:
    - `handle_get_page()` → returns a D-Bus path to the requested page.
    - `handle_get_n_pages()` → returns the total number of pages.

### D-Bus Testing Client (`document-client.c`)
This file was originally intended for testing the D-Bus interface. I am not actively using it right now, since I currently test the handlers using `gdbus call` after opening a document.

### Current Considerations
I am exploring two possible paths:

**Option 1 (Under Consideration):**
- Adapt functions like `pps_document_get_page()` in `pps-document.c` to use the same D-Bus logic as `document-client.c`.
- This would allow the main Papers app to access remote documents via D-Bus.
- Once this integration is stable, `document-client.c` could be safely removed.

**Option 2 (Alternative Plan):**
- Extend `document-client.c` to support full page functionality for testing.
- Add a `page_proxies` GHashTable: `{ "/org/gnome/Papers/Doc1/Page1" → PpsPageProxy* }`
- Implement:
  - `get_page_proxy()` to retrieve a proxy from the table.
  - Cleanup logic for freeing the `page_proxies` hash table and destroying its contents.

**Functions in `document-client.c`:**
- `get_annotations_callback()` — Called when the asynchronous `GetAnnotations` D-Bus call completes.
- `render_page_callback()` — Called when the asynchronous `RenderPage` D-Bus call completes.
- `test_dbus_calls()` — Starts the test sequence by making an asynchronous `GetAnnotations` D-Bus call.
- `test_dbus_calls_timeout()` — Timer callback that triggers shortly after the main loop starts.
- `quit_main_loop_timeout()` — Timer callback to quit the main loop after the test sequence completes.

### Utility Functions (`process-utils.c`)
This file contains helper functions that are used multiple times across the codebase. It helps reduce duplication and keeps commonly used operations organized and reusable.

### Process Tracking Structures (in `pps-jobs.c`)

When the daemon process was removed, process tracking was moved into the main process. Here is the structure and globals used:

```c
typedef struct {
    gchar *document_uri;
    gchar *dbus_path;
    pid_t sidecar_pid;
} DocumentProcessInfo;

static GHashTable *document_processes = NULL;
static GHashTable *uri_to_path = NULL;
static guint document_process_id = 0;
```

### How I Test My Work
To test the D-Bus interfaces and handlers, I follow this process:
1. Run Papers with debugging and sidecar support enabled:
   ```sh
   flatpak run --env=PPS_ENABLE_SIDECAR_LOADING=1 --env=G_MESSAGES_DEBUG=all \
     org.gnome.Papers.Devel
   ```
   This launches the Papers development version with verbose debugging output and enables loading documents in a separate (sidecar) process.
2. Open a document in the UI to trigger the creation of the document process and D-Bus object (e.g., `/org/gnome/Papers/Doc1`).
3. Open another terminal and issue D-Bus method calls manually using `gdbus`:
   ```sh
   gdbus call --session --dest :1.214 --object-path /org/gnome/Papers/Doc1 --method org.gnome.Papers.Test.GetAnnotations 0
   ```
   Replace `:1.214` with the actual bus name of the document process (you can get this from the debug output or by listing active D-Bus connections).
4. Observe the response in the second terminal (the one calling `gdbus`), and monitor the debug output from the first terminal (running Papers) for logging, warnings, or handler traces.

This manual testing method helps me verify that D-Bus methods are wired correctly and that the data flow between the main process and the document process behaves as expected.

---

## 🆕 Major Refactors and Architectural Simplifications (Late June)

As the month progressed, I made several major architectural changes and refactors to further simplify and modernize the backend isolation prototype. Here are the most important ones:

### 1. Remove Daemon Process, Centralize Document Tracking

I removed all daemon process code and file-based IPC mechanisms, moving document process tracking and D-Bus path generation directly into the main process. Now, the main process manages all sidecar processes, and each document gets a unique D-Bus path using a combination of PID and timestamp, which solves the fork inheritance issue where child processes would generate identical paths. This also eliminated the need for periodic monitoring and complex D-Bus service discovery logic, making the architecture much simpler and more robust.

**Commit:** [Remove daemon process and centralize document tracking](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=b5a456f5f74b3734caf53a3f6272b67910e4a935)

### 2. Use g_spawn_async and Counter-Based D-Bus Paths

I replaced `fork()` with `g_spawn_async` for better process management and implemented a static counter for unique D-Bus paths. Now, each document gets a D-Bus path like `/org/gnome/Papers/Doc0`, `/org/gnome/Papers/Doc1`, etc., instead of timestamp-based paths. This approach is simpler, more reliable, and provides predictable D-Bus path generation for document sidecars. The counter is incremented before each sidecar spawn, and the process ID is passed as the first argument to the sidecar executable. This also improves compatibility and maintainability.

**Commit:** [Replace fork() with g_spawn_async and static counter for D-Bus paths](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=7859fc5bbbdd709a9e39a206df2bbe5a2527225c)

### 3. Split dbus-test into Client/Server, Modernize Interface

I split the original `dbus-test.c` into two separate files: `document-process.c` (server) and `document-client.c` (client). I also created `process-utils.c/h` for shared functionality, renamed the D-Bus interface from `org.gnome.Papers.Test` to `org.gnome.Papers.DocumentProcessor`, and updated the build system and process spawning logic accordingly. This refactor improved code organization, enabled better signal handling and error management, and made the codebase much easier to maintain and extend.

**Commit:** [Split dbus-test, modernize interface](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/598/diffs?commit_id=be93fcdf73314c3be9704d02052921d2a3a04f2a)

---

June has been a month packed with foundational work, architectural decisions, and technical growth. I’m excited to continue building on this progress in July and to see how these changes will shape the future of GNOME Papers!
