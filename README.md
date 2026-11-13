# Security Requirements Document (SRD) & Security Response Plan (SRP)
## Case Study: Product Tampering & Broken Access Control Remediation

> **DISCLAIMER & ETHICAL STATEMENT**  
> This project is a security analysis study, risk governance framework mapping, and security requirements engineering exercise (GRC Documentation) using **OWASP Juice Shop** (a locally hosted lab environment intentionally deployed on Ubuntu Linux for educational purposes). This document does not represent security testing conducted on unauthorized or commercial enterprise systems.

---

## 1. Executive Summary

This document outlines the technical security evaluation, risk assessment, **Security Requirements Document (SRD)**, and **Security Response Plan (SRP)** addressing an identified **Product Tampering (Broken Access Control / Client-Side Parameter Manipulation)** vulnerability within the OWASP Juice Shop application.

The primary objective of this documentation is to analyze the mechanics of manipulating frontend component parameters (such as altering the `href` hyperlink target within the *OWASP SSL Advanced Forensic Tool (O-Saft)* product description to an arbitrary destination like `https://owasp.slack.com` or others), establish server-side authorization controls and input validation requirements, and map the risk lifecycle to **ISO/IEC 27001:2022** and **NIST CSF v2.0** enterprise frameworks.

---

## 2. Risk & Impact Analysis & Risk Treatment Plan

| Parameter | Initial Assessment (Pre-Mitigation) | Post-Mitigation (Residual Risk) |
| :--- | :--- | :--- |
| **Vulnerability** | Client-Side Parameter Manipulation / Broken Access Control **(Product Tampering)** (CWE-472 / CWE-284) | Remediated / Neutralized |
| **CVSS v3.1 Score** | **5.3 (MEDIUM)** (`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N`) | **0.0 (NONE)** |
| **Risk Treatment Strategy** | N/A | **Risk Reduction (Mitigation)** |
| **Risk Owner** | Lead Backend Engineer / Lead DevSecOps | Lead Frontend Engineer / UI-UX Developer |
| **Remediation SLA** | **7 Business Days** (Medium Severity Standard) | Verified & Closed |

### 2.1 Technical & Business Impact
* **UI Defacement & Malicious Redirect / Phishing Risk**: Exploiting client-side parameter manipulation allows threat actors to alter product metadata and inject unauthorized external hyperlinks (such as redirecting the OWASP SSL Advanced Forensic Tool description to malicious domains), exposing end-users to phishing campaigns, credential theft, and trust exploitation under the guise of an authentic application interface.
* **Absence of Server-Side Validation**: The application architecture relies entirely on frontend trust without enforcing rigorous authorization checks, input sanitization, or access control validation on the backend server, permitting unauthorized data tampering and compromising overall data integrity.

---

## 3. Compliance Framework Mapping

| Standard Framework | Control ID & Name | Strategic Alignment Description |
| :--- | :--- | :--- |
| **ISO/IEC 27001:2022** | **Annex A.8.26** (Application Security Requirements) | Establishes formal security requirements for application development and runtime environments to enforce strict server-side input validation and prevent client-side parameter manipulation. |
| **ISO/IEC 27001:2022** | **Annex A.8.28** (Secure Coding) | Mandates secure coding practices to ensure component metadata and hyperlink rendering logic cannot be arbitrarily overridden via frontend manipulation or unverified parameters. |
| **ISO/IEC 27001:2022** | **Annex A.8.3** (Information Access Restriction ) | Enforces strict role-based access control (RBAC) and least privilege principles to restrict unauthorized modifications of product components and interface metadata. |
| **NIST CSF v2.0** | **PR.AA-05** (Access Control & Permissions) | Enforces protective authorization controls and server-side verification mechanisms to safeguard product metadata, application endpoints, and prevent unauthorized client-side tampering. |
| **NIST CSF v2.0** | **PR.PS-06** (Secure Software Development) | Requires continuous application security testing, code reviews, and runtime validation safeguards to ensure component integrity and prevent client-side security bypasses. |

---

## 4. Proof of Concept (PoC) & Execution Evidence

1. **Vulnerability Mechanics**:  
   Product catalog interfaces render mutable DOM attributes and client-side form parameters directly within the user's browser environment. While frontend UI logic restricts unauthorized modifications (such as hiding administrative edit controls or enforcing read-only attributes), the backend API fails to validate user authorization and parameter integrity, allowing attackers to tamper with product properties (e.g., prices, descriptions, or href links) via **Browser DevTools** or an intercepting proxy.

