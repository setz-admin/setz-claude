# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**UIGen** ist ein KI-gestützter React-Komponenten-Generator. Nutzer beschreiben Komponenten per Chat, Claude generiert Code in einem virtuellen Dateisystem, und eine Live-Vorschau rendert das Ergebnis über Babel im Browser.

## Commands

```bash
npm run setup       # Erstinstallation: dependencies + Prisma generate + migrations
npm run dev         # Dev-Server starten (Turbopack)
npm run build       # Produktions-Build
npm run lint        # ESLint ausführen
npm test            # Vitest Test-Suite ausführen
npm run db:reset    # Datenbank zurücksetzen (Prisma)
```

Einzelnen Test ausführen:
```bash
npx vitest run src/components/chat/__tests__/ChatInterface.test.tsx
```

## Architecture

### Datenfluss

```
User Chat → /api/chat (streaming) → Claude Haiku via Vercel AI SDK
  → Tools: str_replace_editor / file_manager
  → VirtualFileSystem (in-memory, kein Disk-I/O)
  → Preview: Babel-Transpilation im Browser
  → DB: SQLite via Prisma (nur für eingeloggte User)
```

### Kernmodule

**`/src/lib/file-system.ts`** — Virtuelles Dateisystem (Map-basiert, JSON-serialisierbar). Alle generierten Dateien leben hier, nie auf der Festplatte.

**`/src/lib/tools/`** — AI-Tool-Implementierungen (`str_replace`, `file_manager`) als Zod-validierte Vercel AI SDK Tools. Sie erhalten eine VirtualFileSystem-Instanz.

**`/src/lib/provider.ts`** — Wählt zwischen Anthropic-API und Mock-Provider (Fallback wenn kein `ANTHROPIC_API_KEY` gesetzt).

**`/src/app/api/chat/route.ts`** — Streaming-Endpunkt. Verwaltet den Chat-Loop mit Claude, maximale Request-Dauer: 120s.

**`/src/components/preview/PreviewFrame.tsx`** — Rendert generierten Code mit `@babel/standalone` in einem isolierten Context im Browser.

**`/src/lib/contexts/`** — `FileSystemContext` und `ChatContext` als zentrale State-Manager für die Client-seitige App.

### Layout der App

- **Linkes Panel (35%)**: Chat-Interface
- **Rechtes Panel (65%)**: Tabs — Preview (Babel-Sandbox) | Code (Dateibaum + Monaco-Editor)

### AI-Konventionen (System-Prompt in `/src/lib/prompts/`)

Claude erstellt immer `/App.jsx` als Einstiegspunkt, Komponenten in `/components/`. Ausschließlich Tailwind CSS, kein Inline-Styling. Import-Alias `@/` für lokale Imports.

## Environment Variables

```
ANTHROPIC_API_KEY   # Optional – ohne diesen Key wird der Mock-Provider genutzt
JWT_SECRET          # Optional – Default: "development-secret-key"
```

## Tech Stack

- **Next.js 15** (App Router, Server Components, Turbopack)
- **Vercel AI SDK 4** mit `@ai-sdk/anthropic`
- **Prisma 6** + SQLite (`prisma/dev.db`)
- **Tailwind CSS v4**, shadcn/ui (New York Style), Radix UI
- **Monaco Editor** für Code-Bearbeitung
- **Vitest** + Testing Library (jsdom) für Tests
- TypeScript strict mode, Path-Alias `@/*` → `./src/*`

## Auth

JWT in httpOnly-Cookies (7 Tage), Passwörter mit bcrypt gehasht. Session-Handling in `/src/lib/auth.ts`, Server Actions in `/src/actions/`.