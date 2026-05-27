This module is about one of the MOST important realities in cybersecurity:

# “Errors leak information.”

Developers usually think:

> “Error page = something broke.”

But attackers think:

> “Error page = free information.”

This entire section is teaching you:

* how applications leak internal details
* how attackers use those details
* how logs reveal attacks
* how error messages help enumeration
* how stack traces expose technologies

This is a BIG real-world topic.

---

# What This Module Covers

From your screenshot:

Checklist:

1. Different types of error pages
2. HTTP RFC 7231 error codes
3. Analyze error codes and stack traces
4. Username enumeration
5. Analyze IIS and Apache logs

This means this module is about:

* HTTP errors
* Server behavior
* Information disclosure
* Log analysis
* Recon through failures

---

# Core Idea of This Entire Module

When applications fail:
they often accidentally reveal:

* technologies
* paths
* usernames
* database details
* programming language
* frameworks
* internal architecture

Attackers LOVE errors.

---

# 1. Different Types of Error Pages

Task:

> Find five different types of error pages

---

# What is an Error Page?

When something goes wrong:
server returns an error response.

Examples:

* page missing
* unauthorized
* server crash
* forbidden resource

---

# Why Error Pages Matter

Error pages reveal:

* server type
* framework
* application structure
* internal paths
* sometimes credentials

---

# Common Error Pages

---

# 200 OK

Means:

> Request successful

```http
HTTP/1.1 200 OK
```

Not an error.

---

# 301 / 302 Redirect

Means:

> Page moved somewhere else

Example:
redirect to login page.

---

# 403 Forbidden

Means:

> Resource exists, but access denied

Example:

```txt
/admin
```

exists but blocked.

---

# 404 Not Found

Means:

> Resource does not exist

Example:

```txt
/test.php
```

not found.

---

# 500 Internal Server Error

Means:

> Server crashed internally

VERY interesting for attackers.

---

# 502 Bad Gateway

Usually reverse proxy issue.

Often seen in:

* Nginx
* load balancers

---

# 503 Service Unavailable

Means:

* server overloaded
* maintenance
* backend unavailable

---

# 401 Unauthorized

Means:

> Authentication required

---

# Why Different Error Pages Matter

Different applications:

* reveal different technologies
* leak different information

Example:

Apache error page:

```txt
Apache/2.4.49 Server at ...
```

ASP.NET error:

```txt
System.NullReferenceException
```

PHP error:

```txt
Warning: include()
```

Java error:

```txt
java.lang.NullPointerException
```

Now attacker knows:

* programming language
* framework
* possible vulnerabilities

---

# 2. RFC 7231 — HTTP Semantics

Your screenshot mentions:

> RFC 7231

This is VERY important.

---

# What is RFC?

RFC =

> Request For Comments

Official internet standards documentation.

---

# RFC 7231 Defines

How HTTP should behave:

* methods
* responses
* status codes
* semantics

---

# Important HTTP Methods

---

# GET

Retrieve data.

Example:

```http
GET /index.html
```

---

# POST

Send data.

Example:

```http
POST /login
```

---

# PUT

Update resource.

---

# DELETE

Delete resource.

---

# HEAD

Get headers only.

---

# OPTIONS

Ask server:

> What methods allowed?

---

# Why Attackers Care

Sometimes dangerous methods enabled:

* PUT upload
* DELETE
* TRACE

Misconfiguration can lead to compromise.

---

# Important Status Codes

HTTP status codes are grouped.

---

# 1xx = Informational

Rarely important.

---

# 2xx = Success

* 200 OK
* 201 Created

---

# 3xx = Redirection

* 301
* 302

---

# 4xx = Client Errors

User/request problem.

Examples:

* 401
* 403
* 404

---

# 5xx = Server Errors

Server problem.

Examples:

* 500
* 502
* 503

These are gold mines sometimes.

---

# 3. Stack Traces and Sensitive Information

Task:

> Analyze error codes and stack traces

This is VERY important in web security.

---

# What is a Stack Trace?

When application crashes:
programming framework prints debugging info.

Example:

```txt
java.lang.NullPointerException
at login.java:45
```

or:

```txt
Fatal error in db.php line 82
```

---

# Why Dangerous?

Stack traces reveal:

* file paths
* source code structure
* frameworks
* libraries
* database info
* usernames
* OS details

