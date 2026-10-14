# Deploy Dify on Coolify

This repository keeps Dify's upstream `docker-compose.yaml` unchanged. Use
`docker-compose.coolify.yaml` only for a Coolify deployment.

## Coolify application settings

Create a Git repository application with the **Compose** build strategy.

| Setting | Value |
| --- | --- |
| Branch | `main` |
| Base directory | `/docker` |
| Docker Compose Location | `docker-compose.coolify.yaml` |
| Compose deployment | Normal (not Raw) |
| Preserve repository during deployment | Off |
| Escape special characters in labels | On |

After Coolify loads the file, add the public HTTPS domain to the **nginx**
service and set its internal port to `80`. Do not add a host port mapping:
Coolify's proxy owns ports 80 and 443 and provisions TLS.

## Environment variables

This Compose file reads `./.env`. In Coolify, open **Environment Variables**,
switch to advanced or bulk-edit mode, and import the contents of
`docker/.env.example`. Coolify writes those values to the Compose environment
file at deploy time; do not commit a `docker/.env` file.

Before the first deploy, replace these values with unique random strings:

- `SECRET_KEY`
- `DB_PASSWORD`
- `REDIS_PASSWORD`
- `SANDBOX_API_KEY`
- `WEAVIATE_API_KEY`
- `PLUGIN_DAEMON_KEY`
- `PLUGIN_DIFY_INNER_API_KEY`
- `DIFY_AGENT_API_TOKEN`
- `DIFY_AGENT_SERVER_SECRET_KEY`
- `DIFY_AGENT_LOCAL_SANDBOX_AUTH_TOKEN`

Keep these pairs synchronized after changing them:

- `CODE_EXECUTION_API_KEY` must equal `SANDBOX_API_KEY`.
- `WEAVIATE_API_KEY` must equal `WEAVIATE_AUTHENTICATION_APIKEY_ALLOWED_KEYS`.
- `PLUGIN_DIFY_INNER_API_KEY` is used by API, worker, plugin daemon, and agent backend.
- Set `CELERY_BROKER_URL` to `redis://:<REDIS_PASSWORD>@redis:6379/1`, using
  the actual `REDIS_PASSWORD` value (not the angle brackets).

Set all public URL variables to the one Coolify HTTPS domain before deploying:
`CONSOLE_API_URL`, `CONSOLE_WEB_URL`, `SERVICE_API_URL`, `APP_API_URL`,
`APP_WEB_URL`, `FILES_URL`, `TRIGGER_URL`, `ENDPOINT_URL_TEMPLATE`, and
`NEXT_PUBLIC_SOCKET_URL` (use `wss://` for the socket URL).

## If a previous deployment is still queued

Cancel it in the Coolify UI first. If a new deployment again remains blank for
more than a minute, the application has not reached Docker Compose. On the
Coolify server, inspect the control-plane containers rather than Dify:

```bash
docker ps --filter name=coolify
docker logs --tail 200 coolify
docker logs --tail 200 coolify-redis
```

Restart only the control-plane containers after collecting those logs:

```bash
docker restart coolify coolify-redis
```

Do not delete deployment records or Docker volumes while diagnosing a queue.
