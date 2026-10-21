# Content API Service (V1)

Section content, validation, autosave with revision control, history/restore and duplication for the
Independent Website Builder. Express + PostgreSQL (BIGINT + JSONB) + Zod + Pino. See `docs/DECISIONS.md`
and `docs/openapi.yaml`.

## Run with Docker

```bash
cp .env.example .env        # then replace the placeholder secrets
docker compose up -d --build   # db + one-shot migrate + api on http://localhost:4080
curl localhost:4080/ready
```

## Run on the host

```bash
docker compose up -d db     # Postgres on localhost:5435
npm install
npm run migrate
npm start
```

## Auth

Every protected call sends `Authorization: Bearer <BRR/JWT>` plus `X-Workspace-Id`. Mint a local token with
`npm run token -- 9001 1001,1002 "content:*"`. Internal duplicate uses `X-Internal-Service-Token`.

## Postman

* `postman/Content-API.postman_collection.json` - 97 requests / 299 assertions (CRUD, all 8 section types, bulk,
  autosave + 409, revisions/restore, duplication, authz + workspace isolation, validation/security, soft delete).
* `postman/Content-API-Local.postman_environment.json` - committed with **empty** secrets.
* `npm run postman:env` writes `postman/Content-API-Local.postman_environment.local.json` with real local tokens
  (git-ignored) - import that one into Postman.
* `npm run postman:test` runs the whole collection with newman against the running API.
* `node scripts/build-postman.js` regenerates the committed files.

## Tests

`npm test` (unit) and `npm run postman:test` (integration, needs the stack running).

## Error codes

400 `BAD_REQUEST` `WORKSPACE_REQUIRED` · 401 `AUTHORIZATION_REQUIRED` `INVALID_TOKEN` · 403 `FORBIDDEN` ·
404 `CONTENT_NOT_FOUND` `SECTION_NOT_FOUND` `REVISION_NOT_FOUND` · 409 `CONTENT_REVISION_CONFLICT`
`CONTENT_ALREADY_EXISTS` · 413 `CONTENT_PAYLOAD_TOO_LARGE` · 415 `UNSUPPORTED_MEDIA_TYPE` ·
422 `CONTENT_SCHEMA_INVALID` `SECTION_TYPE_MISMATCH` `INVALID_LINK` `INVALID_MEDIA_REFERENCE` ·
429 `RATE_LIMITED` · 500 `INTERNAL_ERROR` · 502 `UPSTREAM_UNAVAILABLE`.
