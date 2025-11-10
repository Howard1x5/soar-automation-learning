Intro to Data Transformation – Utilities

Purpose:
Learn how to use Torque utilities for data extraction, manipulation, and transformation in workflows. Utilities simplify handling JSON, logs, and tabular data, making it easy to pass results between steps and into external platforms like Slack.
Common Utility Categories

    Extract Fields – pull specific values from JSON objects or API responses.

    String & Array Utilities – join, split, encode, replace, or manipulate collections of data.

    Data Conversion – transform lists into formatted outputs (HTML, ASCII, CSV).

    Formatting & Presentation – send structured data to chat platforms or email.

Example 1 – Extract Fields from a Website

Goal: Fetch city names from a webpage, extract them, and send as a list to Slack.

Steps:

    Fetch Data

        Use an HTTP GET step to retrieve webpage content.

    Extract Fields

        Use Regex or Extract Fields utility to capture city names.

        Output: JSON list of city names.

    Join Strings

        Convert the list into a single string using the Join utility.

        Example: New York, London, Paris → New York | London | Paris

    Replace Characters

        Replace commas/quotes with Markdown formatting for cleaner Slack output.

        Add backticks (`) or spacing to improve readability.

    Send to Slack

        Deliver formatted results directly into a Slack message or snippet.

Example 2 – Parse Firewall Logs

Goal: Filter log entries for denied connections, highlight key values.

Steps:

    Fetch Logs via HTTP GET or local sample.

    Filter Data with utilities:

        Extract only DENY actions.

        Replace or capitalize specific keywords for readability.

    Transform Output

        Use count, replace, and string utilities to summarize.

        Example: Count denied events, uppercase protocol types.

    Send Results to Slack or save for reporting.

Example 3 – Convert User List to Tables

Goal: Convert API output of users into readable tables for Slack.

Steps:

    Extract Fields

        Example: user_id, profile.email, status

    Format as Table

        Use CreateHTMLTable → Produces HTML output for email/web display.

        Use CreateASCIITable → Produces ASCII output for Slack/message formatting.

    CSV Alternative

        Convert list into CSV format.

        Send to Slack as a snippet (.csv) → Slack automatically displays as a table.

Slack Output Options
Format	Utility Used	Best For
Plain String	Join Strings	Simple lists or inline values.
ASCII Table	CreateASCIITable	Quick, lightweight tables in Slack messages.
HTML Table	CreateHTMLTable	Clean tables for emails or web dashboards.
CSV Snippet	Convert to CSV	Slack-native table rendering.
Best Practices

    Always clean extracted data (quotes, commas, brackets) before presenting.

    Use Slack Snippets for large data (CSV/JSON).

    Use HTML tables for email reports, ASCII/Markdown tables for chat.

    Keep workflows modular — one step for extraction, one for formatting, one for delivery.

    Test with small datasets first before scaling to production data.

Practice Lab – Rebuild in Shuffle SOAR

Since Torque access may not be available, replicate these utility exercises in Shuffle SOAR (open source).

Lab 1 – Extract & Format Cities

    Create a workflow with an HTTP app to fetch dummy JSON data (e.g., {"cities": ["New York", "London", "Paris"]}).

    Add a JSON extractor step to isolate the cities field.

    Use the String Join step to merge into a single string (New York | London | Paris).

    Send the result into the Slack app (or a mock output if Slack is not configured).

Lab 2 – Parse Firewall Logs

    Import a sample JSON log with ALLOW and DENY actions.

    Use Filter Array to keep only entries with DENY.

    Apply Replace to highlight the word DENY in uppercase.

    Send the formatted results into Slack or output as text.

Lab 3 – Convert User Data to a Table

    Create a dummy dataset:

    [
      {"user_id": "101", "email": "alpha@example.com", "status": "active"},
      {"user_id": "102", "email": "beta@example.com", "status": "inactive"}
    ]

    Use the Table utility to convert this into ASCII or HTML.

    Output to Slack, or save as .csv and preview.

    Compare the difference between ASCII, HTML, and CSV outputs.

