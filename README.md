# LiteLLM Enterprise Proxy

A self-hosted LiteLLM proxy with OpenRouter, Gemini, PostgreSQL, Redis, and
Nginx.

## Architecture

```text
Client -> Nginx :8000 -> LiteLLM :4000 -> OpenRouter / Gemini API
                              |       |
                         PostgreSQL  Redis
```

## Local setup

Requirements: Docker Compose, an [OpenRouter API key](https://openrouter.ai/settings/keys),
and a [Google AI Studio API key](https://aistudio.google.com/apikey).

1. Copy `.env.example` to `.env`.
2. Set the database, Redis, LiteLLM, OpenRouter, and Gemini values. The
   `LITELLM_MASTER_KEY` value must start with `sk-`.
3. Start the services:

   ```powershell
   docker compose up -d
   docker compose ps
   ```

The API is available at `http://localhost:8000`.
The dashboard is available at `http://localhost:4000/ui/`.

The local PostgreSQL `latest` tag can move to a new major version. Migrate an
existing `pgdata` volume before recreating the database container.

## Create a client key

Replace `<LITELLM_MASTER_KEY>` with the value from `.env`:

```powershell
curl.exe http://localhost:8000/key/generate `
  -H "Authorization: Bearer <LITELLM_MASTER_KEY>" `
  -H "Content-Type: application/json" `
  -d '{"models":["openrouter-free","gemini-flash"]}'
```

Use the returned key as `VIRTUAL_KEY` in API requests.

## Chat request

```powershell
curl.exe http://localhost:8000/v1/chat/completions `
  -H "Authorization: Bearer <VIRTUAL_KEY>" `
  -H "Content-Type: application/json" `
  -d '{"model":"openrouter-free","messages":[{"role":"user","content":"Say hello."}]}'
```

Use `gemini-flash` as the model to call Gemini. `openrouter-free` uses
OpenRouter's free-model router.

## Managed PostgreSQL and Redis

Set `DATABASE_URL` and `REDIS_URL` in `.env`, then run Compose without the
local override:

```powershell
docker compose -f docker-compose.yml up -d
```

The default `docker compose up -d` includes local PostgreSQL and Redis.
