<p align="center">
  <img src="https://raw.githubusercontent.com/NovaChat-Enterprise-ERP/.github/main/profile/assets/logo.png" alt="NovaChat logo" width="88" />
</p>

<h1 align="center">NovaChat</h1>

<p align="center">
  <strong>Chat, share files and meet with your team in one place.</strong><br />
  An open-source team workspace built as TypeScript microservices.
</p>

<p align="center">
  <a href="https://novachat-web-beryl.vercel.app"><strong>Live demo</strong></a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#repositories">Repositories</a> ·
  <a href="#run-it-locally">Run it locally</a>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/NovaChat-Enterprise-ERP/.github/main/profile/assets/novachat-login.png" alt="NovaChat sign-in page" width="100%" />
</p>

> The demo runs on free hosting. If it has been idle, the first requests can take up to a minute while the services wake up.

## What you can do

- **Workspaces** with custom roles and permissions, member management, ownership transfer and an audit log
- **Invites** by email or shareable link, with expiry dates and usage limits
- **Channels and direct messages**: public and private channels, threads, reactions, pins and full-text search
- **Real-time updates**: new messages, reactions and presence arrive instantly
- **Files**: uploads straight to object storage, folders, sharing, a trash bin with restore, and avatars
- **Video meetings** with join links, participant management and host controls
- **Notifications** in the app and by email, with per-user preferences
- **Sign in with Google**, light and dark themes

## Architecture

```
                 ┌──────────────┐
  Browser ─────▶ │  web (Next)  │ ── /api/* ──┐
                 └──────────────┘             ▼
                                     ┌─────────────────┐
                                     │   api-gateway   │  sessions · rate limits · routing · WebSockets
                                     └────────┬────────┘
        ┌──────────┬───────────┬──────────────┼───────────┬────────────┬──────────────┐
        ▼          ▼           ▼              ▼           ▼            ▼              ▼
      auth     workspace    message       realtime       file         call      notification
        │          │           │              │           │            │              │
        └──────────┴───────────┴── Redis Streams (events) ┴────────────┴──────────────┘
                 Postgres (one schema per service) · S3-compatible storage · LiveKit
```

- The **gateway** is the only public entry point. It checks the session, then forwards requests with an HMAC-signed identity header. Services reject any request without a valid signature.
- Services own their data (separate Postgres schemas and migrations) and talk to each other through **signed internal HTTP calls** and **events on Redis Streams**.
- Each service is layered as routes → controllers → services → repositories, with business logic only in the services.

## Repositories

| Repository | What it does |
|---|---|
| [web](https://github.com/NovaChat-Enterprise-ERP/web) | Next.js 16 client: React 19, Tailwind CSS 4, TanStack Query, LiveKit components |
| [api-gateway](https://github.com/NovaChat-Enterprise-ERP/api-gateway) | Entry point: authentication, rate limiting, request and WebSocket proxying. Also holds the Render Blueprint (`render.yaml`) |
| [auth-service](https://github.com/NovaChat-Enterprise-ERP/auth-service) | Users and sessions with Better Auth (Google OAuth) |
| [workspace-service](https://github.com/NovaChat-Enterprise-ERP/workspace-service) | Workspaces, members, roles and permissions, invites, audit log |
| [message-service](https://github.com/NovaChat-Enterprise-ERP/message-service) | Channels, direct messages, threads, reactions, pins, search |
| [realtime-service](https://github.com/NovaChat-Enterprise-ERP/realtime-service) | Socket.IO server with a Redis adapter for live updates |
| [file-service](https://github.com/NovaChat-Enterprise-ERP/file-service) | File uploads via presigned S3 URLs, folders, sharing, trash |
| [call-service](https://github.com/NovaChat-Enterprise-ERP/call-service) | Video meetings on LiveKit |
| [notification-service](https://github.com/NovaChat-Enterprise-ERP/notification-service) | In-app notifications and email via Resend |
| [packages](https://github.com/NovaChat-Enterprise-ERP/packages) | Shared libraries published as `@novachat-enterprise-erp/*`: common, contracts, events, cache, testing, tsconfig |

## Tech stack

**Backend:** Node.js 24, TypeScript, Express 5, Drizzle ORM, PostgreSQL (Neon), Redis (Upstash), Better Auth, Socket.IO, LiveKit, Zod<br />
**Frontend:** Next.js 16, React 19, Tailwind CSS 4, Radix UI, TanStack Query<br />
**Infrastructure:** Docker, GitHub Actions, GitHub Packages, Render (backend), Vercel (frontend)

## Run it locally

Every service is its own repository. To work on one:

1. Create a GitHub [personal access token](https://github.com/settings/tokens) with the `read:packages` scope and add it to `~/.npmrc`:
   ```
   //npm.pkg.github.com/:_authToken=<your-token>
   ```
2. Clone the repository, copy `.env.example` to `.env` and fill in the values.
3. Install and start:
   ```sh
   pnpm install
   pnpm db:migrate   # services with a database
   pnpm dev
   ```

The gateway expects the services on ports 4001–4007 and listens on 4000. The web app runs on 3000 and proxies `/api` to the gateway.
