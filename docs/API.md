# API Documentation for UIGen

UIGen’s backend is built with **Next.js API Routes** and primarily exposes a single endpoint that powers the AI‑driven component generation workflow. Below you will find the request/response contracts, authentication details, and error handling guidelines.

---

## Base URL
```
https://<your‑deployment>.vercel.app/api
```
All paths are relative to this base URL.

---

## Endpoints

### `POST /chat`
**Purpose**: Accept a conversation payload, invoke the Anthropic Claude model (or a mock provider), and stream back the updated virtual file system along with any new AI messages.

#### Request
- **Headers**
  - `Content-Type: application/json`
  - Optional `Authorization: Bearer <session‑token>` – required only when you want the server to persist the result to a user project.
- **Body** (JSON)
```json
{
  "messages": [
    { "role": "user", "content": "Create a button component" }
  ],
  "files": {
    "/App.jsx": {
      "content": "export default function App() { return <div />; }",
      "type": "file"
    }
  },
  "projectId": "optional‑project‑uuid"
}
```
- `messages` – an array of chat messages in the format expected by the Vercel AI SDK. The first element will be prepended with the system prompt defined in `src/lib/prompts/generation.tsx`.
- `files` – a map of file paths to their serialized `FileNode` representation (see `src/lib/file-system.ts`). This allows the AI to read and modify the virtual file system.
- `projectId` – (optional) the UUID of a persisted project. If supplied **and** the request is authenticated, the server will update the corresponding Prisma record with the new messages and file data.

#### Response
The endpoint streams a **text/event‑stream** compatible response (compatible with the Vercel AI SDK `streamText`). The stream contains JSON fragments that represent:
1. Updated file system state (`files` map).
2. Any new AI messages generated during the turn.

Typical client‑side handling (using the Vercel AI SDK) looks like:
```ts
import { streamText } from "ai";
const result = await streamText({
  url: "/api/chat",
  method: "POST",
  body: payload,
});
await result.toDataStreamResponse();
```

#### Errors
| Status | Condition | Response Body |
|--------|-----------|---------------|
| `401` | Missing or invalid session when `projectId` is provided. | `{ "error": "Unauthorized" }` |
| `404` | `projectId` does not belong to the authenticated user. | `{ "error": "Project not found" }` |
| `500` | Unexpected server error (e.g., Prisma failure, AI provider error). | `{ "error": "Internal Server Error" }` |

All error responses are JSON with an `error` field.

---

## Authentication
UIGen uses **session‑based authentication** via the `getSession` helper (`src/lib/auth.ts`). When a request includes a valid session cookie (or Bearer token, depending on your auth implementation), `getSession` resolves to an object containing at least `userId`. The endpoint only attempts to persist data when both `projectId` is present **and** a valid session is found.

If you are calling the API from the front‑end within the same Next.js app, the session cookie is automatically sent. For external clients, include the `Authorization` header.

---

## Data Model (Prisma)
The relevant Prisma model is defined in `prisma/schema.prisma`:
```prisma
model Project {
  id        String   @id @default(uuid())
  name      String
  userId    String
  messages  String   // JSON string of chat history
  data      String   // JSON string of serialized virtual file system
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```
- `messages` stores the full conversation history.
- `data` stores the virtual file system (output of `VirtualFileSystem.serialize()`).

---

## Local Development
When running locally without an Anthropic API key, the server automatically switches to **mock mode**:
- `maxSteps` is limited to 4 to avoid repetitive output.
- The AI provider returns deterministic placeholder code.

This allows developers to work on the UI and file‑system logic without incurring API costs.

---

## Extending the API
If you need additional endpoints (e.g., project listing, deletion, or user profile), follow the same pattern:
1. Create a file under `src/app/api/<resource>/route.ts`.
2. Export the HTTP method handlers (`GET`, `POST`, `DELETE`, …).
3. Use the `prisma` client for database interaction and `getSession` for auth.
4. Return JSON responses or streamed data as appropriate.

---

## License
This API documentation is part of the **UIGen** project and is licensed under the MIT License. See the repository’s `LICENSE` file for details.
