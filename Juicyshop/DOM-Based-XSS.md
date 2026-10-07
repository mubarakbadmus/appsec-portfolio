# DOM-Based XSS in Search Bar >>> OWASP Juice Shop

## Summary

The OWASP Juice Shop application contains a DOM-based Cross-Site Scripting (XSS) vulnerability in its product search feature. User-supplied input from the search bar is written directly into the DOM without sanitization or encoding, allowing an attacker to inject arbitrary HTML/JavaScript that executes in the context of the victim's browser session.

**Target**:  OWASP Juice Shop (training application) |
| **Vulnerability Class** | DOM-Based Cross-Site Scripting (CWE-79) |
| **Component** | Product Search Bar |
| **Severity** | High (per OWASP Top 10: A03:2021 – Injection) |
| **Status** | Intentional vulnerability — Juice Shop is a deliberately insecure training app |

## Background

[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) is an intentionally vulnerable web application maintained by OWASP, used for security training, CTFs, and tooling demos. This finding documents a well-known built-in challenge, reproduced here to demonstrate practical understanding of DOM XSS: how it arises, how to identify it, and how to remediate it.

## Vulnerability Details

The search bar takes the query parameter and renders search-related content back to the page. Instead of treating the input as plain text, the application inserts it into the DOM in a way that allows HTML tags to be parsed and executed a classic **sink** in DOM XSS terminology (e.g., `innerHTML` or an Angular binding that doesn't sanitize by default in this context).

Because the payload never leaves the browser (no server round-trip is required to trigger execution), this is classified as **DOM-based** XSS rather than reflected/stored XSS, even though the search term is also visible in the URL.

## Steps to Reproduce

1. Navigate to the Juice Shop application (e.g., `http://localhost:3000`).
2. Click the search icon in the top navigation bar to reveal the search input.
3. Enter the following payload into the search field:

   ```html
   <iframe src="javascript:alert(`xss`)">
   ```

4. Press Enter to submit the search.
5. Observe that a JavaScript `alert()` dialog fires immediately, confirming that the injected markup was parsed and executed by the browser rather than rendered as literal text.

## Proof of Concept (Payload)

```html
<iframe src="javascript:alert(`xss`)">
```

*(Screenshot placeholder: insert before/after screenshots showing the search input and the resulting alert box.)*

## Impact

In a real-world (non-training) application, this class of vulnerability could allow an attacker to:

- Execute arbitrary JavaScript in a victim's browser session
- Steal session tokens or cookies (session hijacking)
- Perform actions on behalf of the victim (CSRF-style abuse via script)
- Redirect users to phishing pages
- Deface or manipulate visible page content

Since the payload can be delivered via a crafted URL (e.g., `https://target.com/search?q=<payload>`), an attacker could distribute a malicious link via email, chat, or a compromised third-party site to trigger the exploit against any user who clicks it.

## Root Cause

The application inserts user-controlled input into the DOM without:
- HTML-encoding special characters (`<`, `>`, `"`, `'`), and/or
- Using a safe DOM API (e.g., `textContent` instead of `innerHTML`), and/or
- Applying a strict Content Security Policy (CSP) that would block inline script execution

## Remediation

1. **Sanitize/encode output** : Encode all user-supplied data before inserting it into the DOM. Use framework-safe binding methods (e.g., Angular's default interpolation `{{ }}`, which auto-escapes, rather than binding via `[innerHTML]` with raw input).
2. **Use a sanitization library** : If HTML rendering is required, sanitize input through a vetted library such as DOMPurify before insertion.
3. **Implement Content Security Policy (CSP)** :  A strict CSP (e.g., disallowing `unsafe-inline` scripts) provides defense-in-depth even if a sanitization bypass is found.
4. **Input validation** : Reject or strip disallowed characters/tags at the point of input where feasible.


## References

- [OWASP Juice Shop GitHub](https://github.com/juice-shop/juice-shop)
- [OWASP Top 10 – A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [OWASP DOM Based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
