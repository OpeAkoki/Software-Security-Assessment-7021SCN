Task B - Threat Model: NHS Patient Appointment Booking System

System Description

This is a web-based appointment booking system for an NHS GP practice. Patients can register, log in, view available slots, and book or cancel appointments. GPs and practice admins can manage their schedules, view patient records, and send appointment reminders. The system holds patient demographic data, NHS numbers, appointment history, and basic medical notes. It integrates with NHS Login for patient authentication, an SMTP email service for appointment reminders, and a third-party SMS gateway for text notifications.


Data Flow Diagram

```mermaid
flowchart TD
    Patient([Patient])
    Doctor([Doctor / Admin])
    NHSLogin([NHS Login Service])
    EmailSMTP([Email / SMS Gateway])

    WebApp[Web Application]
    AuthService[Authentication Service]
    AppDB[(Appointments Database)]
    PatientDB[(Patient Records Database)]

    Patient -->|Login request| AuthService
    AuthService -->|OAuth token request| NHSLogin
    NHSLogin -->|Token response| AuthService
    AuthService -->|Session token| Patient

    Patient -->|Book / cancel appointment| WebApp
    Doctor -->|Manage schedule, view records| WebApp

    WebApp -->|Read / write appointments| AppDB
    WebApp -->|Read patient data| PatientDB

    WebApp -->|Send reminder| EmailSMTP
    EmailSMTP -->|Email / SMS| Patient
```


STRIDE Table

| STRIDE Category | Specific Threat | Affected Component | Risk | Mitigation | SDLC Phase |
|---|---|---|---|---|---|
| Spoofing | An attacker could spoof a patient's NHS Login session by stealing the session cookie if HttpOnly and Secure flags are not set, allowing them to book or cancel appointments as that patient | Authentication Service | High | Set HttpOnly, Secure, and SameSite=Strict on all session cookies | Design |
| Spoofing | An attacker could impersonate a GP admin by reusing a stolen JWT token if tokens have no expiry, gaining access to all patient records | Authentication Service | High | Set short expiry on JWTs and validate signature on every request | Design |
| Tampering | A patient could tamper with the appointment booking request and change the patient_id parameter to cancel another patient's appointment | Web Application | High | Validate server-side that the authenticated user owns the resource before any modification | Implementation |
| Tampering | An attacker with database access could modify appointment records directly if the database user account has write permissions beyond what the application needs | Appointments Database | Medium | Apply principle of least privilege to the database service account, read-only where possible | Deployment |
| Repudiation | A GP could deny having deleted a patient's appointment record if no audit log captures who made the change and when | Appointments Database | Medium | Log all write operations with user ID and timestamp to an append-only audit table | Implementation |
| Repudiation | A patient could dispute a booking they made if the system does not retain confirmation records with timestamps | Web Application | Low | Store immutable booking confirmation records and send timestamped email receipts | Implementation |
| Information Disclosure | Verbose error messages could expose internal stack traces or database query details to an unauthenticated user if exceptions are not handled | Web Application | High | Catch all exceptions and return generic error messages in production, log detail server-side only | Implementation |
| Information Disclosure | Patient NHS numbers and medical notes could be returned in API responses to a patient who guesses another patient's record ID if object-level authorisation is not enforced | PatientDB | High | Enforce object-level authorisation on every API endpoint, verify record ownership before returning data | Design |
| Denial of Service | An attacker could send thousands of login requests per second to lock out real patients and overwhelm the authentication service | Authentication Service | Medium | Rate-limit login attempts per IP and per account, implement CAPTCHA after a threshold | Design |
| Denial of Service | An attacker could submit thousands of appointment booking requests to fill all available slots, preventing real patients from booking | Web Application | Medium | Rate-limit booking requests per authenticated user and alert on abnormal booking volume | Implementation |
| Elevation of Privilege | A patient could elevate their role to admin by modifying the role claim in a locally stored JWT if the signature is not verified server-side | Authentication Service | High | Always verify JWT signature server-side, store role information in the server session not in the client token | Design |
| Elevation of Privilege | A receptionist account could access GP clinical notes if role-based access control is not enforced at the API layer, only at the UI layer | Web Application | Medium | Enforce RBAC at the API and database layer, not just in the front-end interface | Implementation |


Security Requirements