---

# Example

```txt
C:\xampp\htdocs\admin\login.php
```

Now attacker knows:

* Windows server
* XAMPP
* PHP structure

---

# Another Example

```txt
SQLSTATE[42000]
```

Now attacker knows:

* SQL database exists
* backend query failed

---

# Real Security Risk

Detailed errors help attackers:

* craft exploits
* find injection points
* fingerprint software

---

# Best Practice

Production servers should:

* hide detailed errors
* show generic messages

Good:

```txt
Something went wrong.
```

Bad:

```txt
MySQL syntax error near SELECT...
```

---

# 4. Username Enumeration

Task:

> Username Enumeration using error messages

VERY common vulnerability.

---

# What is Username Enumeration?

Application reveals:
whether username exists or not.

---

# Example

Login form:

Case 1:

```txt
Invalid password
```

Case 2:

```txt
Username does not exist
```

Attacker now knows:

* valid usernames

---

# Why Dangerous?

Attackers can:

* collect real usernames
* perform password attacks
* target employees

---

# Better Secure Message

Instead of:

```txt
Wrong password
```

and:

```txt
User not found
```

Use:

```txt
Invalid credentials
```

for both.

---

# Places Enumeration Happens

* Login forms
* Forgot password
* Registration
* API responses

---

# Example

Forgot password:

```txt
Email does not exist
```

Now attacker learns valid accounts.

---

# 5. IIS and Apache Log Analysis

Task:

> Analyze logs and identify attacks

This is one of the MOST valuable real-world skills.

---

# What Are Logs?

Servers record activities.

Like CCTV for applications.

Logs contain:

* IP addresses
* requests
* errors
* attacks
* timestamps

---

# Apache Logs

Apache stores:

* access logs
* error logs

---

# IIS Logs

Microsoft IIS web server logs.

---

# Access Log Example

```txt
192.168.1.5 - - [25/May/2026] "GET /admin HTTP/1.1" 404
```

Meaning:

* IP accessed `/admin`
* page not found

---

# Error Log Example

```txt
PHP Warning: include(config.php)
```

---

# Why Logs Matter in Security

Blue team uses logs to:

* detect attacks
* investigate incidents
* trace attackers

Attackers study logs too sometimes.

---

# Things You Learn to Detect

---

# SQL Injection Attempts

Example:

```txt
?id=1' OR '1'='1
```

---

# Directory Traversal

Example:

```txt
../../etc/passwd
```

---

# Brute Force

Many login attempts quickly.

---

# Scanning

Requests to:

```txt
/admin
/phpmyadmin
/wp-admin
```

---

# XSS Attempts

Example:

```txt
<script>alert(1)</script>
```

---

# LFI/RFI

Example:

```txt
?page=../../etc/passwd
```

---

# Important Concepts Hidden in This Module

---

# A. Error-Based Recon

Attackers intentionally trigger errors to:

* learn technologies
* discover backend behavior

---

# B. Fingerprinting

Using:

* headers
* errors
* stack traces

to identify:

* framework
* server
* language

---

# C. Security Misconfiguration

Most issues here are:

* verbose errors
* debug mode enabled
* poor logging hygiene

---

# D. Detection Engineering

Logs are foundation of:

* SOC
* SIEM
* threat hunting

---

# E. Blue Team Thinking

This module starts teaching:

* defender perspective

Not only attacking.

---

# Real-World Examples

---

# Example 1 — PHP Error Leak

```txt
Warning: mysql_query()
```

Now attacker knows:

* MySQL
* PHP backend

---

# Example 2 — Java Stack Trace

```txt
Struts2 Exception
```

Now attacker searches:

```txt
Struts2 RCE CVE
```

---

# Example 3 — ASP.NET Error

```txt
ViewState MAC failed
```

Now attacker knows:

* ASP.NET
* possible misconfigurations

---

# Most Important Lesson

Applications reveal the most information:

> when they break.

That’s the key lesson of this entire module.

---

# Skills You Should Learn After This

---

# Beginner Level

Learn:

* HTTP status codes
* headers
* cookies
* browser devtools

---

# Intermediate

Learn:

* Burp Suite
* error analysis
* log reading
* stack traces

---

# Advanced

Learn:

* SIEM
* Splunk
* ELK Stack
* threat hunting
* detection engineering
