# Collaborative Editor & PromiseGrid Feature Backlog

## Must-Have

-   **Reliable line-number toggle**
    -   Fix bug where line numbers remain visible after the
        "hidden/shown" notification.
    -   Ensure margins/gutters auto-adjust when toggling and let
        CodeMirror own spacing.
-   **Document export with stable naming**
    -   "Download as Text" flow exists; define consistent filename rules
        (not just page title / reload state).
-   **Room/document identifiers to prevent collisions**
    -   IndexedDB currently keys by "room name," which risks collisions
        as usage scales; introduce unique, stable IDs (keep human names
        as metadata).
    -   JJ: Store docs in IndexedDB in a flat namespace (no folders),
        with UUID as the key and human-readable doc name as a label.
    -   Add `provider.set` and `provider.get` for room name labels.
    -   Need to manage a separate index or search mechanism for doc
        lookup.
-   **Basic test coverage for core UI behaviors**
    -   Add/repair Playwright tests specifically for the line-number
        toggle so regressions are caught early.
-   **Environment/ports clarity for local runs**
    -   Document expected processes/ports so contributors can run/debug
        consistently: Rust backend on 3000, Node on 8080, Yjs WebSocket
        on 1234.

------------------------------------------------------------------------

## Should-Have

-   **Transient status notifications**
    -   Keep brief success/failure toasts for actions like toggling line
        numbers (auto-dismiss after \~2s).
-   **Issue hygiene & workflow nudges**
    -   Encourage closing resolved issues quickly and linking forward
        when reopening isn't permitted (or repo policy prohibits).
    -   Consider lightweight prompts or templates in PR/issue flows.
-   **Top-of-doc date field**
    -   Add a visible date field/header in shared docs/notes for quicker
        context during sessions.
-   **HOWTO Video for AIDDA**
    -   Needed to make AIDDA usable.
    -   Decision: Steve can make this, or JJ can tackle it.

------------------------------------------------------------------------

## Nice-to-Have

-   **Visual keybinding hints**
    -   Optional on-screen indicator of recent key presses or a help
        overlay so collaborators learn shortcuts faster.
-   **GitHub integration helpers**
    -   From the editor, provide links or prefilled templates to "Create
        new issue referencing #old," since direct reopen may be blocked
        on personal repos.
-   **Draggable panels for markdown and activity log**
    -   UI enhancement for better layout and workspace organization.

------------------------------------------------------------------------

## Open Philosophical / Design Questions

-   **Metaphor for Data Organization**
    -   Should PromiseGrid use a **folder/file metaphor** or a
        **space/room/resource metaphor**?
    -   Who "owns" the root directory if folder/file is used
        (centralization concern)?
    -   Consider **Plan 9 style**: each process/user has their own
        directory tree.
    -   VMS precedent: `filename;version` for versioning.
-   **Handling Versions**
    -   File/folder metaphors don't support versions well.
    -   Explore explicit versioning schemes (like VMS).
-   **Metaphor Choice Implications**
    -   Space/room/resource metaphor allows resources to include docs,
        chats, computing services.
    -   Virtual world metaphor is currently favored by Steve.
    -   Punt option: make persistence/naming per-application, but risk
        fragmentation.
-   **Libraries & Layers**
    -   Provide standard libraries or layers supporting chosen access
        metaphors.
    -   Encourage app developers to adopt consistent practices.
