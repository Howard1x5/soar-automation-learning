# Torq HTTP Request Step – Notes

## 1. Why the HTTP step matters

The HTTP Request step is the main building block for talking to external systems from Torq:

* Prototyping new integrations (REST, GraphQL, SOAP).
* Hitting internal tools or microservices that don’t have a native Torq app yet.
* Quickly testing API calls during workflow development.
* Turning working requests into reusable custom steps.

If you can drive an API with the HTTP step, you can automate it without waiting for a formal integration.

---

## 2. Core configuration: the “must-set” fields

When you drag in an **HTTP Request** step from the library:

* **URL**

  * Full URL, including query string.
  * Example: `https://postman-echo.com/get?foo=bar`.

* **Method** (default: `GET`)

  * Options: `GET`, `POST`, `DELETE`, `PATCH` (may vary by tenant version).
  * Choice controls which other fields are relevant (body vs query params, etc.).

* **Authorization**

  * `None` – no built-in auth; you can still add tokens via the **Headers** section.
  * `Basic` – username/password. Torq will send `Authorization: Basic <base64>` for you.
  * `Bearer` / `Token` (“Newer”) – paste in the token value; Torq will format the header.

* **Headers**

  * Key–value pairs like:

    * `Content-Type: application/json`
    * `Authorization: Bearer <token>` (if you’re not using the built-in auth dropdown).
    * API-specific headers like `x-api-key: <key>`.

* **Body** (for POST/PUT/PATCH)

  * Only visible/relevant for methods that support a body.
  * Usually JSON, but can be plain text or other types depending on **Content-Type**.

Torq’s execution log for this step shows:

* Request URL, query params, and headers (including Torq’s default user agent).
* Response body and headers.
* HTTP status code (key for success/failure logic).

---

## 3. Basic GET example (Postman Echo)

Simple pattern you can reuse for any GET API:

1. **Add HTTP Request step**

   * Method: `GET`
   * URL: `https://postman-echo.com/get?foo=bar&baz=qux`
   * Authorization: `None` (Postman Echo is public)

2. **Run step**

   * In the Execution Log:

     * Request section shows query args under some `args`/`params` object.
     * Headers show Torq’s default `User-Agent` (e.g., `Torq HTTP step 1.0.0`).
     * Response body echoes the args.
     * Status code: `200` indicates success.

3. **Notes**

   * For real APIs, swap `postman-echo` with the target endpoint and replicate the pattern.
   * Use the log to confirm your URL + params are being sent correctly.

---

## 4. Basic POST example

Pattern for creating/updating data via POST:

1. **Duplicate your GET step**

   * Rename original: `GET – Postman Echo`.
   * Duplicate → `POST – Postman Echo`.

2. **Configure POST**

   * URL: `https://postman-echo.com/post`
   * Method: `POST`
   * Content-Type: `application/json`
   * Body (JSON):

     ```json
     {
       "username": "torq-demo",
       "role": "analyst"
     }
     ```

3. **Run step**

   * Execution Log shows:

     * Request: method `POST`, JSON body under `data`/`json`.
     * Response body echoes JSON.
     * Status code: `200`.

4. **Observation**

   * For POST, body fields become part of `$.<step_name>.response.body` (exact path depends on your step name).
   * This is what you map into downstream operators (e.g., Collect, Switch, AI Task).

---

## 5. Authorization patterns

Three main ways you’ll authenticate:

### 5.1 Built-in Basic Auth

* In **Authorization**:

  * Choose `Basic`.
  * Fill `Username` and `Password` (they live under Optional parameters when Basic is chosen).
* Torq creates the header:

  ```text
  Authorization: Basic <base64(username:password)>
  ```

Use this for older / internal APIs that still use basic auth.

### 5.2 Built-in Token / Bearer

* Choose `Token` / `Bearer` / `Newer` (name depends on Torq UI version).
* Paste in the token string.
* Torq will automatically set something like:

  ```text
  Authorization: Bearer <token>
  ```

Use this for modern APIs with OAuth2 or static bearer tokens.

### 5.3 Manual header token

If the API uses a custom header style:

* Set Authorization to `None`.
* Add a header manually, e.g.:

  ```text
  x-api-key: <your_api_key>
  ```

or

```text
Authorization: ApiKey <your_api_key>
```

The execution log will show these under request headers for verification.

---

## 6. Optional parameters (gear icon) – when and why to tweak them

Inside **Optional parameters**:

* **Timeout (seconds)**

  * Default ~30s.
  * Increase for slow systems (on-prem, heavy reports, etc.).
  * Decrease if you want aggressive fail-fast behavior.

* **Skip SSL verification**

  * Use when hitting:

    * Internal systems with self-signed certs.
    * Lab/dev boxes with broken TLS chains.
  * Security note: only enable this for trusted internal environments.

* **Response encoding**

  * Option: “Return response encoded (Base64)” or “Return as file”.
  * Useful when:

    * Response is binary (PDF, CSV, image).
    * You want to avoid treating it as JSON.

* **Retries**

  * Retry on 5xx / 429 errors (rate limits).
  * Fields:

    * `Max retries`
    * `Delay between retries (seconds)`
  * Good for flaky APIs or rate-limited services.

* **Custom certificate**

  * Upload a CA or client cert for mutual TLS or private CAs.
  * Use for more secure internal integrations.

Tuning these is critical when you move from simple internet APIs to real internal enterprise systems.

---

## 7. Output handling

Two important modes:

1. **Default: JSON output**

   * Torq parses the response body as JSON.

   * You access it via something like:

     ```liquid
     {{ $.http_step_name.response.body }}
     {{ $.http_step_name.response.status_code }}
     ```

   * Best for REST/GraphQL APIs returning structured data.

