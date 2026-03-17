# Kaya WPF Developer Guide for AI Assistants

This document provides guidance for AI assistants working on the Kaya WPF codebase. Kaya is a very simple, immutable, local-first-friendly note-taking and bookmarking platform. This is the WPF port of the GTK desktop app, targeting Windows with .NET 9.

The app allows users to store, search, and sync their notes and bookmarks.

## Planning

Read [doc/plan/PLAN.md](./doc/plan/PLAN.md) and follow those instructions for creating a plan before any work is performed.

---

## Prompt History

You can find a chronological list of significant past prompts in [doc/PROMPTS.md](./doc/PROMPTS.md). Major prompts are titled with Subheading Level Two (\#\#), sub-prompts are titled with Subheading Level Three (\#\#\#).

The current major prompt or bugfix will probably also be in this file, uncommitted.

This file will get large, over time, so only prioritize reading through it if you require additional context.

---

## Essential Commands

* `dotnet {build|test|run}`
* `.\build.ps1 {build|test|run|clean|publish}`

---

## Codebase Overview

* .NET 9
* plain WPF
* Fluent theme with `ThemeMode="System"` for dark/light switching
* passwords: Windows Credential Manager
* unit tests in xUnit

---

## Testing

Unit tests should be written for all new features and bug fixes. Prefer test-driving code (Red => Green => Refactor). Prefer code in models rather than services, since it's more testable. Extract code into models whenever possible to reduce duplication.

---

## Core Concept

The simplicity of Kaya comes from a very simple concept: that notes and bookmarks are just files on disk, each named in a sequential fashion, following a `YYYY-mm-ddTHHMMSS` format, in UTC. These files are _immutable_, meaning that the user records one and then never touches it again. The timestamp associated with the file corresponds to the time (in UTC) the user made the record. In the rare case when there is a sub-section collision, the filename prefix format is `YYYY-mm-ddTHHMMSS_SSSSSSSSS`, representing nanoseconds.

The core functionality of Kaya comes from retrieval: looking up old notes, bookmarks, and files.

Anga (notes, bookmarks, and files) use this Core Concept, but it also applies to metadata files and other immutable data within the system. This makes it easy for Kaya clients to use peer-to-peer folder synchronization to stay up to date with one another, without Kaya Server.

---

## Domain Model

### Kaya, the "heap"

"Kaya" means a "heap" or "collection" in Pali. It refers to each user's timestamped pile of files.

### Anga, the "part"

"Anga" means "constituent part" or "limb" in Pali. Every timestamped record in the user's heap is one constituent part.

**Anga File Types:**

* `.md` - Markdown files are notes
* `.url` - URL files, in the style of Microsoft Windows, are bookmarks
* `.*` - any other file types (images, PDFs, etc.) are simply stored as-is

---

## API

Sync with the Rails server (default: savebutton.com) happens through its API, using HTTP Basic Authentication.

**APIs:**

* GET `/api/v1/handshake` - allows the client to discover this user's namespaced API endpoint
* GET `/api/v1/:user_email/anga` - returns a `text/plain` mimetype containing a simple, flat list of files with the format mentioned under "Core Concept": `2025-06-28T120000-note-to-self-example.md` so that clients can "diff" their list of files with the server's list of files for a given user
* GET `/api/v1/:user_email/anga/:filename` - returns the file as though it were accessed directly via Apache or nginx
* POST `/api/v1/:user_email/anga/:filename` - allows the client to directly POST one file a `multipart/form-data` Content-Type with correct Content-Type (MIME type) set on parts or a `application/octet-stream` Content-Type with raw binary file data and the file's MIME type is derived from the file extension
  * if the filename in the `Content-Disposition` does not match the un-escaped filename in the URL, the POST is rejected with a 417 HTTP error
  * if the filename in either the `Content-Disposition` or the URL collides with an existing filename at that same URL, the POST is rejected with a 409 HTTP error

---

## Architecture

Kaya relies on fat models, service objects, and thin views. Where possible, view code is kept to a minimum in favour of "backend" code.

For architectural context from the GTK version (which this app ports), see the [kaya-gtk](https://github.com/lofimx/kaya-gtk) repository's `doc/arch/*.md`.

### Testing

* only permit a few integration tests
* only permit about 12 system tests across the entire repository
* unit tests should test models heavily, controllers lightly, and views not at all
* when fixing bugs, always try to write a failing unit test first; keep the test

### Logging

**Always add appropriate logs during development.**

**Log levels:**
- `debug` - Debug: State transitions, method entry/exit, variable values
- `info` - Info: Key milestones, user actions, important state changes
- `warn` - Warn: Unexpected but recoverable situations
- `error` - Error: Caught exceptions, failures (always include the exception)

**Where to add logs:**
- Effect handlers / side effect execution
- Repository/storage classes: Log I/O operations and results
- UI controllers: Log lifecycle events and user actions
- Caught exceptions: Always include the exception object

---

## Design

* [doc/design/](./doc/design/) contains design documentation and assets
* Visual design should follow [doc/design/DESIGN.md](./doc/design/DESIGN.md)

---

## Development Workflows

When adding a new feature:

1. Understand the domain - Read related models and tests
2. Check existing patterns - Look for similar features
3. Plan data model changes - Alert the human if a data model needs to (or will) change; this requires human consideration.
4. Implement in layers:
  * Models (business logic, validations, state machines)
  * Controllers (user input boundary)
  * Views/Components (UI)
5. Add stamps - Event tracking for visibility
6. Write tests - following existing patterns
7. Update docs - If adding new patterns/conventions:
  * keep it light
  * do not add to `doc/arch/adr-*.md` without asking
8. Never perform git commands

---

## Coding Style

* Prefer meaningful domain objects to code in services
* Extract behaviour into models
* Extract shared behaviour into methods within models
* Avoid magic numbers -- instead, you should either:
  * create constants or
  * create variables
