# Architecture

How People of City is put together, for developers new to the project. Every path is from this repository.

```
┌──────────────────────────── Browser ────────────────────────────┐
│  Next.js page ── React Three Fiber <Canvas> ── the 3D city      │
│        │                └─ HUD (React DOM over the canvas)      │
│        │                                                        │
│        ├── Supabase JS ─────────────► Supabase                  │
│        │   (sign-in, profiles,        (Postgres + RLS, Auth,    │
│        │    connect, chat, live         Realtime channels)      │
│        │    chat messages)                    ▲                 │
│        │                                      │ secret key      │
│        ├── fetch /api/* ────────► Next.js API routes            │
│        │                          (apply, invite, connect,      │
│        │                           admin, cron)                 │
│        │                                                        │
│        └── WebSocket (binary) ──► realtime/ on Cloudflare       │
│            your position 10×/s    Workers + Durable Objects     │
│                                   (the live city)               │
└─────────────────────────────────────────────────────────────────┘
```

**Only the browser part is required.** Without Supabase the city runs as a guest walk-through with sample residents. Without the realtime server there are no live positions.

---

## Tech stack

| | |
|---|---|
| Framework | [Next.js](https://nextjs.org) 16 (App Router), React 19, TypeScript |
| 3D | [three.js](https://threejs.org) via [React Three Fiber](https://r3f.docs.pmnd.rs) and [drei](https://drei.docs.pmnd.rs); post-processing with `@react-three/postprocessing` |
| State | [zustand](https://zustand.docs.pmnd.rs) stores |
| Backend | [Supabase](https://supabase.com): Postgres with row-level security, Auth, Realtime |
| Live city | [Cloudflare Workers](https://workers.cloudflare.com) + Durable Objects (`realtime/`) |
| Tests | [vitest](https://vitest.dev) |

---

## Application structure

```
src/
├── app/                  Next.js routes
│   ├── page.tsx          → GameClient.tsx: loads the game in the browser only (no server rendering)
│   ├── layout.tsx        fonts, metadata, analytics
│   ├── admin/            the admin: review queue, invites, reports
│   ├── api/              server routes (see Backend)
│   └── opengraph-image.tsx, manifest.ts, robots.ts, sitemap.ts
├── proxy.ts              routes the admin subdomain to /admin (see src/lib/site.ts)
├── lib/                  site identity, palette, Supabase client, shared helpers
│   └── server/           server-only code (uses the secret key)
└── game/                 everything in the city
    ├── Game.tsx          the root: <Canvas> + the HUD
    ├── store.ts          the main game state (useGame)
    ├── districts/        the district registry and each district's definition
    ├── world/            the buildings, streets, props and the building kit
    ├── scene/            sky, light, day and night, post-processing
    ├── characters/       the character styles (roster) and avatar rendering
    ├── people/           residents: profiles, wandering, name plates
    ├── player/           your character: input, movement, collisions
    ├── hud/              every on-screen panel and button
    ├── net/              the live-city connection
    └── audio/            music, ambience and sound effects, synthesised
shared/net/               the live-city wire format and movement rules, shared with realtime/
realtime/                 the live-city server (Cloudflare Worker)
public/models/            character models (GLB)
public/audio/             sound files (see CREDITS.md)
```

## Rendering and the 3D environment

- **The root.** `src/game/Game.tsx` mounts one React Three Fiber `<Canvas>` with the sky (`scene/Sky.tsx`), lighting (`scene/Lighting.tsx`), the current district (`world/DistrictWorld.tsx`), the player, the people, the audio director and post-processing (`scene/Effects.tsx`). The HUD is ordinary React DOM layered over the canvas.
- **Built in code.** Buildings are not 3D files. They're assembled from a kit of bevelled pieces in `world/kit.tsx` (`Box`, `Win`, `Balcony`, `Sign`, `Awning`, …). Placement data lives in `game/layout.ts` (Koramangala) and `districts/hsr/layout.ts` (HSR).
- **Draw-call merging.** Thousands of kit pieces would be thousands of draw calls. `world/StaticMerge.tsx` bakes every static mesh into one merged mesh per material per chunk of the map once it mounts.
- **Light and time.** `scene/time.ts` keeps the city's clock (the real local time, or `?time=HH:MM`). `scene/daylight.ts` turns the hour into a look (sun, sky, fog, window lights), which the lights and materials read every frame.
- **Post-processing.** Ambient occlusion (N8AO), bloom, ACES filmic tone mapping, SMAA and a vignette, scaled by a quality level per device (`game/device.ts`). Phones get a lighter profile.
- **Textures.** Asphalt, pavers, grass, shutters, rangoli and every shop sign are drawn on canvases at runtime (`game/textures.ts`), with no image files.

## Districts

A district is a `DistrictDef` (`districts/active.ts`): its colliders, ground height, walking surfaces (`asphalt`, `pavers`, `grass`, `kota`), landmarks, spawn point, portal, hangout spots and trees. Each district registers itself with `registerDistrict(…)`, and `districts/index.ts` loads them and picks the default. `districts/registry.ts` lists all districts with their names and colours, including unbuilt ones that show as "coming soon". Walking into a portal opens `hud/PortalPicker.tsx`, and `hud/Travel.tsx` plays the transition.

## The avatar system

- **Styles.** `characters/roster.ts` defines each character style: a GLB model in `public/models/characters/`, a recolouring of its materials into the palette, parts to hide, and an alternate body.
- **Rendering.** `characters/Avatar.tsx` loads and clones the model, applies the style and plays its animations (idle, walk, run).
- **Your character.** `player/Player.tsx` drives the camera and your avatar from `player/input.ts` (keyboard, mouse, touch) and resolves collisions against the district's boxes (`player/collision.ts`).

## People and world interactions

- **Who's in the city.** `people/directory.ts` loads profiles from Supabase, or shows the sample residents in `people/profiles.ts` when there's no database.
- **Life on the street.** Offline residents wander between hangout spots (`people/wander.ts`, `people/spots.ts`); online residents are driven by their live positions.
- **Name plates.** `people/plates.ts` shows the name, role and status over each head, with less detail further away (`near` / `mid` / `far`) so a busy street stays readable.
- **Interaction.** `hud/Interaction.tsx` shows the prompt when you're near someone (`E`), and their profile panel opens with CONNECT.
- **Find someone.** `hud/Search.tsx` and `game/search.ts` (`F`); `game/tracking.ts` draws a route to the person.
- **Social.** Status (`game/status.ts`), connect (`game/connect.ts`, `hud/Connections.tsx`), chat (`game/chat.ts`, `hud/Chat.tsx`), invites (`game/invites.ts`), getting in (`game/access.ts`, `hud/Create.tsx`), and Day Zero missions (`game/missions.ts`).

## The live city

- **Client.** `game/net/client.ts` opens one WebSocket to the room for your district, sends your position 10 times a second (`SEND_HZ`) while it changes, and receives the people near you (`INTEREST_R`, 40 m). Members only: guests never connect.
- **Wire format.** `shared/net/protocol.ts` is a compact binary format: positions in centimetres, facing in 256 steps. `shared/net/validate.ts` holds the movement rules the server enforces (speed, bounds).
- **Server.** `realtime/src/room.ts` is one Durable Object per district shard (up to `ROOM_CAP` 150 people), holding positions in memory only and hibernating when idle. `realtime/src/city.ts` assigns shards and counts who's online per district. `realtime/src/auth.ts` checks the Supabase login on the first message.

## Backend

### Next.js API routes (`src/app/api/`)

| Route | What it does |
|---|---|
| `POST /api/apply/submit` | Submit an application |
| `POST /api/apply/checkout` | Send a saved application to review (applications are free in the foundation) |
| `POST /api/invite/redeem` | Join with an invite code |
| `POST /api/connect/send`, `/respond` | Send and answer a connect request |
| `POST /api/admin/approve`, `/reject` | Review applications (admins only) |
| `GET /api/cron/chat-digest` | Unread-message reminders, protected by `CRON_SECRET` |

Server code in `src/lib/server/` uses the Supabase **secret key** and never runs in the browser.

### Database

The starter database is in **`supabase/`**: `migrations/` (the steps, in order) and `schema.sql` (the same, as one file). It has 10 tables, all with row-level security on:

- **Read by the app directly** (row-level security decides what each person sees): `profiles`, `applications`, `member_settings`, `app_settings`, `reports`, `admins`
- **Closed to the app**, reached only through database functions: `invites`, `connections`, `messages`, `blocks`
- **Database functions** (called with `.rpc()`, security-definer, guarding everything with row-level security):
  - Invites: `check_invite`, `redeem_invite`, `my_invites`, `create_invite_link`
  - Applications: `submit_application`, `submit_application_for`, `approve_application`
  - Connect and chat: `send_connect_v2`, `respond_connect_v2`, `my_connections_v2`, `withdraw_connect`, `remove_connect`, `mark_connections_seen`, `send_message`, `thread_messages`, `mark_read`, `chat_digest_due`, `chat_mark_emailed`
  - Safety: `block_member`, `report_member`
  - Admin: `is_admin`, `admin_issue_invites`, `admin_founder_invites`, `admin_set_invite_note`, `admin_revoke_invite`, `admin_resolve_report`
- **Realtime:** one private channel per person, `user:<id>`, for new messages and connect updates.
- **Auth:** sign-in with a six-digit email code (`signInWithOtp` / `verifyOtp`).

Admins are people listed in the `admins` table, checked with `is_admin()`.

**Not in the starter database:** the scheduled job that runs the unread-message reminders (`/api/cron/chat-digest`). It needs Supabase's `pg_cron` and `pg_net` extensions and a real email provider; until you add both, there's nothing for it to send.

## External services

| Service | Used for | Required? |
|---|---|---|
| Supabase | Accounts, profiles, the social layer | For the social layer |
| Cloudflare Workers | The live city | Optional |
| Vercel | Hosting (the code reads `VERCEL_ENV` to tell production from previews) | Any Next.js host works |
| Google Analytics | Visitor analytics, production only | Optional (`GA_ID` in `src/lib/site.ts`) |
| Email provider | Fill in `src/lib/server/email/send.ts` | Optional (a filler today) |
| Payment provider | Fill in `src/lib/server/dodo.ts` | Optional (a filler today) |

## Important configuration files

| File | What it configures |
|---|---|
| `.env.example` → `.env.local` | Supabase keys, cron secret, realtime URL |
| `next.config.ts` | Which env values reach the browser, and noindex headers off production |
| `src/lib/site.ts` | Site name, address, admin hosts, analytics |
| `src/lib/palette.ts` | Every colour in the city |
| `src/game/districts/registry.ts` | The districts |
| `shared/net/config.ts` | Live-city limits: tick rate, interest radius, room size, world bounds, district ids |
| `realtime/wrangler.toml` | The live-city server's Cloudflare settings |
| `.github/workflows/checks.yml` | CI: tests and type-checks on every pull request |

## Tests

`npm test` runs vitest: the wire format (`shared/net/protocol.test.ts`), movement validation (`shared/net/validate.test.ts`), day and night (`src/game/scene/daylight.test.ts`), connections and missions. The same checks, plus type-checks of the app and the room server, run on every pull request.
