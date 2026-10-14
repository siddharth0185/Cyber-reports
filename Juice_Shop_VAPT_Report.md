# OWASP Juice Shop: Web Application Security Assessment

**Author:** Siddharth B
**Target:** OWASP Juice Shop (local, self-hosted lab instance, `http://127.0.0.1:3000`)
**Type:** Manual black-box testing, challenge reproduction
**Tools:** Burp Suite Community Edition, browser
**Testing period:** October 2026

> OWASP Juice Shop is an intentionally vulnerable training application. The findings below are **reproductions of known Juice Shop training challenges on a local instance**, not newly discovered vulnerabilities in a production system. They are documented here to demonstrate manual testing methodology, evidence capture and reporting practice.

---

## 1. Executive Summary

This report documents six reproduced vulnerabilities in OWASP Juice Shop, identified through manual black-box testing with Burp Suite and browser DevTools. The findings span five vulnerability classes — SQL injection, DOM-based cross-site scripting, broken access control (IDOR), improper privilege assignment (mass assignment), and client-side trust in a hidden form field — plus one instance of weak default credentials. Two findings (F-01 and F-04) independently lead to full administrator access, showing the application exposes more than one unrelated path to the same high-impact outcome. F-06 required inspecting client-side DOM state rather than tampering with a URL or JSON parameter directly, reflecting a different discovery approach than the other access-control findings. Severity ratings reflect lab impact; real-world impact would depend on deployment context and is noted separately where relevant.

| Severity (lab context) | Count |
|-------------------------|-------|
| Critical | 2 |
| High     | 2 |
| Medium   | 2 |
| Low      | 0 |

## 2. Scope and Methodology

- **In scope:** login/authentication, product search, shopping basket, user registration, feedback submission — all on a local Juice Shop instance.
- **Out of scope:** denial of service, attacks against the host machine, any non-local instance.
- **Approach:** mapped application requests via Burp Proxy/HTTP history, identified user-controlled input points, and tested each for injection, output handling, object-level authorization and server-side trust issues. Each finding was confirmed via the application's built-in challenge-solved notification and, where applicable, Burp Repeater or browser DevTools.
- **Limitation:** some raw requests/responses were not fully preserved during testing. Where that applies, it is stated explicitly in the finding rather than reconstructed.

## 3. Findings Summary

| ID | Title | Class | Severity (lab) | Endpoint |
|----|-------|-------|-----------------|----------|
| F-01 | Authentication Bypass via SQL Injection | SQL Injection (CWE-89) | Critical | `POST /rest/user/login` |
| F-02 | DOM-Based XSS via Product Search | Cross-Site Scripting (CWE-79) | High | Product search UI |
| F-03 | Broken Access Control on Basket Retrieval (IDOR) | Broken Access Control (CWE-639) | High | Basket retrieval endpoint |
| F-04 | Administrator Account Protected by Weak Default Credentials | Weak Credentials (CWE-521) | Critical | `POST /rest/user/login` |
| F-05 | Privilege Escalation via Mass Assignment on Registration | Improper Privilege Assignment (CWE-915) | Medium | `POST /api/Users/` |
| F-06 | Feedback Forgery via Hidden User-ID Field | Broken Access Control / Client-Side Trust (CWE-639) | Medium | `POST /api/Feedbacks/` |

---

## 4. Detailed Findings

### F-01: Authentication Bypass via SQL Injection

| Field | Detail |
|-------|--------|
| **Severity (lab)** | Critical |
| **Class** | CWE-89: SQL Injection |
| **Affected endpoint** | `POST /rest/user/login` |
| **Parameter** | `email` (JSON body) |

**What was reproduced**
A SQL injection payload in the login `email` field bypassed authentication on the local instance; the application's "Login Admin" challenge-solved notification appeared.

**Steps to reproduce**
1. Intercept the login request via Burp.
2. Submit the body shown below.
3. Observe successful authentication and the challenge-solved banner.

**Request (as recorded in testing notes):**
```http
POST /rest/user/login HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{"email":"' OR 1=1--","password":"test123"}
```
This body is reproduced from testing notes; the exact raw request should be pulled from Burp history if an unmodified transcript is needed. No unobserved response details (headers, exact body) are claimed here.

**Root cause (behavior-level, not source-verified)**
Observed behavior is consistent with user input being incorporated into a backend SQL query without parameterization, allowing the input to alter query logic. The exact query or source code was not inspected, so this is the likely explanation rather than a confirmed code-level cause.

