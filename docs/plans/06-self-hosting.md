# 06 — Self-Hosting Plan (your hardware)

**Invariant: nothing on your home network is directly reachable from the internet.**
Players reach one Cloudflare edge; Cloudflare reaches your box through an outbound tunnel.

---

## 1. Topology

```
Players (school Chromebooks, home PCs)
   │ HTTPS :443 / WSS
   ▼
Cloudflare edge  (TLS, WAF, rate-limit rules, Turnstile, hides your IP)
   │ outbound-only tunnel (cloudflared)
   ▼
┌──────────────────────── your box (VM/LXC) ─────────────────────────┐
│  cloudflared ──▶ caddy:80 ──▶ api:3000 (Fastify + WS + game loop)  │
│                                   │                                 │
│                                   ▼                                 │
│                             postgres:5432  (internal network only)  │
│                                   │                                 │
│                    backup (pg_dump + WAL) ──▶ NAS ──▶ B2 (offsite)  │
└─────────────────────────────────────────────────────────────────────┘
   Admin access: SSH over Tailscale only. No port forwards on the router.
```

## 2. Compose sketch

```yaml
# deploy/docker-compose.yml  (sketch — values come from .env, never committed)
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: openworld
      POSTGRES_USER: openworld
      POSTGRES_PASSWORD_FILE: /run/secrets/pg_password
    volumes: [pgdata:/var/lib/postgresql/data]
    networks: [internal]                # no published ports
    healthcheck: { test: ["CMD", "pg_isready", "-U", "openworld"], interval: 10s }
    restart: unless-stopped

  api:
    image: ghcr.io/thegeneral4757/openworld-api:${API_TAG}
    env_file: .env                      # DATABASE_URL, SESSION_PEPPER, TURNSTILE_SECRET, ...
    depends_on: { postgres: { condition: service_healthy } }
    networks: [internal, edge]
    restart: unless-stopped

  caddy:
    image: caddy:2
    volumes: [./Caddyfile:/etc/caddy/Caddyfile:ro]
    networks: [edge]
    restart: unless-stopped

  cloudflared:
    image: cloudflare/cloudflared:latest
    command: tunnel run
    environment: [TUNNEL_TOKEN=${CF_TUNNEL_TOKEN}]
    networks: [edge]
    restart: unless-stopped

networks: { internal: { internal: true }, edge: {} }
volumes: { pgdata: {} }
secrets: { pg_password: { file: ./secrets/pg_password } }
```

A separate `docker-compose.dev.yml` (Q33) runs a second Postgres + API on other ports for staging.

## 3. Hardening checklist

- [ ] Router: no port forwards for the game. SSH via Tailscale only, key auth, no root login.
- [ ] Postgres not published to the host; app role is **not** superuser; separate read-only role for dashboards.
- [ ] `admin_audit_log`: `REVOKE UPDATE, DELETE` from the app role.
- [ ] Unattended security upgrades on the host; pin image versions, update monthly.
- [ ] Cloudflare: WAF rate-limit rule on `/api/auth/*`; "Under Attack" toggle known.
- [ ] Trust `CF-Connecting-IP` only from the tunnel (Caddy `trusted_proxies`).
- [ ] Put the VM on its own VLAN/bridge if your network supports it (it's internet-facing code).
- [ ] Secrets in `.env` / Docker secrets, `.env` gitignored, rotation documented.

## 4. Backups & restore

| What | How | Retention |
|---|---|---|
| Logical dump | `pg_dump -Fc` nightly via cron container | 14 daily, 8 weekly |
| Point-in-time | WAL-G or pgBackRest → NAS | 7 days |
| Off-site | rclone/restic → Backblaze B2 (encrypted) | 8 weekly, 6 monthly |

Restore drill each phase: spin up the dev stack, restore last night's dump, run the e2e smoke
test against it. Write the time it took in this file.

## 5. Observability (minimum viable)

- Pino JSON logs → `docker logs` (later Loki).
- `/healthz` (process up) and `/readyz` (DB reachable) endpoints.
- Uptime Kuma pinging `https://api.example.com/healthz`, alerts to your Discord.
- Postgres: `pg_stat_statements` enabled; check slow queries monthly.

## 6. Capacity sanity check

At ≤ 200 accounts / ≤ 30 concurrent: the whole world (8,800 plots, a few thousand businesses,
chat history) is a few MB. A 2 vCPU / 4 GB VM is idle. The tight resource is **your home upload
bandwidth** — which is why the client/textures stay on a CDN (Pages/Cloudflare) and only API
JSON + WS events come from your box.
