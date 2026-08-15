# PortSwigger Web Security Academy: API Security Lab Series

## Overview
This repository contains detailed technical write-ups, vulnerability analyses, and proof-of-concept (PoC) walkthroughs for the **API Security** lab suite on PortSwigger Web Security Academy.

The series covers real-world API vulnerabilities ranging from hidden endpoint discovery and undocumented route exposure to parameter manipulation and mass assignment flaws.

---

## Lab Modules Summary

| Module | Lab Name | Vulnerability Type | Primary Exploitation Method | Key Finding |
| :--- | :--- | :--- | :--- | :--- |
| **Lab 1** | Exploiting an API endpoint using documentation | Information Disclosure / Exposed Endpoints | API Documentation Reconnaissance (`/api`, `/swagger.json`, OpenAPI) | Identified undocumented DELETE endpoints & administrative API keys to perform unauthorized actions. |
| **Lab 2** | Finding and exploiting an unused API endpoint | Broken Access Control / Method Misconfiguration | HTTP Method Probing (`PATCH`) & Header Fuzzing (`Content-Type: application/json`) | Bypassed front-end limitations via an unlinked backend `PATCH` route to modify item pricing to `$0.00`. |
| **Lab 3** | Exploiting a mass assignment vulnerability | Mass Assignment / Auto-Binding Logic | JSON Payload Injection (`chosen_discount`) | Exploited object auto-binding in backend frameworks to inject unhandled discount fields (`100%`) into `POST` request bodies. |

---

## Key Takeaways & Methodology

Across these modules, a structured API security methodology was applied:

1. **Reconnaissance & Traffic Analysis:**
   - Intercepting client-server communication using Burp Suite to map full REST API structures.
   - Analyzing GET request response payloads for over-exposed data fields and hidden administrative schemas.

2. **Endpoint & HTTP Verb Fuzzing:**
   - Testing alternative HTTP methods (`OPTIONS`, `PATCH`, `PUT`, `DELETE`) on standard endpoints.
   - Identifying hidden/legacy API paths, interactive documentation pages (Swagger/OpenAPI), and unlinked internal routes.

3. **Logic Flaw & Parameter Injection:**
   - Manipulating data types, payload schemas, and missing headers (`Content-Type`).
   - Leveraging Mass Assignment (auto-binding) to overwrite sensitive object properties not intended for end-user modification.

---

## Core Remediation Strategies

To secure modern web APIs against these vulnerability classes, the following defensive controls should be enforced:

- **Disable / Protect Administrative Documentation:** Ensure API documentation endpoints (Swagger, OpenAPI, GraphiQL) are restricted or disabled in production environments.
- **Implement Strict Method Restrictions & RBAC:** Restrict HTTP verbs on sensitive routes and enforce robust Role-Based Access Control on every API endpoint.
- **Enforce Input Filtering via DTOs (Data Transfer Objects):** Prevent Mass Assignment by explicitly binding input parameters to dedicated allowlisted schemas, ignoring unmapped or sensitive fields.
- **Server-Side Integrity & Validation:** Never trust pricing, discounts, user privileges, or state attributes supplied in client-side request bodies. Always validate and enforce business rules server-side.

---

## Repository Structure

```text
├── Lab-1-END-Point EXploitation/
│   └──IMAGES-1            # Screenshots
│   └── README.md          # Exposed Documentation Walkthrough
├── Lab-2-Unused-API-Endpoint/
│  └──IMAGES-2           #Screenshots
│  └── README.md         # Unused PATCH Endpoint Exploitation
├── Lab-3-Mass-Assignment/
│  └──IMAGES-3           #Screenshots
│  └── README.md          # Mass Assignment Injection Write-up
└── README.md              # Global Series Overview (This File)
```
