# crAPI Write-ups

[crAPI](https://github.com/OWASP/crAPI) (Completely Ridiculous API) is an intentionally vulnerable API maintained by OWASP for practicing the OWASP API Security Top 10. These are my findings from testing a local Docker instance with Burp Suite.

## Findings

| # | Vulnerability | Endpoint | Class | Severity | Write-up |
|---|---|---|---|---|---|
| 1 | BOLA: mechanic reports | `/workshop/api/mechanic/mechanic_report` | BOLA (API1:2023) | High | [Read](bola-crapi-mechanic-reports.md) |
| 2 | IDOR: | `[endpoint]` | BOLA (API1:2023) | [ ] | In progress |
| 3 | SQL injection: | `[endpoint]` | Injection (A03:2021) | [ ] | In progress |
| 4 | XSS: | `[endpoint]` | Injection (A03:2021) | [ ] | In progress |

## Tools
Docker · Chrome · Burp Suite 
