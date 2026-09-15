# Crowfoot

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

**[한국어](./README.md)** | **English**

**Where design ends, your database begins.**

Crowfoot is an open-source ERD editor that runs in your browser. From logical modeling to physical schema, team collaboration, and real database provisioning — every step of your data work in one place.

## Services

| Service | Address |
| --- | --- |
| Web (editor · dashboard) | https://crowfoot.java21.net |
| API gateway | https://crowfoot-api.java21.net |
| Collaboration WebSocket server | crowfoot-ws.java21.net |

## Free Managed Databases

The heart of Crowfoot. Spin up **up to 5 free PostgreSQL or MySQL development databases per account**.

- Each database comes with a **dedicated DB account** provisioned automatically — schema-scoped privileges, with instance root credentials never exposed anywhere
- Issued passwords are encrypted at rest (AES-256-GCM); only the owner can view the credentials
- Test the connection right after issuing and connect from any external client — revoke it whenever you're done
- Databases are intended for development, learning, and testing

## Features

### Browser ERD editor

- No install, no setup — draw tables and relationships with Crow's Foot notation, right in your browser
- **Logical and physical models together** — logical/physical names side by side, common logical types auto-mapped to DBMS-specific physical types
- **Five DBMS templates** — PostgreSQL, MySQL, Oracle, MSSQL (+ common), each reflecting its own type, auto-increment, and comment syntax
- Model validation (duplicate names, referential integrity, …), auto layout, and per-document viewport (zoom & pan) memory
- Smooth 60fps panning and zooming even on large documents (100 tables)

### SQL generation, deployment & reverse engineering

- Turn a document into a **DDL script for its DBMS** — preview, copy, or download (table comments follow logical names)
- **Forward engineering** — deploy the generated DDL straight to an issued or registered database
- **Reverse engineering** — read an existing database's schema and comments back into an ERD document

### Real-time collaboration

- Built on WebSocket (STOMP) — edit the same document together with live presence and real-time change sync
- Leave comments on documents

### Workspaces & team roles

- Organize documents in workspaces and invite users or teams
- Role-based permissions — **Owner, Editor, Commenter, Viewer** — keep team work safe

### Bring your own database

- Store connection profiles for databases you already run (encrypted) and test them in one click
- Managed and personal databases live together in a single list

### Auth & admin

- **GitHub & Google OAuth2 sign-in** — access tokens stay in memory, refresh tokens in a `SameSite=Strict` cookie, with per-request introspection at the gateway
- Admin console — manage users, code tables, managed DB instances, issue quota, and audit logs

## Architecture

Five services, one MSA.

```
  Browser
    │ HTTPS                          │ WebSocket (STOMP)
    ▼                                ▼
┌──────────────┐              ┌──────────────┐
│ crowfoot-web │              │crowfoot-collab│  presence · live sync
│  React SPA   │              └──────────────┘
└──────┬───────┘
       ▼
┌──────────────┐     ┌──────────────┐
│ api-gateway  │────▶│  auth        │  OAuth2 · JWT · introspection · Redis
└──────┬───────┘     └──────────────┘
       │             ┌──────────────┐
       └────────────▶│  core-api    │  domain (PostgreSQL) · managed DB
                     └──────┬───────┘  provisioning (PostgreSQL · MySQL)
```

## Repositories

