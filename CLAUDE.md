# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev           # Start dev server with Turbopack (requires node-compat.cjs)
npm run dev:daemon    # Start dev server in background, logs to logs.txt

# Build & Lint
npm run build
npm run lint          # next lint

# Testing
npm run test                          # Run all tests
npm run test -- src/path/to/file.test.tsx  # Run a single test file

# Database
npm run setup         # Install deps + generate Prisma client + run migrations
npm run db:reset      # Reset database (destructive)
```

## Environment Variables

- `ANTHROPIC_API_KEY` — required for real Claude API calls (omit to use mock provider)
- `JWT_SECRET` — defaults to `"development-secret-key"` if unset

## Architecture

UIGen is a Next.js 15 (App Router) application that lets users describe React components in natural language and generates them via Claude AI with a live preview.

### Core Data Flow

```
User message → ChatContext → POST /api/chat
  → Claude (streamText) → tool calls (str_replace_editor, file_manager)
  → FileSystemContext updates → PreviewFrame re-renders iframe
```

The chat API route (`src/app/api/chat/route.ts`) drives all component generation. It streams responses from Claude using the Vercel AI SDK and exposes two tools to the model:
- **`str_replace_editor`** (`src/lib/tools/str-replace.ts`) — patch existing file contents
- **`file_manager`** (`src/lib/tools/file-manager.ts`) — create/delete/rename files

### Virtual File System

There is no disk I/O for generated files. `VirtualFileSystem` (`src/lib/file-system.ts`) holds all files in memory. `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) exposes this to the component tree.

### Preview Rendering

`PreviewFrame` (`src/components/preview/PreviewFrame.tsx`) renders an iframe. `jsx-transformer.ts` (`src/lib/transform/jsx-transformer.ts`) transforms virtual JSX files to runnable HTML using `@babel/standalone` before injecting them into the iframe.

### Authentication & Persistence

Auth uses JWT sessions (`src/lib/auth.ts`) + bcrypt. Middleware (`src/middleware.ts`) guards `/[projectId]` routes. When a user is authenticated, the chat API saves the full chat history and virtual file system state as JSON into the `Project` table (SQLite via Prisma).

### State Management

Two React contexts handle all app state:
- **`FileSystemContext`** — virtual FS, selected file, file CRUD operations
- **`ChatContext`** — messages, streaming state, calls to `/api/chat`

### Layout

`MainContent` (`src/app/main-content.tsx`) is a two-panel resizable layout (react-resizable-panels): chat on the left (~35%), tabs with Preview/Code on the right (~65%).

### AI Provider

`src/lib/provider.ts` returns either the real Anthropic model or a mock, depending on whether `ANTHROPIC_API_KEY` is set. The system prompt lives in `src/lib/prompts/generation.tsx`.

## Tech Stack

- **Framework:** Next.js 15, React 19, TypeScript 5
- **Styling:** Tailwind CSS v4, Radix UI primitives
- **Editor:** Monaco Editor (`@monaco-editor/react`)
- **AI:** Vercel AI SDK + `@ai-sdk/anthropic`
- **Database:** SQLite via Prisma
- **JSX transform:** `@babel/standalone` (client-side, in-browser)
- **Testing:** Vitest + React Testing Library + jsdom
- **Path alias:** `@/*` → `src/*`
