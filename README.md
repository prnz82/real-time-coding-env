# Real-Time Collaborative Code Editor

A small React-based collaborative editor that uses an event-driven, socket-based architecture to synchronize edits in real time. The system uses a Last-Write-Wins (LWW) conflict resolution strategy so concurrent edits converge simply and predictably.

## Key Features

- Event-driven, socket-based synchronization between clients and server
- Real-time collaboration with per-change events and cursor updates
- Last-Write-Wins (LWW) conflict resolution for concurrent edits
- Minimal React frontend and a lightweight Node socket server (see `socket.js`)

## Architecture Overview

The app is organized as a client-server system:

- Clients: React app in `src/
- Server: simple socket endpoint

Communication is event-driven over WebSocket (or socket.io): clients emit change events to the server, the server broadcasts events to other clients, and clients apply changes locally.

Events (typical):

- `connect` — client connected
- `init` — initial document state
- `change` — a single edit/change event (payload includes `ops`, `clientId`, `timestamp`)
- `cursor` — user cursor/selection updates
- `ack` — server acknowledgement of a received change
- `presence` — user join/leave notifications

Each `change` event includes a logical timestamp (e.g., ISO string or epoch ms) and a `clientId`. The system relies on timestamps to apply Last-Write-Wins.

## Conflict Resolution — Last-Write-Wins (LWW)

LWW is intentionally simple and deterministic: when two or more conflicting edits target the same region, the edit with the latest timestamp wins and is accepted as the canonical state. Implementation notes:

- Clients timestamp outgoing changes with local time; the server may overwrite with server time to avoid clock skew.
- On receiving a `change`, the server compares the incoming change timestamp with the currently-applied timestamp for the affected range/object. If the incoming timestamp is newer, the server accepts and broadcasts it; otherwise it's rejected or ignored.
- The server should emit `ack` (accepted/rejected) so the originating client can reconcile optimistic UI state.

Tradeoffs: LWW is simple and works well for line/field-level edits or systems where last update semantics are acceptable. It is not suitable for fine-grained OT/CRDT-style merging where intent preservation is required.

## Getting Started (development)

Prerequisites: Node.js (14+ recommended) and npm or yarn.

Install dependencies:

```bash
npm install
```

Run the dev server (React + socket server if available):

```bash
npm start
```

Build for production:

```bash
npm run build
```

Notes: Ensure backend is running alongside the React dev server or configured as a proxy in `package.json`.

## Development Notes

- Look at `src/socket.js` for the client socket wrapper and `src/components/Editor.js` for how change events are produced and applied.
- Ensure every change event contains `clientId` and `timestamp` fields to support LWW.
- Consider normalizing timestamps to server time on receipt to reduce clock skew issues.
- For more robust merges consider CRDTs or Operational Transform if you need to preserve concurrent edits rather than prefer last write.

## Testing & Debugging

- Open multiple browser windows to simulate concurrent editors.
- Observe `change`, `ack`, and `presence` events in the browser console (or server logs) to verify behavior.

## Contributing

Contributions are welcome. Please open issues for bugs or feature requests and submit pull requests with focused changes.


