# mc_serverette

Minecraft server (Java + Bedrock) managed by [Crafty Controller](https://craftycontrol.com), run with Docker Compose on a VPS.
Admin access via Tailscale; public player access via playit.gg.

## Layout

| Path | Purpose |
|------|---------|
| `docker-compose.yml` | Crafty + optional playit agent |
| `.env` | `TS_IP`, `PLAYIT_SECRET` (git-ignored; copy from `.env.example`) |
| `crafty_data/` | Persistent Crafty data (config, servers, backups, logs, import) |

## Startup

Prereqs: Docker + Compose, Tailscale installed and logged in on the VPS, a playit.gg account.

1. `cp .env.example .env` and fill in:
   - `TS_IP` = `tailscale ip -4`
   - `PLAYIT_SECRET` = secret from playit.gg dashboard → Agents → Add (Docker)
2. Start:
   ```
   docker compose --profile playit up -d     # Crafty + playit
   docker compose up -d                      # Crafty only (Tailscale access)
   ```
3. In the playit dashboard add tunnels:
   - TCP (Java) -> `crafty:25565`
   - UDP (Bedrock) -> `crafty:19132`
4. Open `https://<TS_IP>:8443` (accept the self-signed cert warning).

## First login

Username `admin`. Password:
```
cat crafty_data/config/default-creds.txt
```
(or `docker exec crafty-controller cat /crafty/app/config/default-creds.txt`).
Change it after logging in. To pick your own, set `DEFAULT_USERNAME` / `DEFAULT_PASSWORD` (8+ chars) in the compose `environment`; only applies on first run, before the DB exists.

## Connecting

| Who | Java | Bedrock |
|-----|------|---------|
| Tailscale users | `<TS_IP>:25565` | `<TS_IP>:19132` |
| Public players | address shown in playit dashboard | UDP address shown in playit dashboard |

When creating a server in Crafty, set its port to match the tunnel (25565 Java / 19132 Bedrock). Ports 25565-25575 and 19132-19142 are mapped for extra servers.

## Operations

```
docker compose logs -f crafty        # logs
docker compose pull && docker compose --profile playit up -d   # update
docker compose down                  # stop (data stays in crafty_data/)
```
Back up `crafty_data/` (or use Crafty's built-in backups in `crafty_data/backups`).

## Notes

- Ports bind to `TS_IP` only, so nothing is exposed on the VPS's public IP. Playit connects outbound.
- Without `TS_IP` set, ports bind to `127.0.0.1`.
- Timezone is `Asia/Manila` (compose `TZ`).
- Verify the playit image tag in `docker-compose.yml` exists before first use.
