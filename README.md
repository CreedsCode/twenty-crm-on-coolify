Twenty CRM on Coolify
=====================

Self-host **Twenty CRM** on your own infrastructure using **Coolify**, with Docker, PostgreSQL, Redis, and automatic HTTPS via Traefik.

This repository provides a **Coolify-ready `twenty-docker-compose.yml`** that lets Coolify fully manage networking, domains, and TLS certificates.


🚀 Features
-----------

-   Open-source CRM (Twenty)

-   Coolify-native deployment

-   Automatic HTTPS (Traefik + Let's Encrypt)

-   PostgreSQL 16

-   Redis 7

-   Persistent storage for uploads and database

-   Worker service for background jobs

-   Production-ready defaults



📦 Requirements
---------------

-   A server with **Coolify** installed

-   A domain name pointing to your Coolify server

-   Docker (managed by Coolify)

-   Optional:

    -   SMTP credentials (for email)

    -   Google / Microsoft OAuth credentials



🧱 Stack
--------

All images are pinned to exact versions for reproducible deploys (last reviewed 2026-07-18):

-   **Twenty CRM** -- `twentycrm/twenty:v2.22.0`

-   **PostgreSQL** -- `postgres:16.14-alpine`

-   **Redis** -- `redis:7.4.9-alpine`

-   **Reverse Proxy** -- Traefik (via Coolify)



📁 Repository Structure
-----------------------

```
.
├── docker-compose.yml
├── README.md
└── LICENSE

```


⚙️ Deployment (Coolify)
-----------------------

### 1\. Create a new Resource

-   Application type: **Docker Compose Empty**

-   Copy + Paste the `twenty-docker-compose.yaml` 


### 2\. Configure Domain

In **Coolify → Domains**:

-   Add your domain:

    ```
    crm.yourdomain.com

    ```

-   Enable **HTTPS**

-   Let Coolify handle certificates

Coolify will automatically inject:

-   `SERVICE_FQDN_SERVER`

-   `SERVICE_URL_SERVER`


### 3\. Environment Variables

In **Coolify → Configure → Environment Variables**, set at minimum:

```
SERVER_URL=https://crm.yourdomain.com
APP_SECRET=<openssl rand -base64 32>
ENCRYPTION_KEY=<openssl rand -base64 32>
PG_DATABASE_PASSWORD=<strong password>

```

Generate each secret with `openssl rand -base64 32` (use different values for
`APP_SECRET` and `ENCRYPTION_KEY`).

> ⚠️ **Back up `ENCRYPTION_KEY` somewhere safe.** Since Twenty v2.5 it encrypts
> secrets stored in the database (API keys, OAuth tokens, etc.). If you lose it,
> those secrets are unrecoverable. `FALLBACK_ENCRYPTION_KEY` is only needed
> during key rotation — leave it unset otherwise.

#### Optional (Email)

```
EMAIL_DRIVER=smtp
EMAIL_SMTP_HOST=smtp.yourprovider.com
EMAIL_SMTP_PORT=587
EMAIL_SMTP_USER=your_user
EMAIL_SMTP_PASSWORD=your_password
EMAIL_FROM_ADDRESS=contact@yourdomain.com
EMAIL_SYSTEM_ADDRESS=system@yourdomain.com

```

#### Optional (OAuth)

```
AUTH_GOOGLE_CLIENT_ID=
AUTH_GOOGLE_CLIENT_SECRET=
AUTH_GOOGLE_CALLBACK_URL=https://crm.yourdomain.com/auth/google/callback

AUTH_MICROSOFT_CLIENT_ID=
AUTH_MICROSOFT_CLIENT_SECRET=
AUTH_MICROSOFT_CALLBACK_URL=https://crm.yourdomain.com/auth/microsoft/callback

```


### 4\. Deploy

Click **Deploy** in Coolify.

Initial startup may take a few minutes while:

-   Database initializes

-   Migrations run

-   Worker starts


🔄 Upgrading
------------

Twenty releases fast (roughly weekly) and upgrades run database migrations, so
treat them as deliberate maintenance — never automatic. The images here are
pinned so nothing upgrades behind your back; `latest` is intentionally not used.

**Procedure:**

1.  **Read the release notes** for every version between yours and the target:
    <https://github.com/twentyhq/twenty/releases>. Watch for entries marked
    **[Breaking change]**.

2.  **Back up the database** (from the Coolify server, container name may differ):

    ```
    docker exec -t <db-container> pg_dumpall -U postgres > twenty-backup-$(date +%F).sql
    ```

    Migrations are one-way — you cannot roll back to an older image after they
    run, so a restore from backup is your only downgrade path.

3.  **Bump the version**: set `TAG=v2.x.y` in Coolify's environment variables
    (or edit the default in the compose file), using an exact tag — not `latest`.
    Since v1.22 you may jump several versions at once; the server applies all
    intermediate migrations automatically on startup.

4.  **Redeploy and watch the server logs.** First boot after an upgrade can take
    a while (migrations + occasional backfills). Don't interrupt it.

5.  **Verify**: log in, check `/healthz`, confirm background jobs run (worker
    container healthy).

Prefer upgrading to a release that has been out for a few days over the
just-published one — patch releases (e.g. `v2.20.0` → `v2.20.2`) often follow
within days. Upstream guide:
<https://docs.twenty.com/developers/self-hosting/upgrade-guide>

Postgres and Redis are pinned too. Patch bumps (e.g. `16.14` → `16.15`) are
safe to apply anytime; a Postgres **major** upgrade (16 → 17) requires a
dump/restore and should be done separately from any Twenty upgrade.


🩺 Health Checks
----------------

-   Application health endpoint:

    ```
    /healthz

    ```

Coolify uses this to determine container readiness.


🔐 Security Notes
-----------------

-   **Change `APP_SECRET` and `ENCRYPTION_KEY`** before production use, and keep `ENCRYPTION_KEY` backed up

-   Use HTTPS only (handled automatically by Coolify)

-   Back up your database volume regularly

-   Restrict Coolify access to trusted users


🛠 Customization
----------------

Common things you may want to customize:

-   Email provider (SMTP)

-   OAuth providers (Google / Microsoft)

-   Storage backend (local vs S3-compatible)

-   Disable migrations or cron jobs (advanced)


🧩 Troubleshooting
------------------

### App doesn't start

-   Ensure `APP_SECRET` is set

-   Check database health logs

-   Wait for migrations to complete

### Domain not routing

-   Confirm domain DNS points to the Coolify server

-   Verify domain is added in Coolify

-   Ensure HTTPS is enabled

### OAuth login fails

-   Callback URLs must match **exactly**

-   Ensure `SERVER_URL` uses HTTPS


📄 License
----------

This project is licensed under the **MIT License**.

See the `LICENSE` file for details.

* * * * *

🙌 Credits
----------

-   [Twenty CRM](https://twenty.com/)

-   [Coolify](https://coolify.io/)
