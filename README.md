# Oracle960 — Master Architecture Blueprint

> **Document purpose:** Architecture and technology decisions for Oracle960 — a multi-user sovereign AI platform built on Cloudflare's latest stack.
>
> **Status:** Active development, Phase 1 complete.

---

## Architecture Overview

```
Browser ──Web──→ Astro SSR (Landing / Admin UI)
                    │ WebSocket
                    ▼
           Think Agent (Per-User Durable Object)
                    │ HTTP / MCP
                    ▼
           Admin Agent (Privileged Operations)
                    │
           ┌────────┴────────┐
           D1 / Vectorize    R2 Buckets
           (Memory Store)    (File Storage)

External Compute Nodes (Surface, Android, Mac)
        └────── WebSocket / MCP ──────┘
```

### Key Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Framework | Astro 6.3+ | Cloudflare-owned, SSR via Workers, Islands for interactivity |
| Agent Runtime | Think (Cloudflare Agents SDK) | Per-user Durable Objects, built-in SQLite, zero idle cost |
| Memory | D1 → Agent Memory (future) | D1 works now; swappable to managed Agent Memory when available |
| Frontend Chat | Astro Island → WebSocket → Think | Vanilla JS WebSocket client; no React/Vite dependency |
| Auth | GitHub OAuth → email OTP | Start with GitHub, add email OTP for non-technical users |
| Node Network | WebSocket → MCP | Compute nodes register via WebSocket, Think proxies exec calls |
| Scale | 1 Durable Object per user | Thousands of users for pennies (MoltWorker was $35/user/mo) |

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Runtime | Cloudflare Workers | `nodejs_compat`, Workers Paid plan |
| Agent SDK | @cloudflare/think | Latest preview — Durable Object agents |
| AI Models | OpenCode Go | Primary model provider via API |
| Frontend | Astro 6.3+ | SSR via Cloudflare adapter, static output for content pages |
| Database | D1 (now) → Agent Memory (future) | SQLite-compatible, per-user isolation via agent_id scoping |
| Vector Search | Vectorize | 1024d cosine index, embeddings via Workers AI |
| Object Storage | R2 | User file uploads, backups |
| Auth | GitHub OAuth + Cloudflare Access | Access for admin UI, OAuth for user sign-in |
| CI/CD | Wrangler CLI + GitHub | Automated deploy via `npx wrangler deploy` |

---

## Repository Structure

```
oracle960/
├── src/
│   ├── agents/
│   │   ├── user-agent.ts       # Think Agent (per-user Durable Object)
│   │   └── admin-agent.ts      # Admin agent (privileged operations)
│   ├── routes/
│   │   ├── chat.ts             # WebSocket chat handler
│   │   ├── auth.ts             # OAuth handler
│   │   └── nodes.ts            # Node heartbeat + registration
│   ├── tools/
│   │   ├── memory.ts           # D1/Vectorize → Agent Memory adapter
│   │   ├── search.ts           # Web search, knowledge retrieval
│   │   └── credits.ts          # Credit system
│   ├── db/
│   │   └── schema.sql          # D1 tables
│   └── server.ts               # Worker entry point
├── astro/
│   ├── src/
│   │   ├── pages/              # Landing, Alfred chat, Auth, Admin
│   │   ├── components/         # Chat island, UI components
│   │   └── layouts/            # Base layout
│   └── astro.config.ts
├── wrangler.toml
└── package.json
```

---

## Development Phases

### Phase 0 — Foundation (Complete)
- Workers project setup with D1, Vectorize, AI, R2 bindings
- D1 schema applied, secrets configured
- Basic worker deployed with health check, memory push, node heartbeat endpoints

### Phase 1 — Think Agent + Domain Routing (Complete)
- Durable Object (UserAgent) extending Agent base class
- Custom chat logic with OpenCode Go integration
- Domain routing applied
- Astro static site builds without SSR adapter dependency

### Phase 2 — Multi-User Auth (Current)
- GitHub OAuth flow
- User table with tier/credit system
- Per-user data isolation via agent_id scoping
- Session management

### Phase 3 — Memory Backend
- D1 + Vectorize semantic search
- Memory adapter with Agent Memory API-compatible interface
- Cross-session recall

### Phase 4 — Node Network
- Compute node registration via WebSocket
- Task routing from Think agents to local devices
- Existing node agent scripts updated

### Phase 5 — Admin & Billing
- Admin Durable Object with user management
- Credit system integration
- Admin dashboard (Cloudflare Access protected)

### Phase 6 — Agent Memory Swap
- When private beta access granted: swap D1 backend for Agent Memory managed service
- Migration of existing data

---

## License

MIT — Architecture documentation only. No implementation code included.

For the Oracle960 project source code, see the main repository.
