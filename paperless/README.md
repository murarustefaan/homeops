# paperless-ngx

Document archive at https://docs.stefanmuraru.com.

Auth: OIDC via Authentik. Legacy username/password login is retained during rollout;
flip `PAPERLESS_DISABLE_REGULAR_LOGIN=true` in `deployment.yaml` once OIDC is proven.

## Ingestion

- **Web UI** — Documents → Upload.
- **Email** — Settings → Mail → Accounts. Paperless polls IMAP on a schedule.
- **API / scanner apps** — POST to `/api/documents/post_document/` with a per-user API
  token generated in Settings → My Profile → API Auth Token.

## Parsing

`paperless-ngx` delegates to two supporting services in this namespace:

- `paperless-tika` — extracts text/metadata from non-image documents.
- `paperless-gotenberg` — converts office docs to PDF before OCR.

Both scale stateless; kill and recreate freely.
