# API Documentation

This document contains API-level details for the Rate Limiter project.

## Base URLs

- Local API: http://localhost:5000
- Local Load Balancer: http://localhost:8080

## Authentication Modes

- API key query parameter: api_key
- JWT bearer token: Authorization: Bearer <token>
- Admin bearer token: Authorization: Bearer <ADMIN_TOKEN>

## Core API Endpoints

### GET /

Returns service info and a quick endpoint index.

Example response fields:
- message
- version
- endpoints

### GET /dashboard

Human-friendly web dashboard for setup, links, and endpoint summary.

### GET /health

Basic service health check.

### GET /data?api_key=YOUR_KEY

Protected sample endpoint using API key-based rate limiting.

Query parameters:
- api_key (required)

Common responses:
- 200: request accepted
- 401/403: invalid or blocked key
- 429: rate limit exceeded

### GET /usage?api_key=YOUR_KEY

Returns usage counters and remaining requests for the API key.

Query parameters:
- api_key (required)

Example response fields:
- requests_left
- requests_limit
- window_seconds
- total_requests_lifetime

### GET /api/metrics

Returns runtime metrics and counters used by dashboard and monitoring.

## SDK Endpoints

### GET /sdk

Serves the SDK setup/demo page.

### GET /sdk.js

Serves browser SDK script from static assets.

### POST /sdk/check

Pre-request rate-limit check for SDK-integrated clients.

Request body (JSON):
- api_key
- endpoint
- method

Example response fields:
- allowed
- tier
- limit
- remaining
- reset_at

### POST /sdk/track

Tracks request telemetry from SDK.

Request body (JSON):
- api_key
- endpoint
- method
- status_code
- response_time_ms

## Authentication Endpoints

### POST /auth/register

Create a user account.

Request body (JSON):
- email
- password
- organization_name (optional)

### POST /auth/login

Authenticate user and receive tokens.

Request body (JSON):
- email
- password

Common response fields:
- access_token
- refresh_token
- token_type
- user_id

### POST /auth/refresh

Refresh access token using a valid refresh token.

### POST /auth/logout

Invalidate current auth session/token where supported.

### GET /auth/me

Returns profile details for the current JWT identity.

### POST /auth/create_api_key

Creates API key for authenticated user.

Auth required:
- JWT bearer token

## Admin Endpoints

All endpoints in this section require admin bearer token.

### GET /admin/users

Returns user list and account metadata (without subscription information).

### POST /admin/set_rate_limit

Set custom rate limit for an API key.

Expected body fields:
- api_key (required)
- requests (required, integer >= 1)
- window (required, integer >= 1 seconds)

Example:
```json
{
  "api_key": "rk_live_...",
  "requests": 100,
  "window": 60
}
```

### POST /admin/block_key

Blocks an API key.

### POST /admin/unblock_key

Unblocks a previously blocked API key.

### GET /logs

Returns recent logs or log summaries.

### GET /admin/audit

Returns admin action audit trail.

## Load Balancer API

### GET /dashboard

Load balancer dashboard UI.

### GET /stats

Current backend health and load stats.

### POST /change_strategy

Change balancing strategy.

Common strategies:
- round_robin
- least_connections
- weighted
- ip_hash
- least_response_time
- adaptive

### ANY /{path}

Proxy request to selected backend.

## Dynamic Rate Limits

Default rate limits are configured at startup:
- Default requests per window: 100
- Default time window: 60 seconds

Each API key can have custom rate limits set via `/admin/set_rate_limit` endpoint.

Examples:
- 100 requests per 60 seconds (default)
- 50 requests per 30 seconds
- 1000 requests per 60 seconds

Set custom limits:
```bash
curl -X POST http://localhost:5000/admin/set_rate_limit \
  -H "Authorization: Bearer <ADMIN_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "rk_live_...",
    "requests": 100,
    "window": 60
  }'
```

## Error Model (Typical)

Most error responses are JSON with one or more of:
- error
- message
- details

Common status codes:
- 200 OK
- 201 Created
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 429 Too Many Requests
- 500 Internal Server Error

## cURL Quick Examples

Register:

curl -X POST http://localhost:5000/auth/register -H "Content-Type: application/json" -d '{"email":"user@example.com","password":"password123","tier":"free"}'

Login:

curl -X POST http://localhost:5000/auth/login -H "Content-Type: application/json" -d '{"email":"user@example.com","password":"password123"}'

Usage:

curl "http://localhost:5000/usage?api_key=YOUR_API_KEY"

Metrics:

curl "http://localhost:5000/api/metrics"