SR-01: The system SHALL enforce HTTPS on all endpoints using TLS 1.2 or above to prevent network-level information disclosure. Traces to: Information Disclosure row 1.

SR-02: The system SHALL set HttpOnly, Secure, and SameSite=Strict flags on all session cookies to prevent session token theft via XSS or network interception. Traces to: Spoofing row 1.

SR-03: The system SHALL validate the signature and expiry of all JWT tokens on every authenticated request server-side. Traces to: Spoofing row 2, Elevation of Privilege row 1.

SR-04: The system SHALL enforce object-level authorisation on all API endpoints so that a user can only read or modify resources they own. Traces to: Tampering row 1, Information Disclosure row 2.

SR-05: The system SHALL apply the principle of least privilege to all database service accounts, granting only the permissions the application requires. Traces to: Tampering row 2.

SR-06: The system SHALL maintain an append-only audit log recording the user ID, timestamp, and action for all create, update, and delete operations on appointment and patient records. Traces to: Repudiation row 1, Repudiation row 2.

SR-07: The system SHALL suppress detailed error messages and stack traces in production responses, logging full detail server-side only. Traces to: Information Disclosure row 1.

SR-08: The system SHALL rate-limit login attempts to a maximum of 10 attempts per minute per IP address and lock accounts after 5 consecutive failed attempts. Traces to: Denial of Service row 1.

SR-09: The system SHALL rate-limit appointment booking requests to a maximum of 5 bookings per user per hour and alert administrators on abnormal volume. Traces to: Denial of Service row 2.

SR-10: The system SHALL enforce role-based access control at the API and database layer, independent of any front-end interface restrictions. Traces to: Elevation of Privilege row 2.

SR-11: The system SHALL store user roles in the server-side session only and never rely on client-supplied role claims without cryptographic verification. Traces to: Elevation of Privilege row 1.

SR-12: The system SHALL send a timestamped email confirmation to the patient for every booking or cancellation action to support non-repudiation. Traces to: Repudiation row 2.

SR-13: The system SHALL encrypt all patient data at rest using AES-256 or equivalent. Traces to: Information Disclosure row 2.

SR-14: The system SHALL integrate with NHS Login OAuth 2.0 for all patient authentication and SHALL NOT allow patients to set their own passwords. Traces to: Spoofing row 1.


Limitations of STRIDE

STRIDE is a useful starting point for structured threat modelling but it has real weaknesses that are worth understanding. The most obvious one is that it does not account for supply chain threats. In this system we rely on NHS Login, an SMS gateway, and likely several open source libraries. If any of those third parties were compromised, STRIDE as applied here would not catch it because the model treats external services as trusted components within defined trust boundaries. The SolarWinds attack and the Log4Shell vulnerability are examples of exactly the kind of threat STRIDE would miss entirely.

STRIDE also struggles with insider threats. The model assumes that authenticated users are acting within their role. It does not naturally surface scenarios like a receptionist browsing patient records out of curiosity, or a system administrator exporting the patient database. These threats exist within the trust boundaries rather than crossing them, so they do not show up cleanly in a STRIDE analysis. Modelling insider threats requires a different approach, usually combining STRIDE with user behaviour analytics and access reviews.

Another limitation is that STRIDE produces a static document. A threat model written during the design phase reflects the system as it was understood at that point. As the system evolves, new endpoints get added, integrations change, and the attack surface shifts. Without a process to revisit and update the model, it quickly becomes outdated and gives a false sense of security. A threat model that was accurate six months ago may miss the most recently added features entirely.

PASTA (Process for Attack Simulation and Threat Analysis) takes a different approach that addresses some of these gaps. Where STRIDE asks what can go wrong with each component, PASTA starts from the attacker's perspective and works backwards from business impact. It is risk-centred rather than threat-centred, which means it naturally prioritises the threats that would cause the most damage to the organisation rather than producing a flat list where a low-impact spoofing threat sits alongside a catastrophic data breach. For a system holding NHS patient data, the PASTA approach would likely surface the patient record exposure threats much more prominently than STRIDE does.


Trust Boundaries

The main trust boundaries in this system are between the public internet and the web application, between the web application and the internal databases, and between the web application and the external NHS Login and email services. Data crossing these boundaries should be validated and sanitised on the receiving side regardless of where it came from.
