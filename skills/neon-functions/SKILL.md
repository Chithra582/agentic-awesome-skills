---
name: neon-functions
description: Long-running, serverless Node.js HTTP functions deployed onto your Neon
  branch, with DATABASE_URL injected automatically and compute that runs next to your
  data.
license: Apache-2.0
metadata:
  risk: critical
  source: https://github.com/neondatabase/agent-skills/tree/main/skills/neon-functions
  source_repo: neondatabase/agent-skills
  source_type: official
  date_added: '2026-07-01'
  license_source: https://github.com/neondatabase/agent-skills/blob/main/LICENSE
---
# Neon Functions

This is a preview feature and only available in `us-east-2`. Neon Functions are long-running Node.js HTTP handlers deployed onto a Neon branch. Each function gets a public HTTPS URL, runs in the same region as your database, and — if the branch has Postgres — gets `DATABASE_URL` injected automatically. You deploy and manage them through the same Neon CLI, `neon.ts`, and API you already use.

Use this skill to help the user define, run locally, deploy, and manage functions next to their database. Deliver a deployed function with its invocation URL, a working local `neon dev` loop, or a precise answer from the official Neon docs.

## When to Use

Reach for Neon Functions when the workload is a request/response handler that benefits from staying alive and staying close to the data:

- **Long-running request/response flows that outlast lambda-style limits.** Agents that make several LLM calls and tool invocations per request, or image/video generation, routinely blow past the ~10–60s execution caps and short streaming windows of traditional serverless functions. Neon Functions are long-running: the handler just needs to _start_ responding within 15 minutes, and an open stream stays alive as long as bytes keep flowing. That's enough headroom for real agent workloads.
- **Stateful streaming without bolting on Redis.** Because a function stays alive across a request, it can host an SSE endpoint or a WebSocket server and hold the connection open in-process — no external state store (Redis, etc.) needed just to keep a stream coherent. Module-scope state (a `pg` pool, an in-memory counter) persists across requests on the same isolate.
- **Compute that must sit next to Postgres.** The function runs in the same region as the branch's database, so there are no cross-region round trips on every query. `DATABASE_URL` is injected for you.
- **A backend that branches with your data.** Each branch runs its own version of the function at its own URL, against its own isolated database (and storage, and gateway) state. Preview deployments, CI, and dev environments each get a self-contained backend — deploying to a child never affects the parent.
- **Webhooks, bots, and post-response work.** Webhook handlers that fan out into multiple DB writes, Discord/WebSocket bots, and fire-and-forget follow-ups via `waitUntil` (analytics, audit logs) all fit.

If the workload is a pure static site, a cron/background job that needs its own lifecycle and cancellation, or something that must run outside `us-east-2` today, this isn't the right tool yet (see Timeouts and Availability below).

## What It Does

