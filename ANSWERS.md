<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

# Navigation

- [Vulnerabilities Found](#vulnerabilities-found)
   * [SQL Injection Vulnerability in Search Box](#sql-injection-vulnerability-in-search-box)
   * [Cross-Site Scripting (XSS) Vulnerability in Review Submission Feature](#cross-site-scripting-xss-vulnerability-in-review-submission-feature)
   * [Action Reuse Vulnerability in User Bio Update](#action-reuse-vulnerability-in-user-bio-update)
- [Remediation for Identified Vulnerabilities](#remediation-for-identified-vulnerabilities)
   * [Action Reuse Vulnerability](#action-reuse-vulnerability)
      + [Issue](#issue)
      + [Remediation](#remediation-3)
   * [Cross-Site Scripting (XSS) Vulnerability](#cross-site-scripting-xss-vulnerability)
      + [Issue](#issue-1)
      + [Remediation](#remediation-4)
- [Implementing a SAST Tool in the CI Pipeline](#implementing-a-sast-tool-in-the-ci-pipeline)

<!-- TOC end -->


----
----
----
----






<!-- TOC --><a name="vulnerabilities-found"></a>
# Vulnerabilities Found

<!-- TOC --><a name="sql-injection-vulnerability-in-search-box"></a>
## SQL Injection Vulnerability in Search Box

<!-- TOC --><a name="overview"></a>
### **Overview**
A critical **SQL Injection** vulnerability was identified in the **Apothecary Shop** application. This vulnerability allows an attacker to manipulate database queries through unsanitized user input, leading to unauthorized data retrieval and potential database compromise. The issue is particularly severe as **no authentication is required** to exploit it.

<!-- TOC --><a name="vulnerability-details"></a>
### **Vulnerability Details**
The application fails to properly sanitize user-supplied input in the **"name"** parameter of HTTP requests. This allows attackers to inject arbitrary **SQL queries**, which the backend executes.

SQL Injection occurs when user input is concatenated directly into a SQL query without proper escaping or parameterization. In this case, when a user provides input via the **name** parameter, the application constructs a SQL query dynamically. An attacker can inject **malicious SQL payloads** to alter query logic, extract sensitive data, or even modify the database contents.

This flaw is present in routes that accept user input without validation, making the system vulnerable to **SQL injection attacks**.

<!-- TOC --><a name="impact"></a>
### **Impact**
- **Data Exposure**: Attackers can retrieve sensitive user and system data.
- **Data Manipulation**: Unauthorized users may modify, insert, or delete records.
- **Privilege Escalation**: Depending on database permissions, attackers may gain higher access rights.
- **Denial of Service (DoS)**: An attacker can craft malicious queries that cause the database to become unresponsive.
- **Complete System Compromise**: If the database has stored procedures that allow command execution, it may lead to **remote code execution**.

Given that authentication is **not required** to exploit this vulnerability, the risk is significantly increased.

<!-- TOC --><a name="proof-of-concept"></a>
### **Proof of Concept**
<!-- TOC --><a name="valid-request"></a>
#### **Valid Request:**
```
GET /?name=test HTTP/1.1
Host: localhost:4000
Referer: http://localhost:4000/?name=a
Cookie: _apothecary_key=<session_token>
```
This request returns normal application responses.

<!-- TOC --><a name="exploitable-request"></a>
#### **Exploitable Request:**
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

<!-- TOC --><a name="remediation"></a>
### **Remediation**
1. **Use Parameterized Queries**
   - Replace dynamic SQL queries with **parameterized queries**.
2. **Input Validation & Sanitization**
   - Restrict input characters and validate expected formats to prevent SQL manipulation.
---
---

<!-- TOC --><a name="cross-site-scripting-xss-vulnerability-in-review-submission-feature"></a>
## Cross-Site Scripting (XSS) Vulnerability in Review Submission Feature

<!-- TOC --><a name="overview-1"></a>
### **Overview**
A **Cross-Site Scripting (XSS)** vulnerability was identified in the **review submission feature** of the **Apothecary Shop** application. The application does not properly sanitize user input in the review section, allowing an attacker to inject **malicious payloads**. This script executes when a user visits an affected potion’s review page, leading to **phishing, credential theft, session hijacking, and user redirection to malicious websites**.

<!-- TOC --><a name="vulnerability-details-1"></a>
### **Vulnerability Details**
The vulnerability is present in the **review input box** when submitting a potion review. The application does not sanitize or encode **HTML characters**, allowing an attacker to insert **arbitrary JavaScript code**.

When a user submits a review, the text is rendered on the page **without escaping special characters**. This allows an attacker to execute JavaScript in the context of a victim's browser.

<!-- TOC --><a name="steps-to-reproduce"></a>
### **Steps to Reproduce**
1. Navigate to `http://localhost:4000/potion/4`, which displays reviews for potion **#4**.
2. Submit a normal review (e.g., `"Test"`) to confirm functionality.
3. Submit a review with **HTML tags** (e.g., `<i>TestItalics</i>`). The text is displayed in italics, confirming that HTML is **not being sanitized**.
4. Submit a **malicious XSS payload**:
   ```
   <script>window.location.replace("http://google.com");</script>
   ```
5. Now, when any user visits `http://localhost:4000/potion/4`, they are **automatically redirected** to `google.com` or any malicious site.

<!-- TOC --><a name="impact-1"></a>
### **Impact**
- **Phishing Attacks**: Attackers can inject forms or redirect users to fake login pages.
- **Session Hijacking**: If cookies are not secured properly, an attacker can steal session tokens.
- **Malware Distribution**: Users can be redirected to malicious sites hosting malware.

<!-- TOC --><a name="proof-of-concept-1"></a>
### **Proof of Concept**
<!-- TOC --><a name="valid-review-submission"></a>
#### **Valid Review Submission:**
```
POST /potion/review/4 HTTP/1.1
Host: localhost:4000
Content-Type: application/x-www-form-urlencoded

review[body]=Test&review[score]=5&review[email]=shahithya@shahithya.com
```

<!-- TOC --><a name="malicious-xss-payload-submission"></a>
#### **Malicious XSS Payload Submission:**
```
POST /potion/review/4 HTTP/1.1
Host: localhost:4000
Content-Type: application/x-www-form-urlencoded

review[body]=<script>window.location.replace("http://google.com");</script>&review[score]=5&review[email]=shahithya@shahithya.com
```
**Result:**  
Any user visiting `http://localhost:4000/potion/4` will be **automatically redirected** to `google.com` or any attacker-controlled site.

<!-- TOC --><a name="remediation-1"></a>
### **Remediation**
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
<!-- TOC --><a name="action-reuse-vulnerability-in-user-bio-update"></a>
## **Action Reuse Vulnerability in User Bio Update**
An **Action Reuse Vulnerability** was identified in the **User Bio Update feature** of the **Settings** page. The application incorrectly allows user bio updates via both `POST` and `GET` requests, making it vulnerable to unintended modifications.

<!-- TOC --><a name="vulnerability-details-2"></a>
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

<!-- TOC --><a name="remediation-2"></a>
### **Remediation:**
1. **Restrict Updates to POST Requests**.
2. **Use Proper Authentication & Authorization Checks**.
---
---
---
---
<!-- TOC --><a name="remediation-for-identified-vulnerabilities"></a>
# Remediation for Identified Vulnerabilities

<!-- TOC --><a name="action-reuse-vulnerability"></a>
## Action Reuse Vulnerability

<!-- TOC --><a name="issue"></a>
### Issue
I identified an issue in the `router.ex` file that allows for action reuse. The problematic lines in `apothecary-exercise\lib\apothecary_web\router.ex` were:

```elixir
get "/users/settings/edit_bio", UserSettingsController, :edit_bio
post "/users/settings/edit_bio", UserSettingsController, :edit_bio
```

Both `GET` and `POST` requests were mapped to the same `edit_bio` action, which could lead to unintended consequences, such as CSRF or replay attacks.

<!-- TOC --><a name="remediation-3"></a>
### Remediation
To fix this, I removed the `GET` route and retained only the `POST` route:

```elixir
post "/users/settings/edit_bio", UserSettingsController, :edit_bio
```

After making this change, action reuse was no longer possible.

<!-- TOC --><a name="cross-site-scripting-xss-vulnerability"></a>
## Cross-Site Scripting (XSS) Vulnerability

<!-- TOC --><a name="issue-1"></a>
### Issue
The application was rendering user-generated content without sanitization, leading to an XSS vulnerability. Specifically, I found this issue in the file `apothecary-exercise\lib\apothecary_web\templates\potion\show.html.heex` on line 19:

```elixir
<div><%= raw review.body %></div>
```

Using `raw` directly means that any user input is rendered without encoding, allowing attackers to inject arbitrary JavaScript.

<!-- TOC --><a name="remediation-4"></a>
### Remediation
To prevent XSS, I replaced `raw review.body` with Phoenix's built-in HTML escaping function:

```elixir
<div><%= Phoenix.HTML.html_escape(review.body) %></div>
```

After implementing this, I was no longer able to execute XSS payloads in the review box.

---
---
---
---

<!-- TOC --><a name="implementing-a-sast-tool-in-the-ci-pipeline"></a>
# Implementing a SAST Tool in the CI Pipeline

To automatically detect these vulnerabilities as part of the CI pipeline, we can leverage static application security testing (SAST) tools.

- If **GitHub Advanced Security (GHAS)** is enabled, we can use **CodeQL** to scan the JavaScript code in the repository whenever a push or merge occurs.
- Since CodeQL does not support Elixir, we can use **Sobelow**, a static analysis tool for Elixir security.
- The pipeline will need two separate GitHub Actions workflows: one for JavaScript using CodeQL and another for Elixir using Sobelow.

Here’s a sample **GitHub Actions YAML configuration** to integrate both tools:

```yaml
name: SAST Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  codeql_scan:
    name: CodeQL Analysis (JavaScript)
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  sobelow_scan:
    name: Sobelow Analysis (Elixir)
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Elixir
        uses: erlef/setup-beam@v1
        with:
          elixir-version: '1.14'
          otp-version: '25'

      - name: Install dependencies
        run: mix deps.get

      - name: Run Sobelow Security Scan
        run: mix sobelow --exit
```

This pipeline ensures that:
- CodeQL scans JavaScript files for vulnerabilities.
- Sobelow scans Elixir files for security issues.
- Both scans run on every push and pull request to the `main` branch.

By integrating these tools into the CI/CD pipeline, we can proactively detect security vulnerabilities before they reach production.

