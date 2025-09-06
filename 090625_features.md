# Collaborative Editor Feature Backlog

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

------------------------------------------------------------------------

## Nice-to-Have

-   **Visual keybinding hints**
    -   Optional on-screen indicator of recent key presses or a help
        overlay so collaborators learn shortcuts faster.
-   **GitHub integration helpers**
    -   From the editor, provide links or prefilled templates to "Create
        new issue referencing #old," since direct reopen may be blocked
        on personal repos.
