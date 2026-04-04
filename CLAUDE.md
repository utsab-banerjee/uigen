# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev          # Start dev server with Turbopack at localhost:3000
npm run build        # Production build
npm run lint         # ESLint via Next.js

# Testing
npm run test         # Run all Vitest tests
npx vitest run src/path/to/file.test.ts  # Run a single test file

# Database
npm run setup        # Install deps + generate Prisma client + run migrations
npm run db:reset     # Reset database (destructive)
npx prisma studio    # Open Prisma GUI

# Background dev server
npm run dev:daemon   # Start server in background, logs to logs.txt
```

All dev/build commands require the `NODE_OPTIONS='--require ./node-compat.cjs'` prefix (already baked into npm scripts) for Node.js compatibility.

## Environment

Add an `ANTHROPIC_API_KEY` to `.env` to use real Claude AI. Without it, the app falls back to a mock provider (see `src/lib/provider.ts`).

## Architecture

UIGen is an AI-powered React component generator. Users describe UI in a chat, Claude generates code, and the result is rendered live in an iframe.

### Data Flow

1. User sends message → `POST /api/chat` (streaming)
2. Claude responds using two tools: `str_replace_editor` (create/edit files) and `file_manager` (rename/delete)
3. Tool calls update the `VirtualFileSystem` (in-memory, no disk writes)
4. Preview iframe re-renders by transpiling files via `@babel/standalone` at runtime
5. Project state (chat history + file system) serializes to JSON and persists in SQLite via Prisma

### Key Abstractions

**`VirtualFileSystem`** (`src/lib/file-system.ts`) — In-memory file tree that serializes to JSON for database storage. All file operations go through this class.

**`FileSystemContext`** (`src/lib/contexts/file-system-context.tsx`) — React context wrapping `VirtualFileSystem`, exposes file CRUD operations and syncs state to the database.

**`ChatContext`** (`src/lib/contexts/chat-context.tsx`) — Manages conversation history and streams AI responses from `/api/chat`. Handles tool call results and updates the file system.

**`provider.ts`** (`src/lib/provider.ts`) — Abstracts AI model selection. Returns real Anthropic Claude (`claude-haiku-4-5`) when `ANTHROPIC_API_KEY` is set, mock otherwise.

**`PreviewFrame`** (`src/components/preview/PreviewFrame.tsx`) — Renders the virtual file system in a sandboxed iframe. Uses `@babel/standalone` to transpile JSX at runtime, resolving imports across virtual files.

### AI Tools (called by Claude during generation)

- **`str_replace_editor`** (`src/lib/tools/str-replace.ts`) — Creates new files or applies string-replacement edits to existing ones
- **`file_manager`** (`src/lib/tools/file-manager.ts`) — Renames or deletes files in the virtual FS

### Authentication

JWT-based auth using JOSE + HTTP-only cookies. Anonymous users can create projects (tracked in `localStorage` via `anon-work-tracker.ts`). `src/middleware.ts` protects routes and handles anonymous → registered user migration.

### Database Schema

Two models in SQLite (`prisma/dev.db`):
- `User` — email/password accounts
- `Project` — stores `messages` (chat history as JSON string) and `data` (virtual FS as JSON string); `userId` is nullable to support anonymous projects

Prisma client is generated to `src/generated/prisma`.

### UI Structure

Resizable two-panel layout: chat on the left, tabbed preview/code editor on the right. shadcn/ui components (new-york style, neutral base) with Lucide icons. Path alias `@/*` maps to `src/*`.
