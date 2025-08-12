Working with Variables in Torq
Module Summary

This module covers how to:

    Set variables (local and global)

    Update variables

    Delete variables

    Pass data between workflows using variables

    Control variable expiration

Key Concepts
1. Local Variables

    Scope: Only valid within the current workflow execution.

    Can store:

        String (e.g., "192.168.1.1")

        Number (e.g., 42)

        Boolean (e.g., true or false)

        JSON (arrays/objects)

Example:

["192.168.1.1", "8.8.8.8"]

    Use in a Loop step to iterate over each array element.

    Every step below the variable definition can use it until execution ends.

2. Global Variables

    Scope: Persistent storage, accessible across workflows.

    Useful for:

        Storing reusable data

        Passing information between workflows

    Can be given an expiration (e.g., 1 hour, 24 hours).

Key operations:

    Set Variable – Assign value to a unique key.

    Get Variable – Retrieve value using the key.

    Update Variable – Modify the stored value.

    Delete Variable – Remove the value entirely.

3. Practical Example: IP Address Storage

    Set a Local Variable

        Store an IP or array of IPs.

        Use loop to process each IP individually.

    Convert to Global Variable

        Assign a UUID or unique key name.

        Retrieve later using the Get Variable step.

    Update

        Modify part of the data (e.g., last octet in an IP).

    Delete

        Remove when no longer needed.

        Any future Get Variable calls will return empty.

Example Workflow: Passing Data Between Workflows

Workflow A

    Sets a global variable with key uuid-1234 and value "192.168.1.100".

Workflow B

    Retrieves uuid-1234 via Get Variable.

    Uses retrieved IP in security enrichment steps.

Cross-Platform Mapping
Torq Feature	Generic SOAR Concept	Shuffle / Open-Source Equivalent
Local Variable	Workflow execution variable	Context variable in Shuffle
Global Variable	Persistent cross-workflow	Global variable / Key-value store in Shuffle
Expiration Option	TTL (time-to-live)	Key expiry in Redis or Mongo-backed stores
Get Variable Step	Retrieve stored value	GET /variables/{key} in Shuffle API
Set Variable Step	Save variable	POST /variables in Shuffle API
Delete Variable Step	Remove variable	DELETE /variables/{key} in Shuffle API
Torq Variables – Quick Commands (API)

Set Global Variable

curl -X POST "https://api.torq.io/v1/variables" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"key": "uuid-1234", "value": "192.168.1.100", "expiration": "24h"}'

Get Global Variable

curl -X GET "https://api.torq.io/v1/variables/uuid-1234" \
  -H "Authorization: Bearer YOUR_TOKEN"

Delete Global Variable

curl -X DELETE "https://api.torq.io/v1/variables/uuid-1234" \
  -H "Authorization: Bearer YOUR_TOKEN"

Variables Practice Lab (Shuffle / Open SOAR)

    This lab replicates the Torq variables workflow but is fully runnable in Shuffle or any open-source SOAR platform.

Objective

    Create, update, retrieve, and delete both local and global variables.

    Pass data between two workflows without a direct trigger.

Lab Part 1 – Local Variable + Loop

    Create Workflow:
    Name: Variable_Lab_Local

    Add Step 1 – Set Local Variable

        Key: ip_list

        Value:

        ["192.168.1.1", "8.8.8.8"]

    Add Step 2 – Loop

        Source: ip_list

    Inside Loop:

        Add a GET request to https://ipinfo.io/{{loop.value}}/json

        Log or print the result.

Lab Part 2 – Global Variable

    Create Workflow:
    Name: Variable_Lab_Global_Set

    Step 1 – Set Global Variable

        Key: uuid-1234

        Value: "192.168.1.100"

        Expiration: 24h

    Run workflow – Variable is now stored in Shuffle’s key-value store.

Lab Part 3 – Retrieve & Update Global Variable

    Create Workflow:
    Name: Variable_Lab_Global_Get_Update

    Step 1 – Get Global Variable

        Key: uuid-1234

    Step 2 – Modify value

        Example: change last octet to .200

    Step 3 – Set Global Variable

        Key: uuid-1234

        New value: "192.168.1.200"

Lab Part 4 – Delete Global Variable

    Create Workflow:
    Name: Variable_Lab_Global_Delete

    Step 1 – Delete Global Variable

        Key: uuid-1234

    Step 2 – Attempt retrieval

        Expect empty result.

Expected Skills Gained

    Understanding variable scope

    Data passing between workflows without triggers

    Loop iteration over arrays

    State persistence with TTL

    Platform-agnostic SOAR variable handling

