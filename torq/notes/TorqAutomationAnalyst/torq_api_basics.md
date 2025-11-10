Torq — REST API Fundamentals (Notes)
1) Core Ideas (Plain English)

    Resource: The thing you act on (users, alerts, incidents).

    Endpoint: URL for a resource (/api/users, /api/incidents/123).

    Method: What you want to do: GET (read), POST (create), PUT/PATCH (update), DELETE (remove).

    Headers: Metadata for the request (auth, content type).

    Body: Data you send with write operations (usually JSON or form-encoded).

2) Common HTTP Status Codes

    200 OK: Request succeeded (GET/PUT/PATCH/DELETE).

    201 Created: New resource created (POST).

    204 No Content: Success, nothing returned (often DELETE).

    400 Bad Request: Your input is wrong (missing/invalid fields).

    401 Unauthorized: Missing/invalid auth token.

    403 Forbidden: You’re authenticated but not allowed.

    404 Not Found: Endpoint or resource doesn’t exist.

    429 Too Many Requests: You hit rate limits.

    5xx Server Error: API provider is failing—retry with backoff.

3) Auth in Torq (High Level)

    Torq uses Bearer tokens (short‑lived, ~3600s).

    You generate a token by calling Torq’s auth server with client_id and client_secret (your “API key” pair).

    Include the token in the Authorization header for subsequent API calls:

        Authorization: Bearer <access_token>

4) Create a Torq API Key (Summary)

    In Torq UI: Avatar → API KEYS → Generate a Developer API Key

    Copy Client ID and Client Secret once (secret not shown again).

    Keys are account-scoped—requests run against that account.

5) Get a Bearer Token (cURL)

curl -X POST "https://auth.torq.io/v1/auth/token" \
  -H "Accept: application/json" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -u "<CLIENT_ID>:<CLIENT_SECRET>" \
  --data "grant_type=client_credentials"

Response (trimmed):

{
  "access_token": "xxxx-xxxx-xxxx",
  "expires_in": 3600,
  "token_type": "Bearer"
}

Use the access_token value in subsequent requests:

Authorization: Bearer <access_token>

6) Get a Bearer Token (Inside Torq)

    Step: Send an HTTP Request

        URL: https://auth.torq.io/v1/auth/token

        Method: POST

        Authorization: Basic (username = client_id, password = client_secret)

        Content-Type: application/x-www-form-urlencoded

        Body: grant_type=client_credentials

7) Making Authenticated API Calls (Pattern)

    Obtain access_token.

    Call target API with header:

        Authorization: Bearer <access_token>

    Handle non-2xx status codes (retry, fix, or fail gracefully).

Example (generic GET):

curl -X GET "https://example.com/api/resource" \
  -H "Authorization: Bearer <access_token>" \
  -H "Accept: application/json"

8) Practical Tips (What Interviewers Expect)

    Idempotency: Prefer idempotent methods (GET, PUT) for retries; use idempotency keys with POST if the API supports it.

    Pagination: Expect cursors or page/limit params; always handle partial lists.

    Rate Limiting: If 429, respect Retry-After header; implement exponential backoff.

    Error Handling: Log status_code, request_id (if provided), and response body for debugging.

    Security: Never commit secrets. Use Torq’s credentials management or a vault. Rotate keys periodically.

    Token Refresh: Track expires_in; re-auth before expiry (or on 401).

    Data Validation: Validate required fields before sending; sanitize user input.

9) Minimal Troubleshooting Checklist

    401? Check Authorization header format and that the token is fresh.

    403? Confirm account/role permissions for the endpoint.

    400? Verify request body/params and Content-Type.

    404? Check path/IDs; confirm the resource exists.

    429? Implement backoff and respect Retry-After.

    5xx? Retry with jitter; alert if persistent.

10) Future – GitHub Commit Guidelines (API Work)

When you start building Torq API workflows/scripts later, commit **only sanitized, portfolio-safe artifacts**:

- **Sanitized cURL examples**  
  - Replace secrets with placeholders (e.g., `<CLIENT_ID>`, `<CLIENT_SECRET>`, `<ACCESS_TOKEN>`).  
  - Keep only what’s needed to demonstrate the request/response pattern.

- **Short README for the example**  
  - What the call does (1–2 sentences).  
  - How to fetch a bearer token (high-level steps).  
  - One example endpoint call with required headers.

- **Optional: Torq workflow export + screenshot**  
  - Export of the workflow (YAML/JSON) with no secrets.  
  - Screenshot of the workflow canvas and/or a successful run.  
  - Brief note on status-code handling (e.g., 200/201 success, 4xx/5xx error paths).

- **Never commit secrets**  
  - No client IDs, client secrets, tokens, or tenant-specific URLs.  
  - Validate that environment variables / credentials are referenced, not embedded.

- **Good engineering hygiene**  
  - Show pagination/rate-limit handling if relevant.  
  - Include error-handling notes (what happens on 401/403/429/5xx).  
  - Link to any mock data files used for demos (sanitized).