**Impact**
*In lab:* authentication was bypassed entirely, logging in as an arbitrary account without a valid password.
*Real-world framing:* in a production system with the same pattern, this class of flaw typically enables full account takeover and potentially broader database access, depending on query context.

**Remediation**
Use parameterized queries / prepared statements for all database access built from user input. Avoid constructing SQL via string concatenation. Add negative-case authentication tests.

**Evidence**
Request body above; challenge-solved banner screenshot. Response was not fully preserved — retrieve from Burp history for an exact transcript if required for submission.

---

### F-02: DOM-Based XSS via Product Search

| Field | Detail |
|-------|--------|
| **Severity (lab)** | High |
| **Class** | CWE-79: Cross-Site Scripting (DOM-based) |
| **Affected endpoint** | Product search UI (local instance); exact route/URL not preserved |
| **Parameter** | Search query input |

**What was reproduced**
A payload entered into the product search field caused a JavaScript `alert()` to execute, and the "DOM XSS" challenge-solved notification appeared.

**Payload used:**
```html
<iframe src="javascript:alert('xss')">
```

**Root cause (behavior-level, not source-verified)**
Observed behavior is consistent with untrusted search input reaching a DOM sink that permits executable markup, without sanitization or encoding. The exact sink or source line was not verified.

**Impact**
*In lab:* arbitrary JavaScript executed in the page context via a crafted search input.
*Real-world framing:* this class of flaw typically enables session/token theft or forced actions against a victim who opens a crafted link, depending on what the executing script does.

**Remediation**
Avoid unsafe DOM sinks; prefer safe text APIs (e.g. `textContent`) where HTML rendering isn't required. Where HTML must be rendered, use a vetted sanitizer. Apply a restrictive Content-Security-Policy as defense in depth.

**Evidence**
Challenge-solved banner and alert dialog observed directly. Exact request/URL was not preserved — retrieve from browser/Burp history if an exact transcript is required.

**Note:** a separate attempted reflected-XSS exploration via a tracking route was not confirmed (no script execution or solved banner observed) and is excluded from this report.

---

### F-03: Broken Access Control on Basket Retrieval (IDOR)

| Field | Detail |
|-------|--------|
| **Severity (lab)** | High |
| **Class** | CWE-639: Authorization Bypass Through User-Controlled Key |
| **Affected endpoint** | Basket retrieval endpoint (exact method/path not preserved — see note) |
| **Parameter** | Basket ID (user-controlled) |

**What was reproduced**
Two local test accounts were used. Account A's own basket contained Apple Juice. Account B's basket was separately observed to contain Apple Pomace, with basket ID `7`. Account A's authenticated request was then modified to target basket ID `7` instead of its own basket ID. The server returned a basket object matching Account B's known contents (Apple Pomace) under Account A's own session, and the "View Basket" challenge was solved.

