<div align="center">

> [!INFO]
> This revival is practically dead. If you still want to host it yourself, here ya go.

# PurpRev

<img src="images/nonelogo.png" alt="Logo" width="320">

PurpRev is both a backend and frontend for [NoneRev](https://nonerev.pw).
<br>
**A revival that goes for the win.**

</div>

## Tech Stack

| Part | Built with |
| - | - |
| Frontend | [Svelte 5](https://svelte.dev) and [Vite](https://vite.dev) |
| Backend  | [Bun](https://bun.com) and [Elysia](https://elysiajs.com), API docs made by its OpenAPI plugin |
| Database | [PostgreSQL](https://www.postgresql.org) |

## Requirements

It really depends on how you run it, you can pick one below:

| Way                           | You need                                                             |
| - | - |
| [Local](#local)               | [Bun](https://bun.com) v1.4 or newer and a [PostgreSQL](https://www.postgresql.org) database |
| [Docker](#docker)             | [Docker](https://www.docker.com) with compose |
| [Podman](#podman-quadlets)    | [Podman](https://podman.io) on a machine with systemd |

## Setup

All three ways start the same.

**1. Clone the repo**

```bash
git clone https://github.com/nonerev/purprev.git
cd purprev
```

**2. Edit your `.env`**

```bash
cp .env.example .env
```

| Variable | What does it do? |
| - | - |
| `VITE_PORT` | Port of the frontend |
| `API_PORT` | Port of the backend |
| `PG_HOST` | Where the database is, `localhost` if its on the same machine |
| `PG_PORT` | `5432` unless you changed it |
| `PG_DATABASE` | Name of the database |
| `PG_USER` | Database login |
| `PG_PASSWORD` | Database password |
| `RCCMANAGER_HOST` | Where RCCManager is, `localhost` if its on the same machine |
| `RCCMANAGER_PORT` | RCCManager binding port, default `25500` |
| `RCCMANAGER_PASSWORD` | RCCManager password |
| `SETUP_URL` | Where the launcher and client zips are hosted |
| `TOOLBOX_ROBLOX` | `0` keeps studio's free models site only |
| `DISCORD_BOT_TOKEN`, `DISCORD_APPLICATIONS_CHANNEL`, `DISCORD_PUBLIC_KEY` | Where applications get posted, see [Applications](#applications) |
| `DISCORD_ADMIN_ROLE` | Who may press approve and deny, empty is anyone in the channel |
| `DISCORD_WEBHOOK_URL` | No bot? Post through a webhook instead, no buttons then |
| `TURNSTILE_SITE_KEY`, `TURNSTILE_SECRET_KEY` | Cloudflare Turnstile on signup and apply, empty is off |
| `CDN_URL` | The uploads bucket's custom domain, only for `STORAGE=cloudflare` |
| `ROBLOX_COOKIE` | A `.ROBLOSECURITY` cookie, needed for pulling assets from Roblox |
| `SITE_URL` | Public url, ends up in the join scripts the client gets |
| `SITE_INTERNAL_URL` | How RCCService reaches the site, a LAN address and never through Cloudflare |
| `JOIN_PRIVATE_KEY` | Path to the key that signs `Join.ashx`, default `keys/PrivateKey.pem` |
| `STORAGE` | `local` keeps uploads and renders in `UPLOADS_DIR`, `cloudflare` puts them in an R2 bucket behind `CDN_URL` |
| `UPLOADS_DIR` | The local folder, default `uploads/`, RCCManager writes here too when it is local |
| `R2_*` | Account, bucket, prefix and an API token with object read+write, only for `STORAGE=cloudflare` |
| `FIRST_USER_NAME`, `FIRST_USER_PASSWORD` | The admin account made on the first start, when there are no users yet. The name defaults to `NONEREV`; without a password a random one is made and printed once in the API log. Change it after logging in |

> [!WARNING]
> Do not commit `.env`. and PLEASE use throwaway account for `ROBLOX_COOKIE`.

Put the join script key in `keys/PrivateKey.pem` (gitignored). Game servers and renders come from RCCManager (its own repo), the site only talks to it over HTTP.

In Docker and Podman the database is its own container, so `PG_HOST` and `PG_PORT` get set for you. The database, user and password still come from `.env`.

## Running

### Locally (usually dev environment)

Setup a MySQL server and make a database and a user that match your `.env`:

```sql
CREATE DATABASE nonerev;
CREATE USER 'nonerev'@'%' IDENTIFIED BY 'change-me';
GRANT ALL PRIVILEGES ON nonerev.* TO 'nonerev'@'%';
```

After that, install the dependencies:

```bash
bun install
```

Then run these in two terminals:

```bash
# API
bun run dev

# frontend
bun run dev:web
```

`bun run start` runs the API without the watcher.

`bun run db:seed` puts the default avatars in the catalog, the Man and Woman bundles with their hair, shirt and jeans, owned by the first admin. Every new account gets one of the two and a random skin tone. Items an admin marks as Starter go to everyone on top.

Moving to R2 or bringing an old `uploads/` folder over: set `STORAGE=cloudflare` and run `bun run media:sync [folder]`. It records every file in the `media` table and pushes it to the bucket, skipping what is already there.

### Roblox Studio

| What | Where |
| - | - |
| Start page, log in here | `/ide/welcome` |
| Toolbox | `/IDE/ClientToolbox.aspx` |
| Publish to Roblox | `/IDE/publishas`, the place file goes to `/Data/Upload.ashx` |
| Publish Selection | `/UI/Save.aspx`, ends up in `assets` and under My Models |

Free Models also searches Roblox's toolbox, `TOOLBOX_ROBLOX=0` turns that off.

### Applications

No key? `/apply` takes a username, password and a few questions. Applications show up under Applications in the admin panel and in Discord:

1. Make a bot in the Discord developer portal, invite it with send messages in the channel
2. Put `DISCORD_BOT_TOKEN`, `DISCORD_APPLICATIONS_CHANNEL` and `DISCORD_PUBLIC_KEY` in `.env`
3. Set the Interactions Endpoint URL in the portal to `https://your.site/api/discord/interactions`, the site has to be deployed first

Approve makes the account on the spot, Deny asks for a reason. Both work from the buttons in Discord or from the admin panel. The applicant sees the answer on `/apply`.

### Docker

Build the image and start teh database, api and frontend:

```bash
docker compose up -d --build
```

```bash
# see what its doing
docker compose logs -f

# stop it, you can add -v to also delete the database
docker compose down
```

### Podman quadlets

TBA!!!

### Where everything is

Same for all three:

| What     | Where                           |
| -------- | ------------------------------- |
| Frontend | `http://localhost:5173`         |
| API      | `http://localhost:3000`         |
| API docs | `http://localhost:3000/openapi` |

Vite does proxy `/api` to the backend, so `http://localhost:5173/api` works too.

## Site themes

Themes live in their own repo, [nonerev/site-themes](https://github.com/nonerev/site-themes). Clone it into `vendor/site-themes`, vite copies it into `public/` when it starts.

## License

[AGPL-3.0](LICENSE). If you host a modified version, you have to share your changes with the people using it.