2. **Exploitation Steps (Ubuntu Linux Environment)**:
   * **Step 1 (Navigate and Locate Target Component)**: Access the public product catalog within OWASP Juice Shop and navigate to the targeted product item (e.g., OWASP SSL Advanced Forensic Tool (O-Saft)).
   * **Step 2 (Inspect DOM Elements)**: Open the browser's Developer Tools (DevTools), utilize the element inspector tool, and locate the mutable hyperlink attribute (href) or product parameter within the HTML DOM structure.
   * **Step 3 (Inject Modified Parameters)**: Intercept or directly manipulate the frontend DOM attributes within the browser console or via an intercepting proxy (e.g., modifying the href target from the legitimate link to an unauthorized external destination like [https://owasp.slack.com](https://owasp.slack.com) or any other links).
   * **Step 4 (Validate State Persistence / Trigger Action)**: Submit or observe the application rendering the modified interface component without triggering a server-side authorization error or validation block.

 3. **Execution Evidence (Visual Proofs)**:

   * **DOM Element Inspection**:  
     Visual proof demonstrating the live browser DevTools inspector highlighting the mutable href attribute within the O-Saft product description container.  
     ![Baseline State](./forgotten_backup/curl_as_path.png)

   * **Parameter Modification via Console/Proxy**:  
     Evidence of the modified hyperlink parameter successfully injected into the active client-side session.  
     ![Baseline State](./forgotten_backup/file.png)

   * **Successful Malicious Redirect Verification**:  
     Visual confirmation showing the browser successfully resolving and redirecting to the arbitrary external URL ([https://owasp.slack.com](https://owasp.slack.com)) upon clicking the tampered product link. 
     ![Baseline State](./forgotten_backup/dependencies.png)

---

## 5. Security Requirements Document (SRD) & Technical Remediation

### 5.1. Security Requirements (Functional & Architectural Rules)
To remediate **Product Tampering (CWE-472 / CWE-284)**, the system must transition from client-side UI enforcement to strict server-side verification:

* **SR-01 (Server-Side Authorization Enforcement - CWE-284)**: Backend APIs must never rely on hidden UI elements or `readonly` / `disabled` DOM attributes for security. Every state-changing endpoint (`POST`, `PUT`, `PATCH`, `DELETE`) must verify the authenticated user’s session and Role-Based Access Control (RBAC) privileges before modifying product records.
* **SR-02 (Server-Side Parameter Integrity - CWE-472)**: The application must treat all client-submitted product parameters as untrusted. Critical business attributes (such as `price` or `product_id`) must be validated against server-side business rules and authoritative database records rather than trusted from frontend payloads.
* **SR-03 (Strict HTML & href Attribute Sanitization)**: Product description fields must undergo server-side HTML sanitization to strip unauthorized tags or restrict `href` attributes exclusively to trusted, allowlisted `https://` domains, preventing UI defacement and malicious phishing redirects.
* **SR-04 (Explicit Field Allowlisting)**: API endpoints must explicitly extract and update only permitted fields from incoming JSON payloads to prevent unauthorized property modification via DevTools or proxy interception.

### 5.2. Backend API Authorization & Sanitization (Node.js / Express.js)
Harden product update endpoints by enforcing RBAC middleware, strict schema validation, and server-side HTML sanitization before writing to the database:

```javascript
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');
const DOMPurify = createDOMPurify(new JSDOM('').window);

// Enforce authentication and admin role verification on the server side
app.put('/api/Products/:id', authenticateUser, requireAdminRole, async (req, res, next) => {
    try {
        const { name, price, description } = req.body;

        // 1. Validate data types and prevent negative/tampered pricing
        if (typeof name !== 'string' || typeof price !== 'number' || price <= 0) {
            return res.status(400).json({ error: 'Invalid product attributes' });
        }

        // 2. Sanitize description to strip malicious href links and prevent UI defacement
        const cleanDescription = DOMPurify.sanitize(description, {
            ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p', 'br'], // Disallow <a> tags or enforce domain allowlist
            ALLOWED_ATTR: []
        });

        // 3. Explicitly whitelist updatable columns
        await ProductModel.update(
            { name: name.trim(), price, description: cleanDescription },
            { where: { id: req.params.id } }
        );

        return res.status(200).json({ status: 'Product updated securely' });
    } catch (err) {
        next(err);
    }
});
```

### 5.3. Defense-in-Depth & DevSecOps Controls
* **Output Encoding on Frontend**: Ensure the frontend framework escapes untrusted text by default and avoids unsafe DOM sinks (such as innerHTML or Angular's `innerHTML`) without strict sanitization.
* **API Gateway / WAF Policy**: Deploy Web Application Firewall (WAF) rules to detect and block requests containing suspicious HTML injection payloads (`<a href=...>`, `<script>`) or unexpected JSON parameter mutations.
* **Automated Regression Testing**: Add integration tests in the CI/CD pipeline to verify that unprivileged user tokens receive `HTTP 403` Forbidden when attempting to invoke product modification endpoints.

### 5.4. Verification & Acceptance Criteria
* **Authorization Bypass Test**: Modifying DOM elements via Browser DevTools to enable hidden edit buttons and submitting a `PUT /api/Products/:id` request as a non-admin user must return `HTTP 403 Forbidden`.
* **Price Tampering Test**: Intercepting the HTTP request via a proxy (e.g., Burp Suite) and altering `price` to `0.01` must be rejected by the server (`HTTP 400 Bad Request`) or overridden by the authoritative database price.
* **Link / UI Defacement Test**: Injecting an external href link (e.g., `<a href="http://malicious-domain.com">0-Saft</a>`) into the product description must result in the href attribute being stripped prior to database storage.

---

## 6. Security Response Plan (SRP) - Operational Incident Handling Procedures

### 1. Detection & Triage (Identification)
Identify active exploitation attempts and confirm whether product catalog integrity or transaction pricing has been compromised:

* **Log & SIEM Alerting**: Monitor web server, API gateway, and WAF logs for unauthorized `PUT`, `PATCH`, or `POST` requests targeting product management endpoints (`/api/Products/*`) originating from non-administrative user sessions or IP addresses.
* **Payload & Anomaly Inspection**: Flag HTTP request bodies containing unexpected HTML tags (e.g., `<a href=...>`, `<script>`, `<iframe>`) in product description fields, or abnormal numerical values (e.g., `price <= 0`, massive discounts) in checkout payloads (`/api/BasketItems`).
* **Database Integrity Audits**: Run automated checksum comparisons or database audit queries to detect unauthorized alterations to product prices, titles, or external URLs.
* **Severity Classification**:
  * **Critical / High**: Active price manipulation resulting in fraudulent orders, or injection of malicious `href` phishing links on high-traffic public product pages.
  * **Medium**: Attempted parameter tampering blocked by downstream business logic or isolated to non-public test records[: 16].

### 2. Containment (Immediate Damage Control)
Stop ongoing tampering and prevent end-users from interacting with defaced or malicious product content:

* **Short-Term Containment (Emergency Mitigation)**:
  * Deploy an emergency **Web Application Firewall (WAF)** virtual patch to block all `PUT` / `PATCH` / `DELETE` requests to `/api/Products/*` except from whitelisted internal administrative IP ranges.
  * Apply WAF payload inspection rules to reject requests containing unapproved `href` attributes or raw HTML tags in product update requests.
  * Temporarily freeze automated order fulfillment for transactions exhibiting price discrepancies between the order total and the authoritative catalog price.
* **Account & Session Revocation**: Immediately invalidate active session tokens (`JWT` / session cookies) and suspend user accounts identified as submitting tampered API requests.
* **Long-Term Containment**: Deploy hotfix middleware enforcing strict Role-Based Access Control (`requireAdminRole`) and server-side HTML sanitization (`DOMPurify`) on all product endpoints.

### 3. Eradication (Root Cause & Artifact Removal)
Remove all tampered data from the environment and close the underlying access control and validation gaps:

* **Catalog Data Restoration**: Query database transaction logs to identify all product records modified by compromised or malicious user IDs, and restore original `price`, `description`, and `href` attributes from verified pre-incident backups.
* **Order & Financial Reconciliation**: Audit recent orders placed during the incident window to identify purchases executed with tampered prices; flag, cancel, or refund fraudulent transactions.
* **Code-Level Vulnerability Patching**: Permanently remove reliance on frontend `readonly` / `disabled` UI restrictions by deploying backend RBAC authorization checks (`CWE-284`), schema validation, and server-side price recalculation (`CWE-472`).

### 4. Recovery (Safe Service Restoration)
Validate system integrity before restoring full public and administrative operations:

* **Verification Testing**: Execute automated security regression tests and manual proxy interception tests (e.g., Burp Suite) in staging to confirm that non-admin users receive `HTTP 403 Forbidden` on product modification routes and that malicious `href` tags are stripped.
* **Phased Traffic Restoration**: Lift emergency WAF IP blocks gradually while maintaining elevated SIEM monitoring and real-time alerting on `/api/Products/*` and `/api/BasketItems` endpoints.
* **Cache Invalidation**: Purge CDN and browser caches to ensure end-users no longer receive cached versions of defaced product pages or malicious redirect links.

### 5. Post-Incident Activity & Communication (Lessons Learned)
* **Stakeholder & User Notification**: If malicious `href` links led to external phishing domains, notify affected users to reset credentials and monitor for suspicious activity; coordinate with domain registrars or abuse teams to take down external phishing.
* **Root Cause Analysis (RCA)**: Conduct a formal post-mortem within 5 business days to document why backend authorization and parameter validation were omitted during development.
* **Control Enhancement**: Update Secure SDLC checklists, add mandatory API authorization unit tests to the CI/CD pipeline, and schedule targeted secure coding training for frontend and backend engineering teams.

---

## About the Author

Ex-Product Manager pivoting into Governance, Risk, and Compliance (GRC). This repository demonstrates the practical application of technical vulnerability research mapped directly to enterprise security controls and compliance frameworks (ISO/IEC 27001 & NIST CSF).
