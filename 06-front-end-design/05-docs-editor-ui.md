# FE 05 · Collaborative Document Editor (front end)

| Difficulty | Key ideas |
|---|---|
| Hard | own document model behind contenteditable, transactions, CRDT or OT sync, remote cursors mapped through changes, offline edits, undo per user |

## Requirements
- Rich text: headings, lists, bold, links, images, tables.
- Several people edit the same document live and see each other's cursors.
- Offline edits merge when back online.
- Undo/redo of your own changes only.

## Why not just use contenteditable?
Browsers produce inconsistent HTML from contenteditable, and you can't reliably sync DOM changes between users. Instead keep a **structured document model** (a tree of nodes with marks) and treat the DOM as a view of it. Editors like ProseMirror, Lexical and Slate work this way.

## Architecture
```mermaid
flowchart LR
    DOM["contenteditable view"] -->|"input events (beforeinput)"| CMD["Commands (insert text, toggle bold)"]
    CMD --> TR["Transaction (steps)"]
    TR --> MODEL["Document model (tree)"]
    MODEL --> REN["Renderer (model to DOM, minimal updates)"]
    REN --> DOM
    TR --> SYNC["Sync layer (CRDT like Yjs, or OT with server)"]
    SYNC <-->|"WebSocket"| SRV["Collaboration server"]
    SYNC --> IDB[("IndexedDB: local updates for offline")]
    SYNC --> AW["Awareness: remote cursors and selections"]
```

## Editing flow
```mermaid
sequenceDiagram
    participant U as User
    participant E as Editor
    participant Y as Sync (CRDT)
    participant S as Server
    participant O as Other user
    U->>E: types "a" at position 10
    E->>E: beforeinput -> transaction -> update model -> render
    E->>Y: apply change to shared doc
    Y->>S: send update (binary, batched)
    S->>O: broadcast update
    O->>O: merge, re-render, remap their cursor
```

## Key details
- **Positions move:** when remote text is inserted before your cursor, map your selection through the change so it stays on the same character. CRDTs use relative positions anchored to characters, not indexes.
- **Remote cursors:** awareness messages (user, color, selection) are throttled and not persisted.
- **Undo:** track only local transactions; undo produces an inverse change that is synced like any other.
- **Offline:** keep CRDT updates in IndexedDB; on reconnect, exchange state vectors and send only missing updates.
- **IME and mobile keyboards:** handle composition events carefully; don't re-render mid-composition.
- **Large documents:** render only visible blocks; lazy-load images.
- **Paste:** sanitize HTML into model nodes, never insert raw HTML.

## Interview Q&A

**Q: How do two people typing in the same sentence end up with the same text?**
A: Changes go through a CRDT (or the server's OT transform), which merges concurrent inserts deterministically on every client.

**Q: Why keep a separate model?**
A: It gives predictable structure, makes syncing and undo possible, and lets you control rendering instead of relying on browser quirks.

**Related:** [Google Docs (backend)](../03-problems/35-google-docs.md) · [Collaborative whiteboard](../03-problems/75-collaborative-whiteboard.md)