**Observed response fragment:**
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"id":7,"coupon":null,"UserId":27,...,"Products":[{"id":24,"name":"Apple Pomace"...}]}}
```

The full request method/path and complete response were not preserved in testing notes. The verified behavior is: changing the basket identifier to `7` while retaining Account A's own session returned basket contents matching Account B's (Apple Pomace). The response's `UserId: 27` was not independently cross-checked against Account B's actual user ID during testing — the cross-account access is confirmed by the returned basket contents and ID matching Account B's known basket, not by a separately verified user-ID match.

**Root cause (behavior-level, not source-verified)**
The backend returned basket data keyed on a user-controlled object identifier without verifying that the authenticated session owns that basket. Source code was not inspected.

**Impact**
*In lab:* one authenticated test account retrieved another test account's basket contents, confirmed by matching product data (Apple Pomace, basket ID 7). No broader data exposure beyond this was tested or is claimed.
*Real-world framing:* this class of flaw (IDOR) typically allows systematic enumeration of other users' data at scale.

**Remediation**
Enforce server-side object-level authorization on every basket access: derive the basket to return from the authenticated principal, and verify ownership before returning data for any requested ID. Add automated cross-account authorization tests.

**Evidence**
Response fragment above; challenge-solved banner. The exact request is not reproduced here — retrieve the original from Burp Repeater/HTTP history before using this as a verbatim transcript. Authorization headers/session tokens are not included in this report.

---

### F-04: Administrator Account Protected by Weak Default Credentials

| Field | Detail |
|-------|--------|
| **Severity (lab)** | Critical |
| **Class** | CWE-521: Weak Password Requirements |
| **Affected endpoint** | `POST /rest/user/login` |
| **Parameter** | `email`, `password` |

**What was reproduced**
Login succeeded using the credentials `admin@juice-sh.op` / `admin123` on the local instance, with the "Password Strength" challenge-solved notification appearing. No injection or other exploitation technique was required.

**Root cause**
The training application ships with a weak, guessable administrator password that was never required to change. This is a known lab credential used to demonstrate the risk class; it is not a newly discovered production credential, and no password-storage implementation detail was examined.

**Impact**
*In lab:* full administrator access obtained via normal credential submission.
*Real-world framing:* this finding is significant independent of F-01 — even if the SQL injection in F-01 were fixed, this account would remain compromised through credential guessing alone, demonstrating it as a distinct root cause rather than a side effect.

**Remediation**
Enforce a strong password policy at account creation, reject common/default passwords, require privileged accounts to set a unique password before first use, apply rate-limiting/lockout on repeated failed logins, and support MFA for admin accounts.

**Evidence**
Challenge-solved banner observed directly. Raw login request/response was not preserved — retrieve from Burp history if an exact transcript is needed.

---

### F-05: Privilege Escalation via Mass Assignment on Registration

| Field | Detail |
|-------|--------|
| **Severity (lab)** | Medium |
| **Class** | CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes (Mass Assignment) |
| **Affected endpoint** | `POST /api/Users/` |
| **Parameter** | `role` (JSON body; not exposed in the normal registration form) |

**What was reproduced**
A baseline registration request (no `role` field) created a normal customer account. A second, separate registration — using a different test email — added `"role":"admin"` to the JSON body, and the server created that account directly with administrator privileges. The "Admin Registration" challenge-solved notification appeared. These were two independent registrations (different accounts), not a before/after modification of one account.

**Baseline registration (customer account created):**
```http
POST /api/Users/ HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{
  "email": "orange22@gmail.com",
  "password": "orange",
  "passwordRepeat": "orange",
  "securityQuestion": {
    "id": 7,
    "question": "Name of your favorite pet?"
  },
  "securityAnswer": "[redacted]"
}
```
Observed response:
```http
HTTP/1.1 201 Created
Location: /api/Users/30
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"username":"","role":"customer","id":30,"email":"orange22@gmail.com",...}}
```

**Modified registration (admin account created):**
```http
POST /api/Users/ HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{
  "email": "orange222@gmail.com",
  "password": "orange",
  "passwordRepeat": "orange",
  "securityQuestion": {
    "id": 7,
    "question": "Name of your favorite pet?"
  },
  "securityAnswer": "[redacted]",
  "role": "admin"
}
```
Observed response:
```http
HTTP/1.1 201 Created
Location: /api/Users/31
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"username":"","profileImage":"/assets/public/images/uploads/defaultAdmin.png","isActive":true,"id":31,"email":"orange222@gmail.com","role":"admin",...}}
```

**Root cause (behavior-level)**
The registration endpoint accepts and persists a client-supplied `role` field that is not exposed in the normal registration form and is not intended to be user-controllable. Source code was not inspected; this describes observed behavior only.

**Impact**
*In lab:* any unauthenticated visitor can self-register directly as an administrator, with no privilege check — a repeatable privilege-escalation path at account creation, verified by the `role: admin` field in the 201 response.
*Real-world framing:* mass-assignment flaws of this kind are a well-known class (CWE-915) and typically allow trivial privilege escalation wherever present.

**Remediation**
Explicitly allowlist the fields accepted by the registration endpoint (email, password, security question/answer) and assign `role` server-side with a fixed least-privilege default. Never bind request bodies directly to database/model objects without an allowlist. Restrict role changes to authorized administrative operations, and add mass-assignment tests to the test suite.

**Evidence**
Both request/response pairs above (sanitized — security answers redacted); challenge-solved banner: "You successfully solved a challenge: Admin Registration (Register as a user with administrator privileges.)"

---

### F-06: Feedback Forgery via Hidden User-ID Field

| Field | Detail |
|-------|--------|
| **Severity (lab)** | Medium |
| **Class** | CWE-639: Authorization Bypass Through User-Controlled Key (client-side trust variant) |
| **Affected endpoint** | `POST /api/Feedbacks/` |
| **Parameter** | `UserId` (hidden form field, not exposed in the visible UI) |

**What was reproduced**
The Customer Feedback form contains a hidden `userId` input field that is not shown in the rendered UI but exists in the DOM. The authenticated session belonged to user ID `25`. Using browser DevTools (Elements panel) to locate and edit this hidden field before submitting the form, the value was set to `30` — a different user's ID. The server created the feedback record with `UserId: 30`, exactly matching the forged value rather than the actual authenticated user (`25`). The "Forged Feedback" challenge-solved notification appeared, confirming feedback was posted in another user's name.

**Steps to reproduce**
1. Log in as the authenticated test account (user ID `25`) and open the Customer Feedback page.
2. Open browser DevTools → Elements and locate the hidden `<input id="userId">` field inside the feedback form (not visible in the rendered form).
3. Edit the field's value directly in the DOM from the authenticated user's own ID to a different user's ID (`30`).
4. Fill in the remaining required fields (comment, rating, CAPTCHA) and submit.
5. Observe the response confirms the feedback was created with `UserId: 30`, not `25`.

**Evidence**

Response:
```http
HTTP/1.0 201 Created
Content-Type: application/json; charset=utf-8

