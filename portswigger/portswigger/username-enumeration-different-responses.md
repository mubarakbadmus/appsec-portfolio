Here is my full write-up for this Authentication Lab on PortSwigger Web Security Academy. 

## Username Enumeration via Different Responses

**Target:** PortSwigger Web Security Academy 
**Severity:** Low-To-Medium
**Vulnerability Class:** Broken Authentication (Identification and Authentication Failures)

### Summary
The application's login endpoint returns different responses depending on whether a submitted username exists in the system. This allows an attacker to enumerate valid usernames without needing a correct password, undermining the confidentiality of account information.

### Steps to Reproduce
1. Navigate to the login page and open Burp Suite to intercept the login request.
2. Submit a login attempt with a **An invalid username** and any password (e.g., `notarealuser` / `password123`).
   - Observed response: `"Invalid username"`
3. Submit a login attempt with a **valid username** (obtained from the lab's user list or guessed) and an incorrect password (e.g., `carlos` / `password123`).
   - Observed response: `"Incorrect password"`
4. Compare the two responses  the application returns a distinct error message for each case, confirming the username's validity independent of the password.
5. Send both requests to Burp Intruder, load a username wordlist, and run the attack then filter results by response text (`"Incorrect password"` vs `"Invalid username"`) to identify all valid usernames.

### Impact
An attacker can compile a list of confirmed valid usernames on the platform. This list can then be used for:
- Targeted credential stuffing (testing leaked passwords against known real accounts)
- Password spraying (trying one common password across many confirmed accounts)
- Targeted phishing campaigns aimed at real users

While this finding alone doesn't grant direct access, it materially lowers the effort required for follow-on attacks which is why it's classified low-to-medium rather than critical.

### Root Cause
The authentication logic checks username existence and password correctness as two separate steps, each returning a distinct, human-readable error message. The application never normalizes these responses into a single generic failure state.

### Remediation
- Return an identical, generic error message for all failed login attempts (e.g., `"Invalid username or password"`), regardless of whether the username or password was incorrect.
- Ensure response time, HTTP status code, and response length are also consistent between the two failure cases, since timing/length differences can leak the same information even with identical error text.
- Consider rate-limiting or CAPTCHA on repeated failed login attempts to slow down enumeration attempts generally.

### My thoughs 
This lab reinforced that authentication flaws aren't always about bypassing a password check directly, subtle information leakage in error handling can be just as damaging by enabling downstream attacks. It's also a good example of why generic, symmetric error responses are a baseline requirement for any login system.
