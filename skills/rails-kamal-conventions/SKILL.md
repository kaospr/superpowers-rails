---
name: rails-kamal-conventions
description: Use when configuring Kamal for deployment, setting up deploy files, managing container registries, or configuring PostgreSQL for production
---

# Rails Kamal Conventions

Use the latest version of Kamal. Use a local image registry. Authenticate PostgreSQL with username/password.

## Core Principles

1. **Latest Kamal** - Always use the latest version of Kamal
2. **Local image registry** - Use a local registry, not Docker Hub or other remote registries
3. **PostgreSQL auth** - Always username/password authentication, never trust/peer
4. **Consistent env naming** - All PostgreSQL env vars use `POSTGRES_` prefix
5. **All databases configured** - Every database in `database.yml` (primary, cable, cache, queue, etc.) must be configured for production

## Deploy Configuration

```yaml
# config/deploy.yml
registry:
  server: localhost:5000

# PostgreSQL environment variables — use this exact naming
env:
  secret:
    - POSTGRES_USERNAME
    - POSTGRES_PASSWORD
  clear:
    POSTGRES_HOST: db
    POSTGRES_PORT: "5432"
    POSTGRES_DB: myapp_production
```

## PostgreSQL Setup

**Always** use username/password authentication. Never rely on trust or peer auth.

```yaml
# PostgreSQL accessory in config/deploy.yml
accessories:
  db:
    image: postgres:17
    port: "127.0.0.1:5432:5432"
    env:
      secret:
        - POSTGRES_USERNAME
        - POSTGRES_PASSWORD
      clear:
        POSTGRES_DB: myapp_production
    directories:
      - data:/var/lib/postgresql/data
```

```yaml
# database.yml — ALL databases must have production config
production:
  primary: &primary_production
    adapter: postgresql
    database: <%= ENV["POSTGRES_DB"] %>
    username: <%= ENV["POSTGRES_USERNAME"] %>
    password: <%= ENV["POSTGRES_PASSWORD"] %>
    host: <%= ENV["POSTGRES_HOST"] %>
    port: <%= ENV.fetch("POSTGRES_PORT", 5432) %>
  cache:
    <<: *primary_production
    database: <%= ENV["POSTGRES_DB"] %>_cache
    migrations_paths: db/cache_migrate
  queue:
    <<: *primary_production
    database: <%= ENV["POSTGRES_DB"] %>_queue
    migrations_paths: db/queue_migrate
  cable:
    <<: *primary_production
    database: <%= ENV["POSTGRES_DB"] %>_cable
    migrations_paths: db/cable_migrate
```

## Quick Reference

| Do | Don't |
|----|-------|
| Use latest Kamal version | Pin old Kamal versions |
| Local image registry (`localhost:5000`) | Docker Hub or remote registries |
| `POSTGRES_USERNAME` / `POSTGRES_PASSWORD` | `DATABASE_URL` or mixed naming |
| Username/password PostgreSQL auth | Trust or peer authentication |
| Consistent `POSTGRES_` prefix for all DB vars | `DB_USER`, `PG_PASS`, or other naming |

## Common Mistakes

1. **Using Docker Hub** - Use local registry instead
2. **Trust/peer PostgreSQL auth** - Always use username/password
3. **Inconsistent env var naming** - All DB vars must use `POSTGRES_` prefix
4. **Using `DATABASE_URL`** - Use individual `POSTGRES_*` variables instead
5. **Missing production database configs** - Every database in `database.yml` (primary, cache, queue, cable) must have a production configuration
