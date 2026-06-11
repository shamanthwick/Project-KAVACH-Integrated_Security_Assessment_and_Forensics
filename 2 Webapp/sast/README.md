# SAST Notes

The Project KAVACH brief asks for `before.json` and `after.json` static-analysis reports. This repository does not include the DVWA or OWASP Juice Shop source trees, so a real Semgrep or ESLint scan cannot be truthfully produced from the current artifacts.

This folder therefore contains structured manual SAST placeholders:

- `before.json`: the expected vulnerable source patterns based on the demonstrated findings.
- `after.json`: the expected clean state after applying the proposed remediation patches.

To make this a real SAST diff, add the application source code or clone the vulnerable app source into a separate review folder, run Semgrep/ESLint before fixes, apply the remediation patches, and run the same scanner again.