{"status":"success","data":{"id":10,"UserId":30,"comment":"lab test validation  (***@gmail.com)","rating":3,"createdAt":"2026-10-14T19:32:59.013Z"}}
```

*What to look for:* the response's `UserId` is `30`, exactly matching the forged value set via DevTools, while the authenticated session's actual user ID was `25` — a direct, confirmed mismatch showing the server used the client-supplied field rather than deriving the author from the session. The application's own challenge-solved notification ("Post some feedback in another user's name") independently confirms the forgery succeeded. The request body itself was not fully captured in testing notes — retrieve the exact POST body from Burp history for a verbatim transcript.

**Root cause (behavior-level, not source-verified)**
The feedback submission endpoint appears to accept a client-supplied `UserId` value rather than deriving the author from the authenticated session server-side. This is architecturally similar to F-05 (client-supplied privilege/identity field trusted by the server) but discovered via DOM inspection of a hidden field rather than by modifying a visible JSON body, indicating the same root-cause pattern recurs across more than one endpoint in the application.

**Impact**
*In lab:* feedback was posted and attributed to an arbitrary user ID chosen by the attacker, without that user's knowledge or consent.
*Real-world framing:* this class of flaw typically enables impersonation, reputational harm, or injecting attributed content (reviews, comments, complaints) in another person's name — a trust and integrity issue rather than a data-confidentiality one.

**Remediation**
Never trust a client-supplied identity/ownership field for any write operation. Derive the authoring user strictly from the authenticated session server-side, and reject or ignore any `UserId`/`userId` field present in the request body for this endpoint. Audit other write endpoints for the same hidden-field-trust pattern, since it recurred independently of F-05.

**References**
- OWASP Top 10 2021: A01 — Broken Access Control
- CWE-639: Authorization Bypass Through User-Controlled Key

---

## 5. Conclusion

All six findings trace to the same underlying pattern: the application trusts client-supplied input — a login field, a search query, an object identifier, a registration payload field, or a hidden form field — without validating it against what the server actually intends to allow. This repeats across the authentication layer (F-01, F-04), the rendering layer (F-02), the authorization layer (F-03, F-06), and the data-binding layer (F-05).

Two independent findings (F-01, F-04) each lead to full administrator compromise on their own, meaning remediating only one would leave the account fully exploitable through the other. F-05 and F-06 further show that the same "trust the client" root cause recurs across two unrelated endpoints (registration and feedback), suggesting it is a systemic pattern in how the application handles identity/ownership fields rather than an isolated bug.

**Remediation priority:**
1. **Immediate:** parameterize all database queries (F-01); enforce a strong password policy on privileged accounts (F-04) — both are complete admin-takeover paths.
2. **High priority:** add an explicit field allowlist to the registration endpoint (F-05) and strip client-supplied identity fields from the feedback endpoint (F-06); enforce server-side ownership checks on object-retrieval endpoints (F-03).
3. **Standard priority:** encode all user-controlled output before DOM insertion and apply a restrictive CSP (F-02).

## Notes on scope and honesty of evidence

- All findings are reproductions of known OWASP Juice Shop training challenges on a local, self-hosted instance — not independently discovered vulnerabilities in a production system.
- Severity is rated for lab context; real-world impact is noted separately per finding and would depend on actual deployment, data sensitivity and surrounding controls.
- Where a raw request or response was not fully preserved during testing, this report says so explicitly rather than reconstructing it, and recommends pulling the exact transcript from Burp history before using this document as a verbatim technical record.
- All Authorization headers, session cookies, JWTs and real personal information have been redacted or omitted from this report.
