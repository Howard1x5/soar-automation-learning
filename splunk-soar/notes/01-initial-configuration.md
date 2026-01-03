# 01 – Initial Configuration

This section aligns with Splunk’s *Administering Splunk SOAR* course, Topic 1: Initial Configuration.

The goal of this section is to understand how Splunk SOAR is structured and operates before attempting to build or run automations.

No labs are performed here. This is conceptual grounding only.

---

## Topic Scope (from Splunk course outline)

- Describe SOAR operating concepts  
- Identify documentation and community resources  
- SOAR & Splunk architecture  
- Product settings  
- Access control  
- Authentication settings  
- Response settings  
- Understanding roles  
- Creating users  
- Managing user access  
- Describe SOAR Automation Broker  

---

## 1. SOAR Operating Concepts

Splunk SOAR (formerly Phantom) is an orchestration and automation platform designed to:

- Ingest security events from multiple sources
- Normalize those events into containers and artifacts
- Execute actions against integrated tools (apps/assets)
- Automate decision-making via playbooks
- Track actions, decisions, and outcomes for auditability

At a high level, SOAR sits **above** detection tools and **beside** analysts:
- Detection tools generate alerts
- SOAR coordinates enrichment, response, and workflow
- Analysts intervene when human judgment is required

Key conceptual objects:
- **Containers**: Logical cases or incidents
- **Artifacts**: Individual pieces of data within a container (IP, domain, hash, user, etc.)
- **Apps**: Integrations with external tools
- **Assets**: Configured instances of apps (credentials, endpoints)
- **Actions**: Operations executed by apps against artifacts
- **Playbooks**: Logic that orchestrates actions and decisions

---

## 2. SOAR & Splunk Architecture (High Level)

Splunk SOAR and Splunk Enterprise / Cloud are separate systems that can be integrated.

General architectural relationship:
- Splunk ingests and indexes large volumes of log data
- SOAR consumes selected events or alerts from Splunk
- SOAR enriches, orchestrates, and responds
- Results can be written back to Splunk for visibility and reporting

Important distinctions:
- Splunk = data indexing, search, analytics
- SOAR = workflow, automation, case management

SOAR does not replace Splunk; it operationalizes outcomes from Splunk data.

Integration typically involves:
- Forwarding alerts or notable events from Splunk
- Mapping fields into SOAR artifacts
- Maintaining bidirectional visibility (alerts in, results out)

---

---

## 3. Containers vs Artifacts

Splunk SOAR models incidents using a two-level structure: containers and artifacts.

### Containers
A container represents a single case, alert, or incident. It is the top-level object used for:
- Case tracking
- Workflow execution
- Status and severity assignment
- Analyst collaboration and auditability

Examples of containers:
- A phishing email alert
- An EDR malware detection
- A suspicious login event

A container answers the question:
**“What happened?”**

---

### Artifacts
Artifacts are individual pieces of data associated with a container. They provide the raw inputs used by actions and playbooks.

Common artifact types:
- IP addresses
- Domains
- URLs
- File hashes
- Usernames
- Hostnames
- Email addresses

Artifacts answer the question:
**“What data do we have about this incident?”**

---

### Why the distinction matters
- Playbooks typically operate on artifacts, not containers directly
- Actions are executed against artifact values
- Poor artifact normalization leads to broken or skipped automation
- Multiple artifacts can exist within a single container, enabling parallel enrichment and decision-making

Understanding this separation is critical for:
- Designing reliable playbooks
- Debugging failed actions
- Controlling automation scope and impact


## Notes / Open Questions

- How containers are created (push vs pull)
- Where normalization can fail during ingestion
- What data should live in Splunk vs SOAR
- Version-specific behavior differences

These will be revisited after hands-on setup.

---

## Local Splunk Enterprise Installation (Lab Environment)

Splunk Enterprise was installed locally to support Splunk SOAR integration and hands-on study.

Environment notes:
- Installed Splunk 10.0.2 via `.deb` on Ubuntu
- Installed using `dpkg`
- Initial startup completed successfully
- Web UI accessible at `http://localhost:8000`
- Internal logs available and searchable (`index=_internal`)

This instance is intended as a learning lab and integration substrate, not a production deployment.

---

## Initial Data Ingestion Test

A local lab log file was ingested to validate end-to-end event handling.

- Custom sourcetype (`soar:test`) created and assigned
- Log file monitored from a controlled lab directory
- Events indexed into `main`
- Event preview used to validate timestamp parsing and structure
- Verified `_source`, `_sourcetype`, and `_host` values post-ingestion

This exercise establishes the baseline event structure that downstream alerts and SOAR automations will consume.

---

## Local Event Ingestion and Alert Creation (SOAR Signal Prep)

### Objective
Create a realistic, minimal event → alert pipeline in Splunk to ensure SOAR has deterministic, well-structured signals to consume. This validates that events, searches, scheduling, and alert actions behave as expected before introducing SOAR.

---

### Test Data Setup
- Created a local lab directory:
```

~/Documents/Projects/splunk-labs/

```
- Added a test log file:
```

soar_test.log

```
- Log entries simulate authentication activity:
- Successful logins
- Blocked actions
- Failed logins
- Purpose: emulate a common SOC signal (auth failures) without relying on external integrations.

---

### Data Ingestion
- Added file input via **Add Data → Files & Directories**
- Monitored:
```

~/Documents/Projects/splunk-labs/soar_test.log

```
- Created custom sourcetype:
```

soar:test

```
- Indexed data into:
```

index=main

```
- Verified events indexed and searchable.

---

### Detection Search
Created an aggregation search to summarize authentication failures:

- Groups by `user` and `ip`
- Counts failures
- Assigns severity based on failure count

This search is designed to:
- Reduce event noise
- Produce alert-friendly output
- Match common SOAR ingestion patterns (one alert per entity)

---

### Alert Configuration
Saved the search as a **Scheduled Alert**:

- Alert Name:
```

SOAR - Auth Failure Alert

```
- Schedule:
- Cron: `*/5 * * * *` (every 5 minutes)
- Time range: `Last 5 minutes`
- Trigger condition:
- Trigger when **Number of Results > 0**
- Trigger once per execution (not per result)

---

### Trigger Action
Configured a required trigger action to persist alert execution:

- Action: **Log Event**
- Logged fields:
- Alert name
- Result count
- Destination:
- `index=main`
- Default sourcetype

Purpose:
- Ensure alert execution produces an auditable event
- Validate end-to-end flow: event → alert → secondary event

---

### Outcome
- Confirmed:
- Custom sourcetype creation
- Local file ingestion
- Aggregation search behavior
- Cron-based alert scheduling
- Alert execution with logged output
- Resulting alert is suitable as a SOAR ingestion source.

---

### Alert Output Validation (Step 9)

Validated that the scheduled alert produces a concrete, searchable event suitable for downstream SOAR ingestion.

Validation steps:
- Executed search against alert-generated events:

---

### Notes
- SOAR depends on alerts built from events; skipping this layer leads to fragile automation.
- This exercise surfaced real configuration touchpoints:
- Sourcetypes
- Time ranges
- Cron scheduling
- Required alert actions
- This forms the baseline signal pipeline for later SOAR playbooks.
```

---
