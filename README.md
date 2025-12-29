# Real-Time Collaborative Code Editor

A small React-based collaborative editor that uses an event-driven, Socket.io-based architecture to synchronize code in real time. The system uses a Last-Write-Wins (LWW) approach so the most recently received update from the server becomes the canonical state for all clients.

## Key Features

- Event-driven synchronization between clients and server using Socket.io
- Real-time collaboration with full-document synchronization
- Last-Write-Wins (LWW) conflict resolution for concurrent updates
- Minimal React frontend and a lightweight Node.js server (see `server.js`)

## Architecture Overview

The app is organized as a client-server system:

- **Clients**: React application in `src/` using `socket.io-client`.
- **Server**: Node.js/Express server in `server.js` using `socket.io` to manage rooms and broadcasting.

Communication is event-driven over Socket.io: clients emit updates to the server, the server broadcasts these updates to other clients in the same room, and clients apply the changes locally.

### Key Events (`src/Actions.js`):

- `join` — Client requests to join a specific room.
- `joined` — Server notifies clients that a new user has joined the room.
- `code-change` — Emitted when the editor content changes (payload includes the full `code` string).
- `sync-code` — Used to synchronize the current code state with a newly joined client.
- `disconnected` — Notifies the room when a user leaves.

## Conflict Resolution — Last-Write-Wins (LWW)

This project implements a simplified LWW strategy:

- The server acts as the single source of truth for the broadcast order.
- When multiple clients edit the code, the server broadcasts the updates in the order they are received.
- Each client applies the incoming `code-change` event by overwriting their local editor state.
- Because the server processes and broadcasts sequentially, the "last write" to reach the server effectively "wins" and synchronizes across all clients.

**Tradeoffs**: This approach is simple and easy to implement but can lead to "overwriting" if two users type in different locations at the exact same time, as the entire document is synchronized rather than granular edits.

## Getting Started (development)

Prerequisites: Node.js (14+ recommended) and npm or yarn.

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Run the development environment**:
   - Start the frontend: `npm run start:front`
   - Start the server: `npm run server:dev`

3. **Build for production**:
   ```bash
   npm run build
   npm start
   ```

## Development Notes

- Look at `src/socket.js` for the client-side Socket.io initialization.
- View `server.js` for the backend room management and broadcasting logic.
- The editor is powered by CodeMirror (`src/components/Editor.js`), which handles the rendering and change detection.

## Testing & Debugging

- Open multiple browser windows to simulate concurrent editors.
- Inspect the network tab or console logs to see the `code-change` events being emitted and received.

## Contributing

Contributions are welcome. Please open issues for bugs or feature requests and submit pull requests with focused changes.


