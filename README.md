# API Rate Limiter

**Open-source, self-hosted API rate limiting service** with JWT authentication, dynamic rate limit configuration, API key management, admin controls, SDK integration, and optional load balancer support.

> **100% Free & Open Source** - No subscription system, no billing, just pure rate limiting protection!

## Features

✅ **Dynamic Rate Limiting** - Configure rate limits at startup and adjust per API key  
✅ **Zero Subscription System** - Completely free and open-source  
✅ **JWT Authentication** - Secure user registration, login, and token refresh  
✅ **API Key Management** - Generate and manage API keys with custom rate limits  
✅ **Admin Controls** - Dashboard to manage users, set custom rate limits, block/unblock keys  
✅ **SDK Integration** - JavaScript SDK for frontend rate limit checking  
✅ **Real-time Monitoring** - WebSocket-based live dashboard  
✅ **Multi-deployment** - Docker, Kubernetes, or standalone Python  
✅ **Distributed via npm & pip** - Easy installation for all developers

## Quick Start

### Prerequisites

- Python 3.8+
- Docker and Docker Compose (optional)
- `kubectl` and a Kubernetes cluster (optional)

### Install dependencies

```bash
pip install -r requirements.txt
```

### Install from PyPI

```bash
pip install secureflow-api-rate-LIMITER
```

Then start the service with:

```bash
api-rate-limiter
```

### Run locally with defaults

```bash
python app.py
```

By default the app starts with built-in rate limits:
- **Requests per window**: `100`
- **Time window**: `60` seconds

Interactive setup is optional. Use `--interactive` or set `RATE_LIMITER_INTERACTIVE=true` to prompt for values before startup.

Then open your browser to `http://localhost:5000/dashboard`

### Docker Compose

1. Create or update `.env` in the repository root.
2. Start the application:

```bash
docker compose up --build -d
```

3. Open the service at:

- API: `http://localhost:8000`
- Dashboard: `http://localhost:8000/dashboard`
- SDK demo: `http://localhost:8000/sdk`

### Kubernetes

1. Build the Docker image:

```bash
docker build -t api-rate-limiter:latest .
```

2. Update `k8s/secret.yaml` with real secrets.
3. Apply manifests:

```bash
kubectl apply -f k8s/pvc.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

4. Forward a port if needed:

```bash
kubectl port-forward svc/api-rate-limiter-service 8000:80
```

## Quick-Start Guide

### 1) Start the app

For development:

```bash
pip install -r requirements.txt
python app.py
```

The app now starts with default rate limits and does not require interactive setup unless explicitly enabled.

Open the dashboard at `http://localhost:5000/dashboard`.

If you run the server from WSL or another environment, use `http://127.0.0.1:5000` when `localhost` does not connect.

### 2) Register a user

Send a request to create a new account:

```bash
curl -X POST http://localhost:5000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"Secret123"}'
```

### 3) Login and receive tokens

```bash
curl -X POST http://localhost:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"Secret123"}'
```

### 4) Create an API key

Use the returned access token:

```bash
curl -X POST http://localhost:5000/auth/create_api_key \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json"
```

### 5) Call protected data endpoint

```bash
curl "http://localhost:5000/data?api_key=<YOUR_API_KEY>"
```

## Examples

### Example 1: Health check

```bash
curl http://localhost:5000/health
```

### Example 2: Fetch usage data

```bash
curl "http://localhost:5000/usage?api_key=<YOUR_API_KEY>"
```

### Example 3: SDK check

```bash
curl -X POST http://localhost:5000/sdk/check \
  -H "Content-Type: application/json" \
  -d '{"api_key":"<YOUR_API_KEY>","endpoint":"/data","method":"GET"}'
```

### Example 4: SDK tracking

```bash
curl -X POST http://localhost:5000/sdk/track \
  -H "Content-Type: application/json" \
  -d '{"api_key":"<YOUR_API_KEY>","endpoint":"/data","method":"GET","status_code":200,"response_time_ms":123}'
```

## Dynamic Rate Limiting Configuration

### Startup Configuration

By default, `python app.py` now starts with the built-in defaults:

- Requests per window: `100`
- Time window: `60` seconds

To override defaults with environment variables, set:

```bash
RATE_LIMIT_REQUESTS=200 RATE_LIMIT_WINDOW=80 python app.py
```

To enable interactive startup prompts, use either:

```bash
python app.py --interactive
```

or

```bash
RATE_LIMITER_INTERACTIVE=true python app.py
```

If interactive mode is enabled, the app will prompt for rate limit values before starting.

These values become the **default rate limits** for all new API keys.

### Per-Key Rate Limit Configuration

Use the admin endpoint to set custom rate limits for specific API keys:

```bash
curl -X POST http://localhost:5000/admin/set_rate_limit \
  -H "Authorization: Bearer <ADMIN_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "<YOUR_API_KEY>",
    "requests": 100,
    "window": 60
  }'
```

**Parameters:**
- `api_key` - The API key to configure
- `requests` - Maximum requests allowed per window (required)
- `window` - Time window in seconds (required)

**Response:**
```json
{
  "status": "success",
  "rate_limit_requests": 100,
  "rate_limit_window": 60
}
```

