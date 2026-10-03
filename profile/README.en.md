<div align="center">

<img src="logo.svg" alt="Crowfoot" width="72" />

# Crowfoot

**Plenty of tools stop at the ERD. Crowfoot goes all the way to a real database.**

An open-source ERD platform that takes you from requirements → ERD → a real database → data, all in one browser

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Release](https://img.shields.io/badge/release-v1.32-10b981.svg)](https://crowfoot.java21.net/release-notes/34)
[![Live](https://img.shields.io/badge/live-crowfoot.java21.net-0ea5e9.svg)](https://crowfoot.java21.net)
[![MCP](https://img.shields.io/badge/MCP-Claude%20%C2%B7%20ChatGPT-f97316.svg)](https://crowfoot.java21.net/guide#20.1)

**[한국어](./README.md)** | **English** | **[日本語](./README.ja.md)** | **[简体中文](./README.zh.md)**

[Try it now](https://crowfoot.java21.net) · [User guide](https://crowfoot.java21.net/guide) · [Release notes](https://crowfoot.java21.net/release-notes) · [Browse shared ERDs](https://crowfoot.java21.net/shared)

</div>

<p align="center">
  <img src="images/en/landing-hero.webp" alt="Crowfoot start page — an ERD tool that takes you all the way to a real database, free" width="860" />
</p>

The name comes from **Crow's Foot notation**, which draws the relationships between tables with a symbol shaped like a crow's foot.

## Contents

- [Why Crowfoot](#why-crowfoot)
- [Quick start](#quick-start)
- [Features](#features)
- [Architecture](#architecture)
- [Repositories](#repositories)
- [Run it yourself](#run-it-yourself)
- [Tech stack](#tech-stack)
- [Releases](#releases)
- [Contributing](#contributing)
- [License](#license)

## Why Crowfoot

| | Typical ERD tools | Crowfoot |
| --- | --- | --- |
| Output | Diagrams, DDL files | Diagrams and DDL, plus **a database that actually runs** |
| Database | Bring your own | **Free** MySQL and PostgreSQL development databases (5 per user in each workspace) |
| Applying changes | Run DDL by hand | Computes the difference between the document and the database, then **generates and runs migration SQL**. Drop statements need separate approval |
| AI | None, or an AI tied to the tool | **Connect the Claude or ChatGPT you already use over MCP**. Crowfoot has no AI built in |
| Requirements | A separate document | Requirements are saved with the ERD, and you can **trace which tables implement them** |
| Data | Check it in another tool | Browse and edit data, run SQL and add sample data in the **data browser** |
| Collaboration | Share files | **Real-time co-editing**, comments, version history, share links |

## Quick start

### Use the hosted service

1. Sign in at https://crowfoot.java21.net with your GitHub or Google account.
2. Create a workspace and open an ERD document. Start from a blank document, import an SQL script, or read an existing database into an ERD.
3. On the **Database** tab, issue a free MySQL or PostgreSQL database and deploy your document to it.

### Connect Claude or ChatGPT (MCP)

Issue a token on your workspace's **MCP** tab, and you get a registration command with the token already filled in.

```bash
claude mcp add --transport http crowfoot https://crowfoot-mcp.java21.net/mcp \
  --header "Authorization: Bearer <your token>"
```

Then hand off the work in conversation.

```text
> Collect the requirements for a book rental service and turn them into an ERD
> Issue a free MySQL database and deploy it
> Put 10 rows of sample data into each table
```

What the AI creates shows up in Crowfoot as is, and the AI reads back whatever you change on screen. See [section 20 of the user guide](https://crowfoot.java21.net/guide#20.1) for details.

## Features

<table>
<tr>
<td width="50%"><img src="images/en/editor-overview.webp" alt="ERD editor" /><br/><b>ERD editor</b> — Crow's Foot notation, auto layout, groups, notes</td>
<td width="50%"><img src="images/en/editor-requirements.webp" alt="Requirements tab" /><br/><b>Requirement tracing</b> — progress by domain, tables without a requirement</td>
</tr>
<tr>
<td width="50%"><img src="images/en/mcp-usage.webp" alt="How to use MCP" /><br/><b>AI integration (MCP)</b> — connection command and example requests</td>
<td width="50%"><img src="images/en/data-tab.webp" alt="Data browser" /><br/><b>Data browser</b> — browsing, row editing, SQL console</td>
</tr>
</table>

### Design

- **ERD editor in your browser** — Use it right away, nothing to install. Draw tables, columns, keys, indexes and relationships in Crow's Foot notation, and tidy up even large documents with auto layout (layered, hub-centered, hybrid).
- **Logical and physical models** — Manage logical and physical names together, and convert common types into DBMS-specific types. Supported target DBMSs are MySQL, PostgreSQL, Oracle and SQL Server.
- **Standard dictionary** — Keep names and types consistent with word and term dictionaries and domain types. Crowfoot suggests physical names from logical names, and changing a domain type carries over to the columns that use it.
- **Design validation** — 17 rules, such as duplicate names, FK type mismatches and missing primary keys, are checked continuously.
- **Requirements** — Save requirements in the document and link them to tables. When a requirement changes, it is marked "Pending", and you get progress by domain, acceptance criteria and Markdown/CSV export.

### Databases

- **Free managed databases** — Issue a MySQL or PostgreSQL development database with one click. Each issue gets a dedicated account with privileges on its own schema only, and the instance admin account is never handed out anywhere.
- **SQL generation and deployment** — Turn a document into DDL for its DBMS and deploy it straight to a connected database.
- **Reverse engineering** — Read an existing database or import an SQL script to create an ERD document.
- **Migration** — Recompute the difference between the document and the database, then generate and run the change SQL. Statements that drop tables or columns are not run by default.
- **Data browser** — Browse, filter and sort the data in a connected database, edit rows, and run SQL.

### Collaboration and sharing

- **Real-time collaboration** — Several people edit the same document at once. You see who is online, their cursors and their selections in real time.
- **Workspaces and teams** — Invite users and teams as Owner, Editor, Commenter or Viewer.
- **Version history** — Every save keeps a version. Compare two versions or roll back.
- **Sharing** — Share read-only with a link and collect likes and comments. Anyone can find shared documents in the [shared ERD list](https://crowfoot.java21.net/shared).
- **ERD library** — More than 500 example ERDs on real-world topics, published with notes on requirements and design.

### More

- **Four languages** — Korean, English, Japanese and Chinese screens and user guide
- **AI integration (MCP)** — 20 tools: reading and creating documents, applying requirements and schemas, issuing, deploying and migrating databases, and sample data. Issuing, deploying and applying show a plan first and run only what you approve.
- **Admin console** — Users, code tables, managed DB instances, issue quotas, audit logs and traffic analytics

## Architecture

Crowfoot is made of seven microservices. Every HTTP request from outside goes through the API gateway, and services talk to each other only through internal calls inside the cluster.

![Crowfoot architecture — browser and MCP clients, nginx, seven services in Kubernetes, data stores, and the delivery pipeline](architecture.svg)

### Services

| Service | What it does | Storage | Depends on |
| --- | --- | --- | --- |
| **crowfoot-web** | React SPA. Editor, dashboard, admin console, public pages | — | gateway, collab |
| **crowfoot-api-gateway** | The entry point for all HTTP requests. Routing by path and host, token verification, public-path allowlist, user identity header (`X-USER-ID` and others) injection | — | auth |
| **crowfoot-auth** | GitHub and Google OAuth2 sign-in (PKCE), JWT issue and refresh, token verification (introspection), logout blacklist | Redis | core (members, workspace tokens) |
| **crowfoot-core-api** | The center of the domain. Members, workspaces, teams, documents, requirements and comments, SQL generation, deployment, reverse engineering and migration, managed DB issuing, audit logs | PostgreSQL | managed DB instances, user databases |
| **crowfoot-collab** | Real-time collaboration. Relays presence and edits in WebSocket (STOMP) rooms | Memory | auth, core |
| **crowfoot-database-manager** | Data browser. Browsing, row editing, SQL console and sample data. Has no database of its own and connects per request | — | core (permissions, connection info) |
| **crowfoot-mcp** | MCP server. Turns tool calls from Claude and ChatGPT into calls to core and the DB manager | — | core, database-manager |

### Request flows

**Sign-in and API calls** — The access token lives only in browser memory, and the refresh token in a `SameSite=Strict` cookie. The gateway verifies the token with the auth server on every request.

```mermaid
sequenceDiagram
    participant B as Browser
    participant G as API gateway
    participant A as Auth server
    participant C as Core API
    B->>G: GET /api/v1/core/... (Bearer access token)
    G->>A: Verify token (introspection)
    A-->>G: User id, validity
    G->>C: Request + X-USER-ID
    C-->>G: Response (core decides permissions)
    G-->>B: Response
```

**AI integration (MCP)** — A workspace token (`cfw_…`) carries the permissions of the person who issued it and works only inside that workspace. The token can pass only the MCP path, and the regular API is blocked.

```mermaid
sequenceDiagram
    participant M as Claude · ChatGPT
    participant G as API gateway
    participant A as Auth server
    participant P as MCP server
    participant C as Core API
    participant D as DB manager
    M->>G: POST /mcp (Bearer cfw_…)
    G->>A: Verify workspace token
    A->>C: Look up token (internal API)
    G->>P: Tool call + X-USER-ID, X-TOKEN-WORKSPACE-ID
    P->>C: Read and edit documents, plan and run deployments
    P->>D: Add sample data
    P-->>M: Result and document address
```

**Data browser** — The DB manager does not store connection info. On every request it gets permissions and connection info from the core API, then connects to the target database.

```mermaid
sequenceDiagram
    participant B as Browser
    participant G as API gateway
    participant D as DB manager
    participant C as Core API
    participant T as Target DB
    B->>G: List tables, browse rows, run SQL
    G->>D: Request + X-USER-ID
    D->>C: Check connection access (internal API)
    C-->>D: Role, address, decrypted credentials
    D->>T: Connect and run over JDBC
    D-->>B: Result
```

**Real-time collaboration** — The browser connects to the collaboration server directly over WebSocket. The collaboration server checks the token and document permissions when you connect, and relays the changes in a room in order. Documents are saved over HTTP to the core API, and concurrent saves are prevented by version comparison.

### Security and data protection

- **Password encryption** — Connection passwords and issued account passwords are stored encrypted with AES-256-GCM.
- **Permissions are decided in core** — Other services do not decide permissions on their own; they ask the core API. Other people's resources return 404, hiding that they exist at all.
- **Isolated issued accounts** — Each managed DB issue gets an account with privileges on its own schema only, and revoking it deletes both the schema and the account.
- **Internal addresses** — In production, servers connect to managed databases through internal addresses inside the cluster. Users are shown addresses they can reach from outside.
- **Audit logs** — Important actions such as issuing, revoking, deploying, migrating and viewing connection info are recorded.

### Deployment

Deployment is GitOps. When you push to `main` in a service repository, GitHub Actions runs the tests, builds an image and pushes it to GHCR, then updates the image tag in the deployment repository. Argo CD applies that change to the Kubernetes cluster. All production servers run on port 8080, and secrets are injected only through environment variables.

## Repositories

| Repository | Description |
| --- | --- |
| [crowfoot-web](https://github.com/crowfoot-erd/crowfoot-web) | Frontend — React SPA. ERD editor (React Flow), dashboard, workspaces and teams, admin console, collaboration client (STOMP), four languages and dark mode, user guide |
| [crowfoot-api-gateway](https://github.com/crowfoot-erd/crowfoot-api-gateway) | API gateway — Spring Cloud Gateway. Routing, token verification, user identity headers, public-path allowlist |
| [crowfoot-auth](https://github.com/crowfoot-erd/crowfoot-auth) | Auth server — OAuth2 sign-in (GitHub, Google, PKCE), JWT issue, refresh and verification, workspace token verification, Redis blacklist |
| [crowfoot-core-api](https://github.com/crowfoot-erd/crowfoot-core-api) | Core API — members, workspaces, teams, documents, requirements and comments, managed DB issuing and revoking, SQL generation, deployment, reverse engineering and migration, document editing API, code tables and audit logs |
| [crowfoot-collab](https://github.com/crowfoot-erd/crowfoot-collab) | Collaboration server — WebSocket (STOMP). Real-time relay of per-document presence and edits |
| [crowfoot-database-manager](https://github.com/crowfoot-erd/crowfoot-database-manager) | DB manager — data browsing, row editing, SQL console and sample data. Has no database of its own, connects per request, and asks the core API for permissions |
| [crowfoot-mcp](https://github.com/crowfoot-erd/crowfoot-mcp) | MCP server — Spring AI MCP. Provides tools for reading and writing requirements and ERDs, issuing, deploying and migrating databases, and sample data |

## Run it yourself

### Prerequisites

| Tool | Version | Used by |
| --- | --- | --- |
| Java (Temurin) | 21 | the six servers |
| Maven | 3.9+ | building and running the servers |
| Node.js / pnpm | 20.19+ / 10 | frontend |
| PostgreSQL | 16+ | service database (`crowfoot` database, `crowfoot_core` schema) |
| Redis | 6+ | logout blacklist |

The schema is not created automatically (`ddl-auto: none`). Initialize it with a DDL script first.

### Local ports

| Service | Port | Notes |
| --- | --- | --- |
| crowfoot-web (Vite) | 8080 | Forwards `/api` requests to the gateway (8000) |
| crowfoot-api-gateway | 8000 | Routes to auth, core and database-manager, and `/mcp` on the MCP host to the MCP server |
| crowfoot-auth | 8081 | |
| crowfoot-core-api | 8082 | |
| crowfoot-collab | 8083 | WebSocket — connected directly, not through the gateway |
| crowfoot-database-manager | 8084 | |
| crowfoot-mcp | 8085 | |

### Environment variables

Each server reads `.env-local` (not committed to git) from the repository root. Copy `.env-local.example` and fill in the values. The gateway, collab, DB manager and MCP server need no extra values.

**crowfoot-auth**

| Variable | Description |
| --- | --- |
| `CROWFOOT_AUTH_JWT_SECRET` | JWT HS256 signing key — Base64, 32+ bytes (`openssl rand -base64 48`) |
| `CROWFOOT_AUTH_FLOW_SECRET` | HMAC-SHA256 signing key for the sign-in flow cookie (`auth_flow`) — keep it separate from the JWT key |
| `CROWFOOT_AUTH_GITHUB_CLIENT_ID` / `..._SECRET` | GitHub OAuth app credentials |
| `CROWFOOT_AUTH_GOOGLE_CLIENT_ID` / `..._SECRET` | Google OAuth client credentials (PKCE) |
| `CROWFOOT_REDIS_PASSWORD` / `CROWFOOT_REDIS_DATABASE` | Redis connection (the host is in `application-local.yml`) |

Register `http://localhost:8080/auth/callback` as the redirect URI in your OAuth apps.

**crowfoot-core-api**

| Variable | Description |
| --- | --- |
| `DB_URL` | PostgreSQL JDBC URL — `jdbc:postgresql://{host}:5432/crowfoot?currentSchema=crowfoot_core` |
| `DB_USERNAME` / `DB_PASSWORD` | Service database account |
| `CROWFOOT_CONNECTION_SECRET_KEY` | Encryption key for connection passwords — Base64, 32 bytes. Local has a development default; production must set it |

**crowfoot-web** — Development uses the defaults as they are (leave `VITE_API_BASE_URL` empty so the Vite proxy keeps everything same-origin). Production builds set `VITE_API_BASE_URL` (gateway address), `VITE_WS_URL` (collaboration server address, `wss://`) and `VITE_MCP_URL` (MCP address).

### Start

```bash
# the six servers — from each repository (local profile is the default)
mvn spring-boot:run

# frontend
pnpm install
pnpm dev        # http://localhost:8080
```

The recommended order is auth → core-api → gateway → collab · database-manager · mcp → web. The gateway can verify tokens only once auth is up.

## Tech stack

| Area | Technologies |
| --- | --- |
| Frontend | React 19 · TypeScript · Vite · TanStack Query · Zustand · React Flow · elkjs (auto layout) · Tailwind CSS · shadcn/ui · i18next · CodeMirror |
| Backend | Java 21 · Spring Boot 4 · Spring Security (OAuth2 Client) · Spring Cloud Gateway · Spring Data JPA (Hibernate 7) · Querydsl · Spring WebSocket (STOMP) · Spring AI (MCP) |
| Data | PostgreSQL (service DB, managed issuing) · MySQL (managed issuing) · Redis (logout blacklist) |
| Testing | JUnit 5 · Testcontainers · Vitest · Testing Library · MSW · Playwright |
| Infrastructure | GitHub Actions · GHCR · Kubernetes · Argo CD (GitOps) · nginx |

## Releases

Every version ships with [release notes](https://crowfoot.java21.net/release-notes) in four languages. Each version is tagged with the same git tag (`vX.Y`) in all seven service repositories — repositories with no changes also get the tag to keep the system version aligned.

| Version | Date | Highlights | Release notes |
| --- | --- | --- | --- |
| v1.32 | 2026-10-03 | AI integration extended (sample data, document links, drop statements skipped by default), requirements organized by domain (progress, search, export, acceptance criteria), shared documents list and release notes with a table of contents, new start page, new-version notice | [View](https://crowfoot.java21.net/release-notes/34) |
| v1.31 | 2026-10-02 | Claude integration (MCP — connect Claude Code with a workspace token, build requirements and ERDs through conversation), requirements panel (linked tables, pending status), automatic placement on open, per-connection Allow MCP apply | [View](https://crowfoot.java21.net/release-notes/33) |
| v1.30 | 2026-10-02 | Terms linked to domain types, column name suggestions split into terms and words, one standards panel, user guide (screenshots in four languages, search) | [View](https://crowfoot.java21.net/release-notes/32) |
| v1.29 | 2026-10-02 | Domain types (shared type definitions, previewed propagation of changes), paste into another document, editing a relationship's column mapping, auto layout direction | [View](https://crowfoot.java21.net/release-notes/31) |
| v1.28 | 2026-10-01 | Data browser (browse, filter, sort, CSV), row editing (collected and applied at once, conflict detection), SQL console (syntax highlighting, autocomplete) | [View](https://crowfoot.java21.net/release-notes/30) |

<details>
<summary>Earlier versions (v1.08 – v1.27)</summary>

| Version | Date | Highlights | Release notes |
| --- | --- | --- | --- |
| v1.27 | 2026-10-01 | Duplicate for another DBMS, tidier editor toolbar, document list actions menu, refreshed landing and sign-in pages | [View](https://crowfoot.java21.net/release-notes/29) |
| v1.26 | 2026-09-30 | Link documents to databases, apply migration DDL to the database (recompute at run time, per-statement report) | [View](https://crowfoot.java21.net/release-notes/28) |
| v1.25 | 2026-09-29 | ERD library launch (509 real-world topics), balanced layout spacing | [View](https://crowfoot.java21.net/release-notes/27) |
| v1.24 | 2026-09-29 | Three auto-layout modes, relationship lines rerouting live during drag, automatic unique key on 1:1 relationships, document list pagination | [View](https://crowfoot.java21.net/release-notes/26) |
| v1.23 | 2026-09-28 | Release note image improvements, deployment procedure documented — no feature changes | [View](https://crowfoot.java21.net/release-notes/25) |
| v1.22 | 2026-09-28 | Notifications (header bell, three events, a full notifications page) | [View](https://crowfoot.java21.net/release-notes/24) |
| v1.21 | 2026-09-28 | Document-level feedback (likes, anonymous comments, author replies), SQL export from the public viewer | [View](https://crowfoot.java21.net/release-notes/23) |
| v1.20 | 2026-09-27 | Design validation (17 rules), automatic FK indexes | [View](https://crowfoot.java21.net/release-notes/22) |
| v1.19 | 2026-09-27 | Admin traffic analytics | [View](https://crowfoot.java21.net/release-notes/21) |
| v1.18 | 2026-09-27 | Template showcase, unified share gallery, share SEO, empty-canvas onboarding | [View](https://crowfoot.java21.net/release-notes/20) |
| v1.17 | 2026-09-26 | Collaboration upgrade (cursors, selections and movement, concurrent editing convergence, edit locks) | [View](https://crowfoot.java21.net/release-notes/19) |
| v1.16 | 2026-09-25 | Full support for 4 languages, per-language URLs & SEO | [View](https://crowfoot.java21.net/release-notes/18) |
| v1.15 | 2026-09-25 | System dictionary expansion (34,075 standard tokens) | [View](https://crowfoot.java21.net/release-notes/17) |
| v1.14 | 2026-09-25 | Term dictionary panel, dictionary suggestions for column physical names | [View](https://crowfoot.java21.net/release-notes/16) |
| v1.13 | 2026-09-24 | Logical groups (subject areas), logical name inference, keyboard cheat sheet | [View](https://crowfoot.java21.net/release-notes/15) |
| v1.12 | 2026-09-23 | Model explorer & unified search, SQL import | [View](https://crowfoot.java21.net/release-notes/14) |
| v1.11 | 2026-09-22 | Relationship editing & version compare, faster image export | [View](https://crowfoot.java21.net/release-notes/13) |
| v1.10 | 2026-09-21 | Version compare & migration DDL | [View](https://crowfoot.java21.net/release-notes/11) |
| v1.09 | 2026-09-20 | Document version history & DB sync | [View](https://crowfoot.java21.net/release-notes/10) |
| v1.08 | 2026-09-18 | Community boards, real-time chat, editor safeguards | [View](https://crowfoot.java21.net/release-notes/9) |

</details>

## Contributing

Bug reports, feature ideas and pull requests are all welcome.

- **Bugs and ideas** — Open an issue in the relevant repository. If you're not sure which one, use [crowfoot-web](https://github.com/crowfoot-erd/crowfoot-web/issues). Steps to reproduce, expected behavior, actual behavior and screenshots help us fix things faster.
- **Pull requests** — Keep changes small and include tests. Servers must pass `mvn test`, and the frontend must pass `pnpm vitest run` and `pnpm build`.
- **Changes to the UI** — Update the text and images in the user guide (four languages) as well.
- **Security issues** — Please tell the repository maintainers first instead of opening a public issue.

## License

All repositories are distributed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
