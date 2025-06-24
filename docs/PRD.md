# Product Requirements Document: Claude-code PWA

## Overview
This document outlines different approaches for implementing a Progressive Web App (PWA) that integrates the Claude-code backend. The app offers a chat-based UI for users to interact with Claude-code, similar to the Codex experience, where a GitHub repository acts as the environment for conversation about features and code.

## Goals
- Provide a browser-based chat interface to interact with Claude-code.
- Let users connect to a GitHub repository and discuss code changes via chat.
- Host the Claude-code backend in a Docker image with all required tools pre-installed.

## Assumptions
- The hosted Docker backend exposes an HTTP API for message passing and file operations.
- The PWA acts as a thin client, focusing on sending user messages to the backend and displaying replies.

## Approach 1: Lightweight PWA Client
1. **Frontend**
   - Built using HTML, CSS, and vanilla JavaScript or a minimal framework like Svelte.
   - Implements a chat window with message history and a text input field.
   - Utilizes service workers for offline caching and PWA capabilities.
2. **Backend**
   - A hosted Docker container running Claude-code with all the necessary tools.
   - The PWA sends JSON requests (e.g., {"message": "..."}) to the backend, and the backend returns responses.
3. **Repository Interaction**
   - The frontend passes GitHub repository details (owner/repo, branch) to the backend.
   - The backend handles `git clone`, conversation state, and commit operations.
4. **Pros**
   - Simple architecture with minimal dependencies on the client side.
   - Easy to deploy and maintain.
5. **Cons**
   - Limited UI features compared to richer frameworks.
   - More responsibility placed on the backend for repository state and complex interactions.

## Approach 2: Framework-based PWA (React or Vue)
1. **Frontend**
   - Uses React or Vue for a richer component-based UI.
   - Includes chat history, file browser, and commit view.
   - Service worker for offline support and push notifications.
2. **Backend**
   - Same hosted Docker container running Claude-code.
   - API endpoints for chat messages, repository checkout, and commit actions.
3. **Repository Interaction**
   - The frontend may provide a file tree view using GitHub API data.
   - More interactivity, such as diff view or commit messages, on the client side.
4. **Pros**
   - Better user experience with dynamic components.
   - Clear separation of concerns between UI and backend logic.
5. **Cons**
   - Slightly heavier client footprint.
   - Requires build tooling (Webpack/Vite) for bundling.

## Approach 3: Server-driven UI
1. **Frontend**
   - Minimal HTML and JavaScript; relies on server-rendered templates.
   - Chat messages are delivered as server-sent events or websockets.
2. **Backend**
   - Hosted Docker image also handles template rendering.
   - Claude-code integration generates both the message text and any UI hints.
3. **Repository Interaction**
   - More logic is server-side; the client becomes a dumb terminal.
4. **Pros**
   - Very thin client with low maintenance.
   - All logic centralized in the backend.
5. **Cons**
   - Less flexibility in UI/UX.
   - Server must manage sessions and rendering, increasing complexity there.

## Recommendations
- **Start with Approach 1**: A lightweight client will get the product running quickly and keep the backend-focused architecture straightforward.
- Consider moving to a framework-based approach (Approach 2) if richer UI features like file browsing become necessary.
- The server-driven approach (Approach 3) could be used for a rapid prototype but may limit future enhancements.

## Next Steps
1. Decide on the initial approach.
2. Define the API contract between frontend and backend.
3. Outline the architecture for the Docker-based Claude-code backend.
4. Build a minimal prototype to validate chat interactions and repository integration.
