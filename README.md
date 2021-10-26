# Mastodon
*Your self-hosted, globally interconnected microblogging community.*

Mastodon [official website](https://joinmastodon.org/) and [source code](https://github.com/tootsuite/mastodon/). This repository is a fork of [Wonderfall/docker-mastodon](https://github.com/Wonderfall/docker-mastodon).

## Why this image?
This non-official image is intended as an **all-in-one** (as in monolithic) Mastodon **production** image. You should use [the official image](https://hub.docker.com/r/tootsuite/mastodon) for development purpose or if you want scalability.

## Security
Images are scanned on push by [Trivy](https://github.com/aquasecurity/trivy) for OS vulnerabilities.

## Features
- Rootless image
- Based on Alpine Linux
- Includes [hardened_malloc](https://github.com/GrapheneOS/hardened_malloc)
- Precompiled assets for Mastodon

## Tags
- `latest` : latest Mastodon version (or working commit)
- `x.x` : latest Mastodon x.x (e.g. `3.4`)
- `x.x.x` : Mastodon x.x.x (including release candidates)

You can always have a glance [here](https://github.com/users/Wonderfall/packages/container/package/mastodon).

## Build-time variables
|          Variable         |         Description        |       Default      |
| ------------------------- | -------------------------- | ------------------ |
| **MASTODON_VERSION**      | version/commit of Mastodon |         N/A        |
| **REPOSITORY**            | source of Mastodon         | tootsuite/mastodon |

## Environment variables you should change

|          Variable         |         Description         |       Default      |
| ------------------------- | --------------------------- | ------------------ |
|           **UID**         | user id (rebuild to change) |         991        |
|           **GID**         | group id (rebuild to change)|         991        |
|    **RUN_DB_MIGRATIONS**  | run migrations at startup   |        true        |
|    **SIDEKIQ_WORKERS**    | number of Sidekiq workers   |          5         |

Don't forget to provide [an environment file](https://github.com/tootsuite/mastodon/blob/main/.env.production.sample) for Mastodon itself.

## Volumes
|          Variable            |         Description        |
| -------------------------    | -------------------------- |
| **/mastodon/public/system**  |         data files         |
| **/mastodon/log**            |            logs            |

## Ports
|              Port            |            Use             |
| -------------------------    | -------------------------- |
| **3000**                     |        Mastodon web        |
| **4000**                     |      Mastodon streaming    |

## docker-compose example
Please use your own settings and adjust this example to your needs.
Here I use Traefik v2 (already configured to redirect 80 to 443 globally).

```yaml
version: "3.8"

networks:
  mastodon_network:

services:
  mastodon:
    image: ghcr.io/ropable/mastodon
    env_file: /wherever/docker/mastodon/.env.mastodon
    depends_on:
      - postgres
      - redis
    volumes:
      - /wherever/docker/mastodon/data:/mastodon/public/system
      - /wherever/docker/mastodon/logs:/mastodon/log
    networks:
      - mastodon_network
   redis:
    image: redis:6-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
    volumes:
      - /wherever/docker/mastodon/redis:/data
    networks:
      - mastodon_network
  postgres:
    image: postgres:14-alpine
    volumes:
      - /wherever/docker/mastodon/db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
    env_file: /wherever/docker/mastodon/.env.postgres
    networks:
      - mastodon_network
```
