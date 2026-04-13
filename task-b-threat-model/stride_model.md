Task B - Threat Model: NHS Appointment Booking System

The system is a web app for a GP practice. Patients log in, book or cancel appointments. Doctors manage schedules and view records. Admins run the back end. It stores patient details, NHS numbers and appointment history. It connects to NHS Login, an email service and an SMS gateway.


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

| Category | Threat | Component | Risk | Fix | Phase |
|---|---|---|---|---|---|
| Spoofing | Stolen session cookie used to log in as another patient | Auth Service | High | HttpOnly, Secure, SameSite on cookies | Design |
| Spoofing | Expired JWT reused to access records | Auth Service | High | Short expiry, verify signature every request | Design |
| Tampering | Patient changes booking ID in request to cancel someone else's appointment | Web App | High | Check ownership server side before any change | Implementation |
| Tampering | Attacker edits DB records directly because service account has too many permissions | Database | Medium | Least privilege on DB accounts | Deployment |
| Repudiation | GP denies deleting an appointment, no log to check | Database | Medium | Append-only audit log with user ID and timestamp | Implementation |
| Repudiation | Patient disputes a booking, no confirmation record exists | Web App | Low | Store booking records, send email receipt | Implementation |
| Info Disclosure | Error message leaks stack trace to unauthenticated user | Web App | High | Generic errors in prod, log detail server side | Implementation |
| Info Disclosure | Wrong patient's record returned because ownership not checked | PatientDB | High | Check record ownership on every API call | Design |
| Denial of Service | Login endpoint flooded, real patients locked out | Auth Service | Medium | Rate limit per IP, lock after failed attempts | Design |
| Denial of Service | All appointment slots filled with fake bookings | Web App | Medium | Rate limit bookings per user | Implementation |
| Elevation of Privilege | Patient edits JWT role claim to become admin | Auth Service | High | Store roles server side, never trust client token | Design |
| Elevation of Privilege | Receptionist reads GP notes because RBAC only enforced in the UI | Web App | Medium | Enforce RBAC at API and DB layer | Implementation |


Security Requirements

1. All pages need to use HTTPS, no plain HTTP anywhere. (Info Disclosure row 1)
2. Session cookies need HttpOnly, Secure and SameSite set so they can't be stolen easily. (Spoofing row 1)
3. JWT tokens should expire quickly and the signature needs to be checked on every request. (Spoofing row 2, EoP row 1)
4. Before reading or changing any record, check that the logged in user actually owns it. (Tampering row 1, Info Disclosure row 2)
5. Database accounts should only have the permissions they actually need. (Tampering row 2)
6. Every create, update or delete should be logged with who did it and when. (Repudiation rows 1 and 2)
7. Error messages shown to users should be kept vague, full details go to the server log only. (Info Disclosure row 1)
8. Login attempts should be limited to 10 per minute and lock the account after 5 wrong tries. (DoS row 1)
9. Each user should only be able to make 5 bookings per hour. (DoS row 2)
10. Role checks need to happen at the API level not just the front end. (EoP row 2)
11. User roles should live on the server, the app should never trust what the client says their role is. (EoP row 1)
12. Every booking or cancellation should trigger a confirmation email with a timestamp. (Repudiation row 2)
13. All patient data stored in the database should be encrypted. (Info Disclosure row 2)
14. Patients log in through NHS Login only, no option to set their own password. (Spoofing row 1)


Limitations of STRIDE

Supply chain is a blind spot. If NHS Login, the SMS gateway or an npm package gets compromised, STRIDE won't catch it because it treats those as trusted. Log4Shell is a good example of exactly what STRIDE misses.

Insider threats don't show up well either. A receptionist browsing patient records they shouldn't see isn't crossing a trust boundary, they're already inside one. STRIDE won't flag that.

It also goes stale. Once written it doesn't update itself when new features get added. The attack surface changes, the document doesn't.

PASTA is different because it starts from the attacker's perspective and ranks threats by actual business impact rather than just listing them. For an NHS system that framing would surface the really damaging stuff much faster.