| Repository | Description |
| --- | --- |
| [crowfoot-web](https://github.com/crowfoot-erd/crowfoot-web) | Frontend — React SPA. ERD editor (React Flow), dashboard·workspaces·teams, admin console, collaboration client (STOMP), i18n (ko·en) & dark mode |
| [crowfoot-api-gateway](https://github.com/crowfoot-erd/crowfoot-api-gateway) | API gateway — Spring Cloud Gateway. Routing, bearer-token introspection, user identity header injection, public-path whitelist |
| [crowfoot-auth](https://github.com/crowfoot-erd/crowfoot-auth) | Auth server — OAuth2 sign-in (GitHub·Google·PKCE), JWT issue/refresh/introspection, Redis blacklist (logout) |
| [crowfoot-core-api](https://github.com/crowfoot-erd/crowfoot-core-api) | Core API — users·workspaces·teams·model documents·comments, managed DB provisioning (dedicated account issue & revoke), SQL generation·deployment·reverse engineering, code tables·audit logs |
| [crowfoot-collab](https://github.com/crowfoot-erd/crowfoot-collab) | Collaboration server — WebSocket (STOMP). Per-document presence and real-time edit broadcast, single-instance deployment |

## Running It Yourself

### Prerequisites

| Tool | Version | Used by |
| --- | --- | --- |
| Java (Temurin) | 21 | the four servers |
| Maven | 3.9+ | building & running the servers |
| Node.js / pnpm | 20 / 10 | frontend |
| PostgreSQL | 16+ | domain DB (`crowfoot` database + `crowfoot_core` schema) |
| Redis | 6+ | auth logout blacklist |

The schema is not auto-created (`ddl-auto: none`) — initialize it with a DDL script.

### Local Ports

| Service | Port | Notes |
| --- | --- | --- |
| crowfoot-web (Vite) | 8080 | proxies `/api` to the gateway (8000) |
| crowfoot-api-gateway | 8000 | routes to `auth` (8081) and `core-api` (8082) |
| crowfoot-auth | 8081 | |
| crowfoot-core-api | 8082 | |
| crowfoot-collab | 8083 | WebSocket (STOMP) — connected directly, not through the gateway |

### Environment Variables

Each server automatically reads a `.env-local` file (gitignored) from the repository root — copy `.env-local.example` and fill in the values. The gateway and collab need no environment variables.

**crowfoot-auth**

| Variable | Description |
| --- | --- |
| `CROWFOOT_AUTH_JWT_SECRET` | JWT HS256 signing key — Base64, 32+ bytes (`openssl rand -base64 48`) |
| `CROWFOOT_AUTH_FLOW_SECRET` | HMAC-SHA256 key for the `auth_flow` cookie — kept separate from the JWT key |
| `CROWFOOT_AUTH_GITHUB_CLIENT_ID` / `..._SECRET` | GitHub OAuth app credentials |
| `CROWFOOT_AUTH_GOOGLE_CLIENT_ID` / `..._SECRET` | Google OAuth client credentials (PKCE) |
| `CROWFOOT_REDIS_PASSWORD` / `CROWFOOT_REDIS_DATABASE` | Redis blacklist connection (host lives in `application-local.yml`) |

Register the redirect URI `http://localhost:8080/auth/callback` in your OAuth apps.

**crowfoot-core-api**

| Variable | Description |
| --- | --- |
| `DB_URL` | PostgreSQL JDBC URL — `jdbc:postgresql://{host}:5432/crowfoot?currentSchema=crowfoot_core` |
| `DB_USERNAME` / `DB_PASSWORD` | domain database account |

**crowfoot-web** — development works with the defaults (`VITE_API_BASE_URL` empty → the Vite proxy keeps everything same-origin). Production builds inject `VITE_API_BASE_URL` (API gateway) and `VITE_WS_URL` (collaboration WS).

### Start

```bash
# the four servers — from each repository (local profile is the default)
mvn spring-boot:run

# frontend
pnpm install
pnpm dev        # http://localhost:8080
```

### Production

In production all five services run as container images with a unified server port of 8080. Secrets live only in environment variables — never in code or images. See each repository's `application-prod.yml` for the variables it expects.

## Tech Stack

**Frontend** — React 19 · TypeScript · Vite · TanStack Query · Zustand · React Flow · ELK (auto layout) · Tailwind CSS · shadcn/ui (radix-ui) · i18next · Vitest·Testing Library·Playwright·MSW

**Backend** — Java 21 · Spring Boot 4 · Spring Security (OAuth2 Client) · Spring Cloud Gateway · Spring Data JPA (Hibernate 7) · Querydsl · Spring WebSocket (STOMP)

**Databases** — PostgreSQL (domain · managed provisioning) · MySQL (managed provisioning) · Redis (auth sessions)

**Infrastructure** — GitHub Actions · GHCR · Kubernetes (Rancher) · ArgoCD (GitOps)

## License & Contributing

All repositories are distributed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0). Bug reports and contributions are always welcome — use the Issues on each repository.
