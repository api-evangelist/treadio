---
name: Authenticate and list projects
description: Get a bearer token, confirm the current user, and page through a company's projects.
api: openapi/treadio-horizon-openapi.json
operations: [post-v1-auth-login, get-v1-users-me, get-v1-projects]
---

# Authenticate and list Projects (Tread Horizon)

Base URL: `https://api.tread-horizon.com`. Every request needs `Authorization: Bearer <token>`
and `Accept: application/json`.

## Steps

1. **Get a token.** For a human/user flow call `post-v1-auth-login`
   (`POST /v1/auth/login`) with credentials; Stytch returns a short-lived JWT. For
   server-to-server use, exchange your Client ID/Secret via the OAuth2 client-credentials
   grant (request M2M credentials from developers@tread.io). See
   `authentication/treadio-authentication.yml`.
2. **Confirm identity.** Call `get-v1-users-me` (`GET /v1/users/me`) to verify the token and
   learn the current company context.
3. **List projects.** Call `get-v1-projects` (`GET /v1/projects`). Pass `page[limit]`
   (default 25, max 100). Follow the `Link` header `rel="next"` URL until it is absent;
   the `page[after]` cursor is opaque — do not parse it.

## Rules

- On `401 unauthorized`, fetch a fresh token and retry.
- Errors use the envelope `{ "error": { "code", "errors": [ {model, field, message} ] } }`
  (see `errors/treadio-problem-types.yml`). `code` mirrors the HTTP status.
- Set `Content-Type: application/json` on writes or you get `415 unsupported_media_type`.
