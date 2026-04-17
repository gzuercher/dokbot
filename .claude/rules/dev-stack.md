---
description: Tech Stack, Build-Commands und Projektstruktur für Dokbot
globs: "*.ts,*.tsx,*.js,*.jsx,package.json,tsconfig.json"
---

# Entwicklungs-Stack

## Tech Stack

- Runtime: Node.js (>=20)
- Sprache: TypeScript (strict mode)
- Paketmanager: pnpm
- Monorepo: Turborepo
- Linting: ESLint, Prettier
- Tests: vitest
- Validierung: zod

### Core Dependencies
- LLM-Abstraktion: Vercel AI SDK (`ai`)
- Vektor-Index: LanceDB
- Git: isomorphic-git
- CLI: Commander.js
- MCP: @modelcontextprotocol/sdk
- API: Hono

## Build & Test Commands

```bash
pnpm install        # Install dependencies
pnpm dev            # Development (watch mode)
pnpm build          # Production build
pnpm test           # Run tests
pnpm lint           # Linting
pnpm format         # Prettier
```

## Monorepo-Struktur

```
packages/
├── core/             # Dokbot Core: LLM-Agent, Git-Storage, Indexing
├── cli/              # CLI: Commander.js basiert
└── mcp-server/       # MCP-Server: Knowledge Base als MCP-Ressource
```

## Verifikation

Prüfe JEDE Änderung bevor du sie als fertig meldest:
1. Build muss durchlaufen (`pnpm build`)
2. Linting muss bestehen (`pnpm lint`)
3. Tests müssen grün sein (`pnpm test`)

## Verbotene Eigenimplementierungen

| Thema | Verwende stattdessen |
|---|---|
| LLM-Aufrufe | Vercel AI SDK (nie direkt fetch auf LLM-APIs) |
| Git-Operationen | isomorphic-git (nie shell-exec von git) |
| Validierung | zod |
| Vektor-Suche | LanceDB |
| MCP-Protokoll | @modelcontextprotocol/sdk |
