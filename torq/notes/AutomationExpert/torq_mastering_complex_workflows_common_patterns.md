# Common Patterns

## 1. Pagination Pattern (SentinelOne example)

### Goal

Pull **all SentinelOne threats** using `get_recent_threats`, even though:

* The API returns a **limited number** of items per call (limit/page size).
* Additional pages are accessed via a **cursor** (`pagination.nextCursor`).

### High-Level Flow

1. Run `Get Recent Threats` with a **page size limit** (e.g., 10, 50, or 100).
2. Use a **loop** to repeatedly call `Get Recent Threats` until no `nextCursor` remains.
3. On each iteration:

   * Pass the previous `nextCursor` into the new API call.
   * Collect / flatten the `data` results into a single array.
4. Break the loop when `nextCursor == null` or empty.

### Torq Implementation

#### a) Initial sizing (optional but useful)

* Run `Get Recent Threats` once with:

  * `limit = 100`
* Inspect:

  * `api_object.pagination.nextCursor`
  * `api_object.pagination.totalItems` (if available)
* This gives you a **ballpark** of:

  * How many items exist
  * How many loop iterations you’ll need with a given page size

#### b) Set up the loop

* Add a **Loop step** (range type):

  * Start: `1`
  * End: generous upper bound (e.g., `500`)

    * If `limit = 100`, this gives capacity for **50,000** items.
    * You’re expecting to break early via cursor logic.

#### c) Paginated API call inside the loop

Inside the loop body:

* Use a second `Get Recent Threats` step, e.g. `get_recent_threats_page`.
* Set:

  * `limit = 10` (or your chosen page size)
  * `cursor = {{ $.steps.get_recent_threats_page.api_object.pagination.nextCursor }}`

    * On the **first run**, this reference is empty (no previous iteration)
    * SentinelOne then returns the first page.
    * On subsequent iterations, Torq resolves this variable to the **last** execution’s `nextCursor`.

Effectively:

* Iteration 1: cursor empty → fetch page 1 → returns `nextCursor_1`
* Iteration 2: cursor = `nextCursor_1` → fetch page 2 → returns `nextCursor_2`
* …
* Final iteration: returned `nextCursor` is `null` → signals last page.

#### d) Break condition (IF step)

After `get_recent_threats_page`, add an **IF** step:

* Condition:

  * Check `get_recent_threats_page.api_object.pagination.nextCursor`
  * If **not empty** → continue
  * If **empty/null** → break loop

In Torq:

* TRUE path (cursor exists): continue loop, don’t break.
* FALSE path (cursor missing/null): **break loop**.

#### e) Collect all results

Inside the loop, add a **Collector step**:

* Source:

  * `get_recent_threats_page.api_object.data`
* Options:

  * **Flatten** enabled, so each page’s array gets merged into one final array.

At the end of the loop:

* `collector.output` (or similar) will contain the **full threat list**.
* In the example:

  * Page size = 10
  * Total items = 42
  * Loop iterates 5 times
  * Collector length = 42

#### f) Notes & variations

* Other APIs may use:

  * `nextPageToken`
  * `offset + limit`
  * a `next` URL with an embedded cursor.
* Okta example:

  * Cursor is sometimes part of the **`Link` header** or embedded in the next URL.
  * You may need to parse that header/URL with split / regex before feeding it back in.
* General pattern remains:

  * Loop until **no more pages**.
  * Collect all `data` arrays into a **single flattened list**.

---

## 2. Waiting for a Response (Polling Pattern)

This is the **“submit job → poll until finished”** pattern, demonstrated with VirusTotal URL analysis.

### Classic Use Cases

* URL/file submitted to **VirusTotal**, Hybrid Analysis, sandbox, etc.
* Long-running **malware analysis**.
* Async **report generation**.
* Some cloud APIs that return a job ID and expect you to **poll status**.

### VirusTotal URL Scan Pattern (Torq Nested Workflow)

The Academy example uses a **nested workflow** step that:

* Accepts a list of URLs.
* For each URL:

  1. Checks if VT already has an **analysis report**.
  2. If not, submits the URL for **scanning**.
  3. Uses a **polling loop** to wait until results are ready.
  4. Returns the final status and attributes.

#### a) Submit the job

* Step: `submit_scan` (VirusTotal API step)
* Output:

  * `id` or `analysis_id` used for subsequent queries.

If submission fails:

* Set **failure status** and exit early (error or fallback path).

#### b) Polling loop

Add a **Loop** step (e.g., `1..50`):

* This represents the **maximum number of polling attempts**.
* Each iteration:

  1. Call VT “Get analysis” endpoint with the analysis ID.
  2. Check `data.attributes.status`:

     * If `"completed"` → break loop and persist result.
     * Otherwise → continue looping.

This matches the pagination pattern structurally, but logic is based on **status**, not `nextCursor`.

#### c) Break logic

* Use an IF step inside the loop:

Condition:

```text
data.attributes.status == "completed"
```

* TRUE path:

  * Store relevant fields (e.g., reputation, categories).
  * Break loop.
* FALSE path:

  * Loop again (possibly with a **Delay** step in between to avoid hammering the API).

#### d) Parent workflow usage

The VT polling logic lives in a **nested workflow**:

* Parent workflow calls nested “Scan URLs in VirusTotal”.
* Nested workflow handles:

  * Submission
  * Polling
  * Final result packaging
* Parent sees only the final **analysis status** and **summary fields**.

### General Polling Pattern (SOAR-Agnostic)

1. **Submit Job**

   * POST/PUT to an API.
   * Receive `job_id` / `analysis_id` / `task_id`.

2. **Initialize Loop**

   * Reasonable upper bound on attempts: 20–100.
   * Optional `Sleep`/`Delay` between attempts.

3. **Poll Status**

   * GET `/job/{id}` or equivalent.
   * Evaluate:

     * `status == "completed"` → success
     * `status == "failed"` → fail & break
     * Otherwise → loop again

4. **Timeout Handling**

   * If loop hits max attempts:

     * Mark as `"timeout"` or `"no_result"`.
     * Optionally raise an alert / create a ticket.

---

## 3. Pattern Templates (How You’ll Reuse These Later)

These two patterns are building blocks you’ll reuse across tools:

### Pagination – Reusable Concept

* **Trigger**: “I need *all* items, but the API only gives me a page at a time.”
* Template:

  * Loop from `1..N`
  * Call API with:

    * `cursor` or `page` or `offset`
  * IF:

    * Cursor/page token exists → continue
    * Else → break
  * Collect each page’s `data` into a single flattened array.

### Wait-for-Result – Reusable Concept

* **Trigger**: “I submitted something for processing and received a job ID.”
* Template:

  * Save `job_id`
  * Loop from `1..N`
  * Poll status API with `job_id`
  * IF:

    * `status == completed` → break, return result
    * `status == failed` → break with error
    * Else → wait (optional delay) and loop again