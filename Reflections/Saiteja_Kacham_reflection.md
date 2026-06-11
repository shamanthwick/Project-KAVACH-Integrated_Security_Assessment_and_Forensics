# Reflections: Project KAVACH – Web Application Assessment
**Author: Saiteja Kacham**

## The Challenge
The primary challenge in this workstream was moving beyond "automated scanning" to understand the logic behind the vulnerabilities. While tools like SAST can flag potential issues, manually exploitation of an IDOR or a Command Injection in the lab was much more rewarding. It required a deep understanding of how HTTP requests are structured and how the backend server processes user-provided strings.

## Key Technical Learnings
1.  **Context-Aware Escaping:** I learned that protecting against XSS isn't just about "stripping tags"—it's about context. A payload that is safe in an HTML body might still be dangerous inside a JavaScript attribute or a CSS style.
2.  **The Danger of `shell_exec`:** Seeing a simple `; ls -la` payload work on the DVWA command injection page was a wake-up call. It highlighted why developers should almost always avoid executing shell commands directly from web inputs.
3.  **API Transparency:** Analyzing Juice Shop showed me how modern single-page apps (SPAs) often "over-share" information through their APIs. We found version numbers and endpoint structures just by watching the Network tab in DevTools.

## Successes and "Aha" Moments
My biggest "aha" moment was successfully executing a `UNION`-based SQL injection to dump the database user list. Seeing the internal data appear on the screen after just a few characters of input made the risk of "Injection" (A03 in OWASP) feel very real. Also, using Docker to manage the lab environment made the entire process incredibly clean and repeatable.

## Strategic Impact
This workstream proved that application-layer security is the "front door" of the organization. A single SQLi vulnerability doesn't just leak one record; it can potentially give an attacker a foothold to move into the network layer (connecting Workstream B to Workstream A). My recommendation to use parameterized queries and enforce server-side authorization isn't just a coding fix—it's a critical business safeguard.