2. **Return response as file**

   * Enabling this creates a Torq file (e.g., `.tqf`/`.bin`) with the raw response.
   * You can:

     * Mark it as shareable → Torq gives you a URL.
     * Feed it into other steps (parse CSV, send as email attachment, etc.).

Use JSON mode for data you want to branch/loop on. Use file mode for downloads, large exports, reports, etc.

---

## 8. Creating an HTTP step from a cURL command

You don’t have to build everything from scratch in the UI.

Workflow:

1. In Postman/terminal, get a working `curl` for the API call you want.
2. In Torq:

   * Create a new HTTP Request step.
   * Open the code/command area.
   * Paste the `curl` command directly.
3. Torq parses:

   * URL
   * Method
   * Headers
   * Body

This is extremely useful when reverse-engineering vendor docs that give `curl` examples.

---

## 9. Saving as a reusable custom step

Once you have a solid HTTP step that represents a reusable API action (e.g., “Get Alerts from Tool X”):

1. Open the step menu.
2. Choose **Save as Custom Step**.
3. Give it:

   * Clear name (e.g., `Get XDR Alerts`).
   * Tags/description.

Now it appears in your library like a native operator and can be dropped into other workflows without re-wiring auth/headers each time.

---

## 10. Cross-platform equivalents (Shuffle, XSOAR, Splunk SOAR)

The pattern here applies to every SOAR platform.

### 10.1 Shuffle SOAR

* Use the **HTTP** app / node.
* Fields mirror Torq’s:

  * URL, Method, Headers, Body, Auth.
* Optional features:

  * Timeout, verify SSL, retries.
* Output:

  * JSON body is available as `body` or `json` in subsequent nodes.
* For reusability:

  * Wrap in a subflow or create a dedicated workflow for “call this API once”.

**Debug habit:** Always inspect the raw output in a debug node after the HTTP call before building branching logic.

---

### 10.2 Cortex XSOAR (Demisto)

* Use the **GenericHTTP** / **Requests** integration:

  * Command: `http-get`, `http-post`, etc.
* Define:

  * `url`, `method`, `headers`, `body`, `verify`, `proxy`, `timeout`.
* Authentication:

  * Handled at the integration level (API key / basic / OAuth2).
* Playbooks:

  * Create a sub-playbook dedicated to each API action.
  * Use output context keys to standardize downstream usage.

**Debug habit:** Check the command results tab and the `Context` to ensure fields are mapped correctly before chaining tasks.

---

### 10.3 Splunk SOAR (Phantom / Splunk SOAR)

* Use the **REST** or **HTTP** app:

  * Action: `get`, `post`, `put`, etc.
* Set:

  * `url`, `headers`, `body`, `verify_server_cert`, `timeout`.
* Authentication:

  * Configured on the app asset (username/password, token, etc.).
* Output:

  * JSON output is accessible in playbooks via `action_results` and custom functions.

**Debug habit:** Examine `action_results` in the debugger, confirm response JSON paths, then write custom functions to normalize them.

---

## 11. Python reference pattern (for local testing / Shuffle / custom tools)

A simple Python pattern that mirrors what you’re doing in Torq:

```python
import requests
from typing import Any, Dict

def http_request(
    url: str,
    method: str = "GET",
    headers: Dict[str, str] | None = None,
    body: Dict[str, Any] | None = None,
    timeout: int = 30,
    verify_ssl: bool = True,
    retries: int = 0,
    delay: float = 0.0,
) -> requests.Response:
    method = method.upper()
    headers = headers or {}

    for attempt in range(retries + 1):
        try:
            if method == "GET":
                resp = requests.get(url, headers=headers, timeout=timeout, verify=verify_ssl)
            elif method == "POST":
                resp = requests.post(url, headers=headers, json=body, timeout=timeout, verify=verify_ssl)
            elif method == "DELETE":
                resp = requests.delete(url, headers=headers, timeout=timeout, verify=verify_ssl)
            elif method == "PATCH":
                resp = requests.patch(url, headers=headers, json=body, timeout=timeout, verify=verify_ssl)
            else:
                raise ValueError(f"Unsupported method: {method}")

            # Simple retry logic on 429 / 5xx
            if resp.status_code in (429, 500, 502, 503, 504) and attempt < retries:
                import time
                if delay:
                    time.sleep(delay)
                continue

            return resp

        except requests.RequestException:
            if attempt == retries:
                raise
            if delay:
                import time
                time.sleep(delay)

    # Should never reach here due to returns / raises above
    raise RuntimeError("Unexpected HTTP loop exit")


if __name__ == "__main__":
    r = http_request("https://postman-echo.com/get?foo=bar", method="GET")
    print("Status:", r.status_code)
    print("Body:", r.json())
```

Use this pattern when:

* Prototyping an API locally before wiring into a SOAR tool.
* Writing a tiny microservice that your SOAR platform will call.
* Porting Torq HTTP logic into Shuffle / custom Python steps.

---

## 12. Checklist: building a new HTTP integration in a SOAR workflow

When I wire a new API call (Torq, Shuffle, XSOAR, Splunk SOAR, or Python), I’ll:

1. Start from a known-good `curl` or Postman request.
2. Map **URL**, **method**, **headers**, **auth**, **body** into the HTTP step.
3. Run once and inspect:

   * Request details (URL, headers, body).
   * Response status code and body.
4. Fix:

   * TLS issues (skip verify vs custom cert).
   * Timeouts and retries for slow/flaky endpoints.
5. Decide on output format:

   * JSON for branching/logic.
   * File for downloads/attachments.
6. Stabilize and standardize:

   * Normalize field names in a follow-up step if the API is messy.
   * Wrap the HTTP call into a custom step or sub-workflow for reuse.
7. Only after I trust the HTTP call:

   * Add Switch/If logic, loops, or AI summarization on top of it.