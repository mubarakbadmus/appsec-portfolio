# BOLA in crAPI Mechanic Reports

**Target:** crAPI (OWASP Completely Ridiculous API), local Docker instance
**Severity:** High. Any authenticated user can read other users' personal data, and the report IDs are sequential and guessable.
**Vulnerability Class:** Broken Object Level Authorization (OWASP API Security Top 10, API1:2023), commonly called IDOR

## Summary

The `mechanic_report` endpoint returns a repair report based only on the `report_id` query parameter. It never checks that the report belongs to the user making the request. By changing the ID, any logged-in user can read other users' reports, which contain emails, phone numbers, VINs, and problem descriptions.

## Steps to Reproduce

1. Register a new account in crAPI and log in.
2. Add a vehicle to the account, then use **Contact Mechanic** to submit a service request.
3. In Burp, inspect the response to `POST /workshop/api/merchant/contact_mechanic`. It contains a link to a report endpoint that is not shown anywhere in the UI: `/workshop/api/mechanic/mechanic_report?report_id={id}`.
4. Send that request to Burp Repeater with the `report_id` from your own service request. The API returns your own report.
5. Change `report_id` to a different value (for example, one number lower or higher) and resend. The API returns a report belonging to another user, with their personal and vehicle details.

## Proof of Concept

Request:

<img width="1020" height="653" alt="crapi0" src="https://github.com/user-attachments/assets/926f07a6-c5c1-4b0a-8e07-43147eea84e8" />

```http
GET /workshop/api/mechanic/mechanic_report?report_id=<other_user_report_id> HTTP/1.1
Host: localhost:8888
Authorization: Bearer eyJhbG
Accept: application/json
```

Response (trimmed):
<img width="1017" height="647" alt="crapi2" src="https://github.com/user-attachments/assets/3ff267bc-9094-483d-831b-668e231bdca8" />

```json
"vehicle": {
    "id": 3,
    "vin": "0NKPZ09IHOP508673",
    "owner": {
      "email": "robot001@example.com",
      "number": "9876570001"
    }
  },
  "problem_details": "My car BMW - 5 Series is having issues.\nCan you give me a call on my mobile 9876570001,\nOr send me an email at robot001@example.com\nThanks,\nRobot.",
  "status": "completed",
  "created_on": "25 September, 2026, 01:13:21",
  "updated_on": null,
  "comments": []
}
```

## Impact

Report IDs are sequential integers, so an attacker with a single low-privilege account can script requests across the whole ID range and harvest every user's email address, phone number, VIN, and service problem description. That data can be used for targeted phishing, for identifying specific vehicles and owners, and for social engineering. Because the attack needs only a normal account and no special tooling, the barrier is low, which supports the High rating.

## Root Cause

The endpoint handler uses the user-supplied `report_id` directly in its database lookup and never compares the record's owner to the authenticated user. The API checks *who you are* (authentication) but not *what you are allowed to access* (authorization). The endpoint was also discoverable from an unrelated response, which made it easy to find, but the real flaw is the missing ownership check.

## Remediation

- Enforce object-level authorization on the server for every request that takes an object ID, for example `filter_by(id=report_id, owner_id=current_user.id)`, and return 403 or 404 when the record does not belong to the caller.
- Apply the same check consistently across all endpoints that accept object identifiers, not just this one.
- Use unpredictable identifiers (such as UUIDs) as defense in depth. This makes enumeration harder but does not replace the authorization check.

## Reflection

This lab showed that a valid token only proves identity, not entitlement. It also showed how useful it is to read full API responses closely, since a link in an unrelated response led straight to the vulnerable endpoint.
