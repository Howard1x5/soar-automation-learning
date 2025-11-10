Regex in SOAR Workflows (Torq‑first, tool‑agnostic)

What regex is good for in SOAR

Trigger filtering: accept/ignore events by matching channel names, usernames, or payload text. Example: only run when text contains ioc:\s*\S+.

Branching logic: use regex in if/switch to route flows. Example: separate IPv4 vs domain vs SHA256.

Field extraction: pull structured values (IPs, emails, ticket IDs) out of free text (alerts, emails, chat).

Payload shaping: with jq or JQCommand, combine regex with JSON filters to validate or transform fields.

Using regex in Torq (maps cleanly to other SOARs)

A. In triggers

Add Trigger Conditions → “matches (regex)” against event.text (chat), event.headers.* (email/webhook), or any field in the trigger event.
Tip: prefer case‑insensitive with (?i) (or the UI checkbox) so capitalization doesn’t break flows.

Quick examples:

Only run when message contains jq-test (case‑insensitive)
Pattern: (?i)\bjq-test\b

Webhook header must contain a GUID
Field: headers.X-Correlation-ID
Pattern: ^[0-9a-f]{8}(-[0-9a-f]{4}){3}-[0-9a-f]{12}$

B. In Switch / If statements

Switch (regex mode): test a field once and add multiple regex cases

^10.\d+.\d+.\d+$ → label “RFC1918 10/8”

^(?:172.(1[6-9]|2\d|3[0-1]).\d+.\d+)$ → label “RFC1918 172/12”

^192.168.\d+.\d+$ → label “RFC1918 192/16”

Fallback → label “Public”

If (regex) example: event.text matches (?i)\b(sev(ere)?|critical)\b

C. In steps (Utilities)

Extract by regex: pass text (body, headers, notes) → returns matches and capture groups.

Replace by regex: sanitize PII. Example: \b\d{3}-\d{2}-\d{4}\b replaced with --***.

Validate by regex: quick boolean gate before enrichment.

D. In jq / JQCommand (JSON pipeline)

test() ⇒ boolean guard
Example: select(.email | test("(?i)@example\.com$"))

match() ⇒ returns capture objects
Example: .subject | match("ID-(?<id>[A-Z0-9]{6})") | .captures[0].string

scan() ⇒ returns all matches as an array
Example: .notes | scan("\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b")

Battle‑tested regex snippets (copy/paste safe; patterns only—no secrets/tokens)

IPv4
Pattern: \b(?:25[0-5]|2[0-4]\d|1?\d?\d)(?:.(?:25[0-5]|2[0-4]\d|1?\d?\d)){3}\b
Notes: Matches 0–255 octets.

Domain
Pattern: \b(?=.{1,253}\b)(?:a-z0-9