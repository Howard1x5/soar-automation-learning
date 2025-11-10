Torq Exam – Token and GET Data (Shuffle SOAR Equivalent)

Overview  
This exercise reproduces the Torq Automation Analyst “Generate Token and HTTP GET Data” lab using Shuffle SOAR Cloud as an open-source equivalent.  

Goal:  
– Perform an HTTP POST request to generate an access token.  
– Save that token as a variable for reuse.  
– Use it in a follow-up HTTP GET request.  
– Demonstrate how authentication headers and variable passing work inside SOAR playbooks.  

---

Architecture of the Workflow  
Flow: Generate_Token → Save_Token → Get_Data  

Step 1 – Generate Token  
Action: HTTP POST  
URL: https://postman-echo.com/post  
Body: {}  
Purpose: Simulates an API authentication endpoint returning token-like JSON.  
Result: Returns HTTP 200 with dummy response later parsed in Step 2.  

Step 2 – Save Token  
Action: Set JSON Key  
Input: {{ Generate_Token.body }}  
Key: auth_token  
Value: 101 (for simulation)  
Purpose: Demonstrates storing authentication data between workflow steps.  

Step 3 – Get Data  
Action: HTTP GET  
URL: https://postman-echo.com/get  
Header: Authorization: Bearer {{ Save_Token.body.auth_token }}  
Purpose: Demonstrates authenticated data retrieval using stored token variable.  
Result: 200 OK with echoed header and response body confirming the token.  

---

Issues Encountered and Resolutions  

Issue: 401 Unauthorized with reqres.in  
Description: The reqres.in test API rejects unauthenticated calls from Shuffle Cloud.  
Resolution: Replaced with Postman Echo, which accepts public GET and POST requests.  

Issue: Autofill credentials  
Description: Browser autofill inserted username and password into HTTP fields.  
Resolution: Cleared browser autofill settings for app.shuffle.io and re-entered JSON manually.  

Issue: Missing “Set Variable” action  
Description: Shuffle’s action library differs from Torq; “Set JSON Key” performs the same role.  
Resolution: Used “Set JSON Key” to store the token value.  

Issue: Connection sequencing  
Description: Steps were not initially linked on the canvas, causing run failures.  
Resolution: Connected workflow nodes manually via drag-to-connect lines.  

---

Verification Results  

Generate Token – 200 OK – Returned simulated JSON body  
Save Token – Executed – Stored token in workflow context  
Get Data – 200 OK – Echoed Bearer token in header  

Sample Get_Data Output (abridged)  
args = {}  
headers.authorization = Bearer 101  
headers.host = postman-echo.com  
url = https://postman-echo.com/get  

---

Lessons Learned  
– SOAR token handling can be demonstrated without real authentication endpoints.  
– Public echo APIs like Postman Echo or JSONPlaceholder make the lab portable and safe.  
– Variable creation, reference syntax, and header injection are identical across Torq, Shuffle, XSOAR, and Splunk SOAR.  
– Always verify step outputs before chaining downstream variables.  

---