# Splunk SOAR (Phantom)

## 1. Purpose
This folder documents hands-on learning and experimentation with Splunk SOAR (Phantom), focused on automating security operations that are commonly handled manually in SOC environments.

The goal is not academic study or UI familiarity, but operational fluency: understanding how to design, build, troubleshoot, and maintain automations that reduce analyst workload and increase consistency.

## 2. Focus Areas
1. Building practical playbooks that mirror real SOC workflows  
   - Phishing triage  
   - Malware and EDR alerts  
   - Suspicious authentication and account activity  

2. Core platform mechanics  
   - Assets and authentication  
   - Actions and action results  
   - Playbook logic and control flow  
   - Custom functions  
   - REST API usage and external integrations  

3. Reliability and day-to-day operability  
   - Error handling in playbooks  
   - API failures and rate limits  
   - Data quality and parsing issues  
   - Asset misconfiguration and credential problems  
   - Ingestion and normalization edge cases  

4. Certification-aligned understanding  
   - Concepts and workflows aligned with the Splunk SOAR Certified Automation Developer (SCAD) exam  
   - Emphasis on hands-on capability over rote memorization  

## 3. Repository Structure
- `Notes/`  
  Platform behavior, patterns, gotchas, and troubleshooting references.

- `Exercises/`  
  Small, focused drills to practice specific concepts (filters, loops, parsing, conditions, error handling).

- `Labs/`  
  Guided builds and experiments with written context and outcomes.

- `Playbooks/`  
  Completed playbooks with explanations of intent, logic, and limitations.

- `Screenshots/`  
  UI screenshots referenced in notes and writeups.

## 4. How This Is Used
This folder serves as a working notebook and reference while learning Splunk SOAR:

- Capture what works and what breaks  
- Record repeatable patterns and anti-patterns  
- Build a personal library of automation logic that can be adapted across tools and environments  

It is intentionally iterative and experience-driven rather than polished or static.

## 5. Playbook Documentation Standard
Each completed playbook is documented with:
1. Purpose — what operational problem it solves  
2. Inputs and assumptions  
3. Logic flow and decision points  
4. Error handling and failure modes  
5. Known limitations and possible improvements  
