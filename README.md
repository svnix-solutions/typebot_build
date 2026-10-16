# typebot_build

Komodo stack for [Typebot](https://typebot.io) 3.19 (builder + viewer) on the
node's **shared** Postgres and Redis, behind Nginx Proxy Manager.

| Service | Image | Host port (loopback) | Public URL |
| --- | --- | --- | --- |
| `typebot-builder` | `baptistearno/typebot-builder:3.19.0` | 8090 | `TYPEBOT_BUILDER_URL` (editor, admin) |
| `typebot-viewer` | `baptistearno/typebot-viewer:3.19.0` | 8091 | `TYPEBOT_VIEWER_URL` (bots, chat API) |

No bundled database: Postgres from
[`postgres_build`](https://github.com/svnix-solutions/postgres_build) and Redis
from [`redis_build`](https://github.com/svnix-solutions/redis_build), over the
external `db` network. Both containers join `proxy` (shared with NPM only).

## Setup

1. Networks: `docker network create db; docker network create proxy`.
2. Database: `CREATE ROLE typebot LOGIN PASSWORD '<hex>'; CREATE DATABASE typebot OWNER typebot;`
   (or add `typebot:<hex>` to `postgres_build`'s `DB_PROVISION`). The builder
   runs migrations on start.
3. Fill the environment from `.env.example`. Builder and viewer must be on
   **different hostnames** (cookies ignore ports).
4. NPM proxy hosts: builder host → `typebot-builder:3000`, viewer host →
   `typebot-viewer:3000`, both with Websockets on.
5. Sign in to the builder with `TYPEBOT_ADMIN_EMAIL` (magic link by email). That
   account gets the `UNLIMITED` plan, which allows hiding Typebot branding:
   Typebot → Settings → General → *Typebot branding* off.

## Storage (S3 / B2)

Uploads use presigned `PUT`s from the browser, so the bucket's CORS must allow
`PUT` from every origin that runs a bot: the viewer host and any page that
embeds it.

- **File input blocks:** set *Visibility* to **Private**. Files then go to
  `private/…` and are served only through short-lived signed URLs, so a fully
  private bucket works (B2 has no per-prefix public policy).
- Media uploaded *into the editor* (avatars, images) goes to `public/…` and
  needs public read. With a private bucket, use image URLs hosted elsewhere.

## Notes

- `DISABLE_SIGNUP=true`: only invited users can join.
- Login is by email link, so configure SMTP before first use.
