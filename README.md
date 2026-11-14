
- Modified Version of 0k (original 0k can be found here: https://github.com/abndnce/0k)
- Made by Claude, with
  checks and tests performed by me.
- credit to https://github.com/nautilus-os/GUST for ideas 
# 0k HTML

**A standalone HTML-based web proxy designed to operate directly from `file://`.**

## Overview

0k HTML is an experimental browser-based proxy runtime that aims to reproduce the functionality of conventional service-worker-based proxies without requiring an HTTP server or actual service-worker registration.

Browsers impose significant restrictions on documents opened through `file://`, including opaque origins, unavailable service-worker functionality, restricted networking, and differences in URL resolution.

0k addresses these limitations through a custom JavaScript interception and rewriting architecture.

## Architecture

0k consists of several cooperating components:

1. **HTML Preprocessing (VSW):** Processes documents before browser parsing, rewriting URLs and deferring resource attributes to prevent premature native requests.
2. **JavaScript Rewriting:** Applies Scramjet-style transformations to website code, handling navigation, networking, module imports, and other browser operations.
3. **Runtime Shim:** Intercepts dynamically created elements, resource attributes, CSS modifications, and browser APIs that cannot be handled during initial preprocessing.
4. **Virtual Service Worker:** Provides a custom request-handling layer that coordinates URL resolution, request processing, and response delivery.
5. **Parent Networking Controller:** Handles actual network transport and communicates responses back to the virtual environment.
6. **Blob-Based Delivery:** Uses browser-managed object URLs to deliver retrieved resources without relying on native requests to their original URLs.

## Resource Interception

A major challenge with `file://` is preventing the browser from bypassing the proxy through native resource loading.

0k uses a two-stage interception approach:

- **Preprocessing:** Original resource URLs are preserved in internal `data-zk-*` attributes while native resource attributes are replaced with placeholders.
- **Runtime restoration:** The shim processes deferred resources and routes them through the proxy's loading mechanisms.

Resource handling includes:

- Images and responsive images (`src`, `srcset`)
- Audio and video
- Iframes and frames
- Stylesheets and CSS imports
- Fonts and CSS resource URLs
- Scripts and module imports
- Resource hints and preload directives
- Dynamically created DOM elements

CSSOM operations, including `insertRule()`, `replace()`, `replaceSync()`, and `setProperty()`, also require interception to handle dynamically modified stylesheets.

## Networking

The virtual networking layer aims to reproduce functionality normally associated with service-worker-based proxies.

Supported or targeted interfaces include:

- Fetch
- XMLHttpRequest
- WebSocket
- EventSource
- Workers

Requests are routed through the virtual networking layer and parent controller rather than relying exclusively on native browser networking.

## Why `file://`?

Conventional browser proxies generally depend on HTTP(S) origins and service workers for request interception.

0k takes a different approach: it implements much of the required interception functionality directly in JavaScript.

This avoids depending on service-worker registration while introducing additional compatibility challenges involving browser security, origin virtualization, resource loading, and dynamic execution contexts.

**Importantly, 0k does not turn a local document into a genuine HTTPS origin.** It virtualizes selected browser behaviors while operating within the restrictions of the local-file environment.

## Development Goals

The primary engineering objectives are:

- Eliminate native resource-loading leaks.
- Improve URL resolution and rewriting accuracy.
- Ensure deferred resources are reliably restored.
- Expand HTML, CSS, SVG, and JavaScript interception coverage.
- Improve Fetch, XHR, and streaming compatibility.
- Support dynamic imports and worker execution.
- Preserve website behavior through origin and API virtualization.
- Improve compatibility with modern, complex websites.

## Project Status

0k is experimental and actively developed. Individual interception mechanisms and networking features require continued testing.
Individual sites are also receiving patches as well.

**The ultimate goal is a self-contained HTML proxy runtime capable of reproducing much of the functionality of conventional web proxies, such as Scramjet, while running directly from `file://`.**
