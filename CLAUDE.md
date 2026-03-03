# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server (Turbopack)
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Vitest
npm run setup        # npm install + prisma generate + prisma migrate dev
npm run db:reset     # Hard reset Prisma database
```

Run a single test file: `npx vitest run src/components/chat/__tests__/MessageList.test.tsx`

## Architecture

**UIGen** is an AI-powered React component generator with live preview. Users describe components in natural language; Claude generates them in real-time.

### Key Data Flow

1. User sends a message in the chat
2. `/api/chat` reconstructs the `VirtualFileSystem` from the client's state
3. Claude receives a system prompt + chat history and uses tools (`str_replace_editor`, `file_manager`) to create/modify files
4. Streamed changes update the in-memory VFS on the client
5. On completion, the project is saved to the database (authenticated users only)

### Core Contexts

- `lib/contexts/chat-context.tsx` — Chat state, message handling, streaming
- `lib/contexts/file-system-context.tsx` — In-memory file system state shared across panels

### Virtual File System

`lib/file-system.ts` — In-memory file tree (no disk writes). Serialized to JSON for database persistence and sent back and forth with the API.

### AI Tools

- `lib/tools/str-replace.ts` — `str_replace_editor` tool: creates/edits files via string replacement
- `lib/tools/file-manager.ts` — `file_manager` tool: create/delete/move files

### Provider

`lib/provider.ts` — Wraps Anthropic Claude (claude-haiku-4-5). Falls back to a mock provider that returns static components when `ANTHROPIC_API_KEY` is not set.

### UI Layout

`app/main-content.tsx` — Resizable horizontal split:
- Left (35%): Chat interface (`components/chat/`)
- Right (65%): Preview tab (live sandbox) / Code tab (file tree + Monaco editor)

### Authentication

JWT sessions stored in HttpOnly cookies (7-day expiry). `lib/auth.ts` handles token creation/verification. `src/middleware.ts` protects API routes. Server actions in `src/actions/` handle sign-up, sign-in, sign-out.

### Database

SQLite + Prisma. Schema in `prisma/schema.prisma`. Generated client at `src/generated/prisma`. Two models: `User` and `Project` (stores JSON-stringified messages and file system data).

### Code Transformation

`lib/transform/jsx-transformer.ts` — Babel standalone transforms JSX in the browser for live preview. Handles missing imports and CSS detection.

### Path Alias

`@/*` maps to `src/*`. Generated components use `@/` imports.

## Code Style

- Use comments sparingly. Only comment complex code.

## Environment

- `ANTHROPIC_API_KEY` — Optional; falls back to mock provider without it
- Database: `prisma/dev.db` (SQLite, local)
