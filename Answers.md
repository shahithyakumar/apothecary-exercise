# SQL Injection Vulnerability in Search Box

## **Vulnerability Overview**
A critical **SQL Injection** vulnerability was identified in the **Apothecary Shop** application. This vulnerability allows an attacker to manipulate database queries through unsanitized user input, leading to unauthorized data retrieval and potential database compromise. The issue is particularly severe as **no authentication is required** to exploit it.

## **Vulnerability Details**
The application fails to properly sanitize user-supplied input in the **"name"** parameter of HTTP requests. This allows attackers to inject arbitrary **SQL queries**, which the backend executes.

SQL Injection occurs when user input is concatenated directly into a SQL query without proper escaping or parameterization. In this case, when a user provides input via the **name** parameter, the application constructs a SQL query dynamically. An attacker can inject **malicious SQL payloads** to alter query logic, extract sensitive data, or even modify the database contents.

This flaw is present in routes that accept user input without validation, making the system vulnerable to **SQL injection attacks**.

## **Impact**
- **Data Exposure**: Attackers can retrieve sensitive user and system data.
- **Data Manipulation**: Unauthorized users may modify, insert, or delete records.
- **Privilege Escalation**: Depending on database permissions, attackers may gain higher access rights.
- **Denial of Service (DoS)**: An attacker can craft malicious queries that cause the database to become unresponsive.
- **Complete System Compromise**: If the database has stored procedures that allow command execution, it may lead to **remote code execution**.

Given that authentication is **not required** to exploit this vulnerability, the risk is significantly increased.

## **Proof of Concept (PoC)**
### **Valid Request:**
```
GET /?name=test HTTP/1.1
Host: localhost:4000
Referer: http://localhost:4000/?name=a
Cookie: _apothecary_key=<session_token>
```
This request returns normal application responses.

### **Exploitable Request:**
```
GET /?name=test%27+OR+1%3D1%3B-- HTTP/1.1
Host: localhost:4000
Referer: http://localhost:4000/?name=test
Cookie: _apothecary_key=<session_token>
```
**Payload Used:**
```
test' OR 1=1;--
```
By injecting `OR 1=1`, an attacker forces the SQL query to always return **true**, potentially exposing all records from the database.

## **Remediation**
1. **Use Parameterized Queries**
   - Replace dynamic SQL queries with **parameterized queries**.
2. **Input Validation & Sanitization**
   - Restrict input characters and validate expected formats to prevent SQL manipulation.
---
---
---
---
# Cross-Site Scripting (XSS) Vulnerability in Review Submission Feature

## **Vulnerability Overview**
A **Cross-Site Scripting (XSS)** vulnerability was identified in the **review submission feature** of the **Apothecary Shop** application. The application does not properly sanitize user input in the review section, allowing an attacker to inject **malicious payloads**. This script executes when a user visits an affected potion’s review page, leading to **phishing, credential theft, session hijacking, and user redirection to malicious websites**.

## **Vulnerability Details**
The vulnerability is present in the **review input box** when submitting a potion review. The application does not sanitize or encode **HTML characters**, allowing an attacker to insert **arbitrary JavaScript code**.

When a user submits a review, the text is rendered on the page **without escaping special characters**. This allows an attacker to execute JavaScript in the context of a victim's browser.

## **Steps to Reproduce**
1. Navigate to `http://localhost:4000/potion/4`, which displays reviews for potion **#4**.
2. Submit a normal review (e.g., `"Test"`) to confirm functionality.
3. Submit a review with **HTML tags** (e.g., `<i>TestItalics</i>`). The text is displayed in italics, confirming that HTML is **not being sanitized**.
4. Submit a **malicious XSS payload**:
   ```
   <script>window.location.replace("http://google.com");</script>
   ```
5. Now, when any user visits `http://localhost:4000/potion/4`, they are **automatically redirected** to `google.com` or any malicious site.

## **Impact**
- **Phishing Attacks**: Attackers can inject forms or redirect users to fake login pages.
- **Session Hijacking**: If cookies are not secured properly, an attacker can steal session tokens.
- **Malware Distribution**: Users can be redirected to malicious sites hosting malware.

## **Proof of Concept (PoC)**
### **Valid Review Submission:**
```
POST /potion/review/4 HTTP/1.1
Host: localhost:4000
Content-Type: application/x-www-form-urlencoded

review[body]=Test&review[score]=5&review[email]=shahithya@shahithya.com
```

### **Malicious XSS Payload Submission:**
```
POST /potion/review/4 HTTP/1.1
Host: localhost:4000
Content-Type: application/x-www-form-urlencoded

review[body]=<script>window.location.replace("http://google.com");</script>&review[score]=5&review[email]=shahithya@shahithya.com
```
**Result:**  
Any user visiting `http://localhost:4000/potion/4` will be **automatically redirected** to `google.com` or any attacker-controlled site.

## **Remediation**
1. **Sanitize user input**:
   - Use libraries like **`html_sanitize_ex`** in Elixir to filter out scripts.
2. **Escape output before rendering**
3. **Set HTTP security headers**:
   - Implement **Content Security Policy (CSP)** to block inline scripts.
   - Example:
     ```
     Content-Security-Policy: default-src 'self'; script-src 'self'
     ```
4. **Use HTTPOnly and Secure cookies**:
   - Prevents JavaScript from accessing sensitive session cookies.
---
---
---
---
## **3. Action Reuse Vulnerability in User Bio Update**
An **Action Reuse Vulnerability** was identified in the **User Bio Update feature** of the **Settings** page. The application incorrectly allows user bio updates via both `POST` and `GET` requests, making it vulnerable to unintended modifications.

### **Vulnerability Details:**
When a user updates their bio via the settings page, the request is expected to be a `POST` request:
```
POST /users/settings/edit_bio
```
However, by simply changing the request method to `GET` and appending the `user[bio]` parameter from the POST request body to the URL:
```
GET /users/settings/edit_bio?user[bio]=SpidermanIsTheBest
```
The bio gets updated without requiring form submission, violating secure API design principles.

### **Remediation:**
1. **Restrict Updates to POST Requests**.
2. **Use Proper Authentication & Authorization Checks**.


