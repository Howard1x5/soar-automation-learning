Torq Automation Analyst – Q3 XML to JSON
1. Summary

This exercise focuses on data normalization and enrichment — specifically converting XML data into JSON for further processing inside a SOAR workflow.
The provided XML feed represents a bookstore catalog. The goal is to:

Retrieve the XML data.

Convert it into JSON.

Filter the dataset to include only books published in the year 2000.

Count how many books match that year.

Calculate the total combined price for those books.

2. Objective

Question: How many books were published in the year 2000, and what is their total price?

3. Workflow Design (Torq implementation)
Step 1. Trigger

Manual or HTTP trigger (no authentication required).

No input variables needed.

Step 2. Fetch XML Data

Action: HTTP Request

Method: GET

URL: https://hooks.torq.io/v1/webhooks/2893bc2a-8b35-4969-9540-f2ce6c6db146/workflows/63546432-76fc-4447-a7ea-acf8a88e1fad/sync

Expected Output: XML response body representing a catalog of <book> objects.

Step 3. Convert XML → JSON

Action: Convert to JSON

Input: The response body from the HTTP request.

Output Example (simplified): 
json
{
  "catalog": {
    "book": [
      { "id": "bk101", "author": "Gambardella, Matthew", "title": "XML Developer's Guide", "price": "44.95", "publish_date": "2000-10-01" },
      ...
    ]
  }
}
Step 4. Filter by Publish Year

Action: Filter Array (or conditional loop)

Input: {{ $.convert_to_json.output.catalog.book }}

Condition: item.publish_date starts with "2000"

Output: A filtered list of books published in the year 2000.

Step 5. Count Results

Action: Count Array

Input: Filtered array from the previous step.

Output Variable: count_2000

Step 6. Extract and Sum Prices

Action: Map Array → extract the price field and cast to number.

Action: Math Utilities – Sum

Output Variable: total_price_2000

Step 7. Exit / Output

Action: Exit

Output JSON:

{
  "count_2000": "{{ $.count_2000 }}",
  "total_price_2000": "{{ $.total_price_2000 }}"
}

4. Results (from the provided dataset)

Books published in 2000:

ID	Title	Price
bk101	XML Developer's Guide	44.95
bk102	Midnight Rain	5.95
bk106	Lover Birds	4.95
bk107	Splish Splash	4.95
bk108	Creepy Crawlies	4.95
bk109	Paradox Lost	6.95
bk110	Microsoft .NET: The Programming Bible	36.95
bk111	MSXML3: A Comprehensive Guide	36.95

Workflow Output Example:
{
  "count_2000": 8,
  "total_price_2000": 146.6
}

5. Python Reproduction (Local Equivalent)

If you don’t have Torq access, the same transformation can be demonstrated locally using Python.

File: scripts/solve_q3.py

import xml.etree.ElementTree as ET

xml_data = """<?xml version="1.0"?>
<catalog> ... full XML data here ... </catalog>
"""

root = ET.fromstring(xml_data)

books_2000 = []
for book in root.findall(".//book"):
    publish_date = book.findtext("publish_date")
    price = book.findtext("price")
    if publish_date and publish_date.startswith("2000"):
        books_2000.append(float(price))

count_2000 = len(books_2000)
total_price_2000 = round(sum(books_2000), 2)

print({
    "count_2000": count_2000,
    "total_price_2000": total_price_2000
})

Output:

{'count_2000': 8, 'total_price_2000': 146.6}

6. Equivalent Workflow in Shuffle SOAR
Step	Description	App / Function
1	Trigger: Manual trigger	Built-in
2	HTTP Request: Fetch XML data	HTTP app
3	Python Script Node: Convert XML→JSON and filter results	Python app
4	Set Variable / Return: Return count and total	Core output

import xml.etree.ElementTree as ET

xml_text = data["body"]
root = ET.fromstring(xml_text)

books_2000 = []
for book in root.findall(".//book"):
    pub = book.findtext("publish_date")
    price = book.findtext("price")
    if pub and pub.startswith("2000"):
        books_2000.append(float(price))

count_2000 = len(books_2000)
total_price_2000 = sum(books_2000)

return {"count_2000": count_2000, "total_price_2000": total_price_2000}

This same pattern applies to most SOAR tools:

Retrieve structured or semi-structured data.

Normalize it (convert XML→JSON or CSV→JSON).

Filter and aggregate.

Return clean structured output for follow-on playbooks or reporting.

7. Takeaways

Data normalization is key — most SOAR pipelines depend on JSON objects.

Looping vs. array utilities — Torq supports both; prefer native array actions for readability.

Cross-tool portability — the same logic applies across Torq, Shuffle, Splunk SOAR, and XSOAR.

Automation thinking — treat every data-conversion task as an opportunity to standardize input for downstream enrichment or decision logic.

Final Quiz Answer

Books published in 2000: 8

Total price of those books: 146.6