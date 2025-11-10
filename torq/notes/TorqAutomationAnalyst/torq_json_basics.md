JSON — Fundamentals (Notes)
1) What is JSON?

    JavaScript Object Notation — a lightweight, text-based format for storing and transferring structured data.

    Language-independent: works with Python, Java, C, etc.

    Commonly used in APIs to send requests/responses between systems.

2) JSON Structure

    Keys: Always strings, usually descriptive names for data fields.

    Values: The data associated with each key.

    Key-Value Pair: "key": value

    Pairs are separated by commas; objects are enclosed in {}.

Example:

{
  "name": "John",
  "age": 30
}

3) JSON Data Types

Values in JSON can be one of six types:
Data Type	Example	Notes
String	"hello"	Text wrapped in quotes
Number	42	No quotes; can be integer or float
Boolean	true	Only true or false
Array	["red", "blue"]	Ordered list, can mix data types
Object	{ "key": "value" }	Collection of key-value pairs
Null	null	Explicit “no value”
4) Nested Data

    JSON supports nesting — arrays can contain objects, and objects can contain other objects or arrays.

    This makes it ideal for complex data structures.

Example of nesting:

{
  "name": "Jane Smith",
  "grades": ["A", "B", "C"],
  "address": {
    "street": "123 Main St",
    "city": "Anytown"
  },
  "phoneNumbers": [
    { "type": "home", "number": "555-555-1234" },
    { "type": "work", "number": "555-555-5678" }
  ],
  "email": null
}

5) Why JSON Matters in SOAR Workflows

    Many API integrations return JSON — knowing how to read it is essential.

    You’ll often extract specific values (e.g., status_code, email, alert_id) for use in later workflow steps.

    JSON structure determines how you write parsing logic in low-code/no-code steps or Python scripts.

6) Tips for Working with JSON

    Validate JSON format with tools like jsonlint.com or VS Code’s built-in linter.

    Always check for null or missing keys before processing values.

    Be aware of case sensitivity — "Name" and "name" are different keys.

    Use dot notation or bracket notation in code to access nested data:

        Dot: person.address.city

        Bracket: person["address"]["city"]

7) Future – GitHub Commit Guidelines (JSON Work)

When committing JSON examples to your portfolio later:

    Use mock or sanitized data (no real names, phone numbers, emails).

    Pair the JSON with a short README explaining:

        What the JSON represents.

        Key fields and their meanings.

        Example of extracting one or two fields in a workflow.

    Optional: include a workflow export that parses and uses the JSON.