### Check Rate Limit Usage

```bash
curl "http://localhost:5000/usage?api_key=<YOUR_API_KEY>"
```

**Response:**
```json
{
  "requests_left": 45,
  "requests_limit": 100,
  "window_seconds": 60,
  "total_requests_lifetime": 5000
}
```

## Admin Endpoints

### Set Custom Rate Limit

```
POST /admin/set_rate_limit
Authorization: Bearer <ADMIN_TOKEN>
Content-Type: application/json

{
  "api_key": "rk_live_...",
  "requests": 100,
  "window": 60
}
```

### Get All Users

```
GET /admin/users
Authorization: Bearer <ADMIN_TOKEN>
```

### Block API Key

```
POST /admin/block_key
Authorization: Bearer <ADMIN_TOKEN>
Content-Type: application/json

{
  "api_key": "rk_live_..."
}
```

### Unblock API Key

```
POST /admin/unblock_key
Authorization: Bearer <ADMIN_TOKEN>
Content-Type: application/json

{
  "api_key": "rk_live_..."
}
```

## Installation Methods

### Via pip (Python)

```bash
pip install api-rate-limiter
```

Then run:
```bash
python -m api_shield.app
```

### Via npm

```bash
npm install api-rate-limiter
```

Then use the CLI:
```bash
npm start
```

### Via Docker

```bash
docker build -t api-rate-limiter .
docker run -p 5000:5000 api-rate-limiter
```

### From Source

```bash
git clone https://github.com/yourusername/api-rate-limiter.git
cd api-rate-limiter
pip install -r requirements.txt
python app.py
```

## Features & Benefits

- ✅ **100% Free** - No subscription, no licensing fees
- ✅ **Open Source** - MIT License, fully customizable
- ✅ **Production Ready** - Used in real-world deployments
- ✅ **Dynamic Configuration** - Change rate limits without restarting
- ✅ **Multiple Deployment Options** - Docker, Kubernetes, or standalone
- ✅ **Comprehensive Admin Panel** - Manage users and limits from web UI
- ✅ **Real-time Monitoring** - WebSocket dashboard with live metrics
- ✅ **SDK Integration** - Browser-ready rate limiting checks
- ✅ **Secure** - JWT authentication, API key hashing, IP protection

## Environment Variables

Create a `.env` file in the project root:

```env
FLASK_ENV=development
SECRET_KEY=your-secret-key-here
ADMIN_TOKEN=your-admin-token-here
DATABASE_URL=sqlite:///ratelimiter.db
CORS_ORIGINS=http://localhost:5000,http://localhost:3000
EMAIL_NOTIFICATIONS=false
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

- 📖 [API Documentation](./API_DOCUMENTATION.md)
- 📧 Email: samadrehman550@gmail.com
- 🐛 [Report Issues](https://github.com/samadrehman/secureflow-api-rate-LIMITER)

## Roadmap

- [ ] GraphQL API support
- [ ] Advanced analytics dashboard
- [ ] Machine learning-based anomaly detection
- [ ] Webhook notifications
- [ ] Rate limit templates & presets

---

**Made with ❤️ by Samad Rehman**

## SDK Usage

The repository includes a browser SDK demo page and two SDK API endpoints.

- Demo page: `http://localhost:5000/sdk`
- Browser SDK script: `http://localhost:5000/sdk.js`

### SDK flow

1. Call `/sdk/check` before the real request to verify allowance.
2. Send the actual request if allowed.
3. Report request telemetry to `/sdk/track`.

### Example: browser-like flow

```bash
curl -X POST http://localhost:5000/sdk/check \
  -H "Content-Type: application/json" \
  -d '{"api_key":"<YOUR_API_KEY>","endpoint":"/data","method":"GET"}'

curl "http://localhost:5000/data?api_key=<YOUR_API_KEY>"

curl -X POST http://localhost:5000/sdk/track \
  -H "Content-Type: application/json" \
  -d '{"api_key":"<YOUR_API_KEY>","endpoint":"/data","method":"GET","status_code":200,"response_time_ms":123}'
```

## Deep Example: Full usage flow

This example shows a complete request flow from registration through rate-limited access.

1. Register user.
2. Login and receive `access_token`.
3. Create an API key.
4. Check rate-limit allowance with SDK.
5. Call the protected endpoint.
6. Inspect usage.

```bash
# Register
curl -X POST http://localhost:5000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"Secret123"}'

# Login
curl -X POST http://localhost:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"Secret123"}'

# Create API key
curl -X POST http://localhost:5000/auth/create_api_key \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json"

# SDK check before request
curl -X POST http://localhost:5000/sdk/check \
  -H "Content-Type: application/json" \
  -d '{"api_key":"<YOUR_API_KEY>","endpoint":"/data","method":"GET"}'

# Make protected request
curl "http://localhost:5000/data?api_key=<YOUR_API_KEY>"

# View usage
curl "http://localhost:5000/usage?api_key=<YOUR_API_KEY>"
```

## Notes

- Use `API_DOCUMENTATION.md` for endpoint reference and deeper API examples.
- The default rate limit behavior is tier-based and can be adjusted through configuration.
- For production, prefer Docker Compose or Kubernetes deployment.