- **Long-running & serverless** — Built for WebSocket servers (see [WebSocket servers](#websocket-servers)), SSE endpoints (see [Server-sent events (SSE)](#server-sent-events-sse)), long agent HTTP streams, and APIs. Still scales to zero when idle.
- **Web-standard handler** — A function is any default export with a `fetch(request)` method returning a `Response` (Workers/WinterTC-compatible). A Hono app exports exactly that shape, so `export default app` just works. Runs on Node.js 24, so all Node APIs are available.
- **Close to your database** — Runs in the branch's region; `DATABASE_URL` injected automatically when the branch has Postgres.
- **Branchable** — Each branch runs its own function version at its own URL against its own isolated state.
- **Same CLI/API** — Deploy and manage via `neon`, `neon.ts`, or the Neon API.

## Architecture: where Functions fit

Neon (Functions included) is **backend primitives, not full-stack app hosting**. Host your app on **Vercel** (or Netlify, or another frontend/app host); Functions are the long-running, stateful slice of your backend that lives next to your data. They compose with that platform in two ways:

- **Add a Function to a full-stack app.** Your Next.js / TanStack Start app on Vercel (or Netlify) owns UI, auth (e.g. Neon Auth), and talks directly to Neon Postgres and Object Storage. When one workload outgrows the host's short serverless limits — a WebSocket or SSE server, or a long-running agent that would time out — move just that piece onto a Neon Function. (See [Functions as an agent backend](#functions-as-an-agent-backend-nextjs-and-similar-frameworks) for the client-direct pattern.)
- **Run the whole backend control plane on Functions.** Especially when the frontend is **client-only** — TanStack Router, React Router in client mode, and similar SPAs hosted on Vercel or Netlify — the client calls Functions **directly**. Build REST APIs and request/response agents, host **MCP servers**, and run anything stateful or that belongs close to Postgres and Object Storage.

Either way, secure a Function like any standalone REST API: verify a JWT or API key at the top of the handler (see the WARNING under [Functions as an agent backend](#functions-as-an-agent-backend-nextjs-and-similar-frameworks)). Because a Function is just your backend, you can **move pieces between your host and Neon** — relocate an agent or a stateful WebSocket server onto a Function when it needs more runtime, and back if needed.

## Setup

Functions are declared in `neon.ts` (see the `neon` skill for the branch-first workflow and `neon.ts` basics). Add `@neon/config` and declare functions under `preview.functions`, keyed by **slug**:

```typescript
// neon.ts
import { defineConfig } from "@neon/config/v1";

export default defineConfig({
  preview: {
    functions: {
      todos: {
        // slug: ^[a-z0-9]{1,20}$ — lowercase letters/digits, no hyphens
        name: "todo api", // display label only
        source: "src/index.ts", // entry file, relative to neon.ts
      },
    },
  },
});
```

The slug is the function's permanent identity (it appears in the invocation URL and CLI commands) and can't be changed after the first deploy. Use `name` for a human-readable label.

A minimal function — a Hono app that queries the branch's Postgres via the injected `DATABASE_URL`:

```typescript
// src/index.ts
import { Hono } from "hono";
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";
import { parseEnv } from "@neon/env";
import config from "../neon";
import { todos } from "./db/schema";

const env = parseEnv(config);
const pool = new Pool({ connectionString: env.postgres.databaseUrl, max: 5 });
const db = drizzle(pool);

const app = new Hono();
app.get("/", (c) => c.text("Neon + Hono + Drizzle"));
app.post("/todos", async (c) => {
  const { text } = await c.req.json<{ text: string }>();
  const [row] = await db.insert(todos).values({ text }).returning();
  return c.json(row, 201);
});
app.get("/todos", async (c) => c.json(await db.select().from(todos)));

export default app;
```

Create the `pg` pool at module scope (reused across requests on the same isolate) and keep `max` small (e.g. 5), since each isolate keeps its own pool.

`parseEnv(config)` requires _every_ variable the config implies. A function that only talks to Postgres over the pooled URL can scope it to just that key — `parseEnv` then validates and returns only what you asked for (the keys autocomplete from your `neon.ts`):

```typescript
const { postgres } = parseEnv(config, ["DATABASE_URL"]); // not the unpooled URL, auth, etc.
const pool = new Pool({ connectionString: postgres.databaseUrl, max: 5 });
```

## Develop locally and deploy

```bash
neon dev      # serves every function in neon.ts with hot reload; injects DATABASE_URL & friends
neon deploy   # bundles with esbuild, uploads, and applies neon.ts to the linked branch
```

To deploy a single function without `neon.ts`: `neon functions deploy <slug> --path . --entry src/index.ts`. Retrieve the public URL with `neon functions get <slug>` (the `invocation_url` field, of the form `https://<branch_id>-<slug>.compute.c-1.us-east-2.aws.neon.tech`). Manage with `neon functions list|get|delete`.

When `neon checkout` _creates_ a new branch and a `neon.ts` is present, it applies the policy automatically — deploying the function to the fresh branch. Checking out an existing branch does not re-deploy; run `neon deploy` explicitly.

## Neon Infrastructure as Code (`neon.ts`)

The `preview.functions` block from [Setup](#setup) is part of `neon.ts`, Neon's infrastructure-as-code file — one TypeScript file declares every function (its `source`, display `name`, and `env`) alongside any other branch services, in version control (see the `neon` skill for the full reference). Treat it like Terraform for your branch:

```bash
neon config status   # print the branch's live config (deployed functions)
neon config plan     # dry-run diff of what apply would change
neon config apply    # bundle + deploy the declared functions  (neon deploy is an alias)
```

Functions are **branch-scoped**: each branch runs its own deployment at its own URL. When a `neon.ts` is present, `neon checkout` applies the policy as it _creates_ a branch, so a fresh preview/CI branch comes up with the function already deployed. Checking out an _existing_ branch doesn't redeploy — run `neon deploy` to apply changes.

Per-branch deploy tuning (e.g. `runtime`) lives in the `branch` closure, keyed by slug, so it can vary by branch without changing which functions exist:

```typescript
export default defineConfig({
  preview: {
    functions: { todos: { name: "todo api", source: "src/index.ts" } },
  },
  branch: (branch) => ({
    preview: { functions: { todos: { runtime: "nodejs24" } } },
  }),
});
```

## Environment variables

Neon injects branch-scoped connection strings and service URLs at runtime — you don't declare these or pass them at deploy time:

| Variable                | Notes                                                                                    |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| `NEON_BRANCH`           | The branch **name** (e.g. `main`, `preview/foo`). Injected on every branch, including the default. |
| `DATABASE_URL`          | Pooled connection string. Use for most queries. Present only if the branch has Postgres. |
| `DATABASE_URL_UNPOOLED` | Direct connection. Use for migrations, `LISTEN`/`NOTIFY`, multi-round-trip transactions. |
| `NEON_AUTH_BASE_URL`    | Present when Neon Auth is enabled on the branch.                                         |
| `NEON_DATA_API_URL`     | Present when the Data API is enabled on the branch.                                      |

Object storage (`AWS_*`) and AI Gateway (`OPENAI_*`, `NEON_AI_GATEWAY_*`) vars are also injected when those services are declared — see the `neon-object-storage` and `neon-ai-gateway` skills.

`neon env pull` / `neon-env run` / `neon dev` emit `NEON_BRANCH` (and the connection strings) into your local dev environment too, so local runs mirror the deployed runtime.

**Your own secrets** are per-deployment. Set them with `--env KEY=VALUE` on `neon functions deploy` (repeatable; `--env KEY=` deletes a key, unmentioned keys carry over), or declare them in `neon.ts` under the function's `env` (resolved at deploy time, so read from `process.env` to avoid hardcoding):

```typescript
functions: {
  todos: {
    name: "todo api",
    source: "src/index.ts",
    env: { OPENAI_API_KEY: process.env.OPENAI_API_KEY! },
  },
}
```

Load a `.env` before deploy with `neon deploy --env .env.production`. Pull the branch's Neon-managed vars onto disk for local dev with `neon env pull` (`link`/`checkout` do this automatically; pass `--no-env-pull` to skip and use `neon-env run -- <cmd>` for runtime injection). Limits: ≤1,000 vars, ≤64 KiB total, and the `NEON_` prefix is reserved.

## Connecting to Postgres

When the branch has Postgres, Neon **injects the connection strings at runtime** — you don't declare them, pass them at deploy time, or hardcode anything. The two you'll use:

- `DATABASE_URL` — **pooled** connection string (routed through Neon's connection pooler). Use it for normal request/response query traffic. Kept un-prefixed because every Postgres ORM (Drizzle, Prisma, Knex, …) reads `DATABASE_URL` by default.
- `DATABASE_URL_UNPOOLED` — **direct** connection string to the same database. Use it for migrations, `LISTEN`/`NOTIFY`, and long multi-statement transactions.

**Use Drizzle (or another ORM) on top of node-postgres (`pg`)** for queries and schema management — not Neon's serverless driver. Functions are long-running and reuse an isolate across many requests, so a persistent `pg` pool is the right fit; the serverless driver's HTTP transport is meant for fully isolated, lambda-style runtimes.

Create the connection pool **once at module scope** and reuse it across requests — don't open a connection per request:

```typescript
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";

// Created once per isolate; reused by every request that isolate handles.
const pool = new Pool({ connectionString: process.env.DATABASE_URL, max: 5 });
const db = drizzle(pool);
```

**Pooling is recommended because an isolate is reused across many requests** (and several requests can be in flight on the same isolate at once — see [Timeouts and runtime limits](#timeouts-and-runtime-limits)). A module-scope pool is opened once on cold start and then shared by every subsequent request that isolate serves, so you amortize connection setup instead of paying it on every request and you avoid exhausting Postgres connections under load.

Keep `max` small (e.g. `5`): each isolate keeps its own pool, so total connections to Postgres scale with the number of live isolates. You don't need to close the pool on shutdown — when the runtime evicts an isolate it sends `SIGINT`/`SIGTERM`, and Neon's pooler reclaims those connections for you, so an explicit drain handler is redundant.

> Reading `process.env.DATABASE_URL` directly works everywhere. The function in [Setup](#setup) instead uses `@neon/env`'s `parseEnv(config)` to read the same value in a typed, validated way — either is fine.

## WebSocket servers

A WebSocket server is the canonical Functions workload: a long-running handler holds connections open in-process, with no external state store needed to keep a stream coherent. Because a function is a real Node.js process (not a lambda), the WebSocket handshake works the way it does in any Node server — the [`ws`](https://github.com/websockets/ws) library upgrades the socket, and the connection stays alive as long as bytes flow (15-minute heartbeat, see [Timeouts](#timeouts-and-runtime-limits)).

**The return signature is the whole trick.** A function's default export is normally `{ fetch }`. To also accept WebSockets, export an `upgrade` method alongside it — the runtime routes plain HTTP to `fetch` and the WebSocket handshake to `upgrade`:

```typescript
export default {
  fetch(request: Request): Response | Promise<Response> { /* HTTP */ },
  async upgrade(req: IncomingMessage, socket: Duplex, head: Buffer) { /* WS handshake */ },
};
```

**Simple example** — raw `ws`, no framework, with auth. Browsers can't set headers on a WebSocket, so authenticate with a `?token=` query param (verify it the same way as the [agent backend](#functions-as-an-agent-backend-nextjs-and-similar-frameworks): `jwtVerify` against your JWKS) before accepting the connection:

```typescript
// src/index.ts
import type { IncomingMessage } from "node:http";
import type { Duplex } from "node:stream";
import { WebSocketServer, type WebSocket } from "ws";

const clients = new Set<WebSocket>();
const wss = new WebSocketServer({ noServer: true });

export default {
  // Plain HTTP (health checks, REST) is handled by fetch.
  fetch: () => new Response("WebSocket endpoint — connect with ?token=<jwt>"),

  // The runtime hands the WebSocket handshake to upgrade().
  async upgrade(req: IncomingMessage, socket: Duplex, head: Buffer) {
    const url = new URL(req.url ?? "/", "http://localhost");
    const identity = await verifyToken(url.searchParams.get("token")); // reject if invalid
    if (!identity) {
      socket.write("HTTP/1.1 401 Unauthorized\r\n\r\n");
      socket.destroy();
      return;
    }
    wss.handleUpgrade(req, socket, head, (ws) => {
      clients.add(ws);
      ws.on("close", () => clients.delete(ws));
      ws.on("message", (data) => broadcast(data.toString())); // see fan-out below
    });
  },
};
```

**Hono variant.** If you only need Hono for the HTTP side and are happy driving `ws` yourself, just swap `fetch` in the simple example for `app.fetch` and keep the raw `upgrade` — Hono serves routing/middleware, `ws` serves the socket.

To instead declare WebSocket routes _inside_ the Hono app — `app.get("/ws", upgradeWebSocket(...))` with the standard `onOpen`/`onMessage`/`onClose` lifecycle — you need an adapter that bridges Hono's `upgradeWebSocket()` helper to Neon's `upgrade(req, socket, head)`. Hono ships adapters for Cloudflare/Deno/Bun/Node, but **none for Neon**, and the Node one (`@hono/node-ws`) is deprecated and assumes it owns the HTTP server. [references/hono-websockets.md](references/hono-websockets.md) has a small self-contained `createNeonWebSocket(app)` adapter to copy in — it depends only on `hono` and `ws` (no deprecated package; adapted from `@hono/node-ws`, MIT) and returns a ready-to-export `{ fetch, upgrade }` handler. Usage is idiomatic Hono, and because the handshake routes through `app.request`, **auth is just normal route middleware**:

```typescript
// src/index.ts
import { Hono } from "hono";
import { createNeonWebSocket } from "./hono-ws";

const app = new Hono();
const { upgradeWebSocket, handler } = createNeonWebSocket(app);

app.get(
  "/ws",
  async (c, next) => {
    if (!(await verifyToken(c.req.query("token")))) return c.text("Unauthorized", 401);
    await next();
  },
  upgradeWebSocket(() => ({
    onOpen: (_evt, ws) => ws.send("welcome"),
    onMessage: (evt, ws) => ws.send(`echo: ${evt.data}`),
    onClose: () => console.log("disconnected"),
  })),
);

expo

<!-- Truncated for OpenGAP token limits -->
