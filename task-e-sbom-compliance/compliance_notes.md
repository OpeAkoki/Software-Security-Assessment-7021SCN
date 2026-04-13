SSDF Compliance Mapping

This maps each portfolio task to a practice in the NIST Secure Software Development Framework (SSDF).

SSDF Practice ID | Practice Name | How the portfolio satisfies it | File location
---|---|---|---
PO.1.1 | Define security requirements | The STRIDE threat model works out what could go wrong with the NHS appointment system and turns that into a list of 14 security requirements. Each requirement traces back to a specific threat in the table. | task-b-threat-model/stride_model.md

PO.3.1 | Keep the build environment secure | GitHub Actions runs Semgrep every time code is pushed to main. So any new code gets scanned for security issues automatically before it stays in the repo. | .github/workflows/semgrep.yml

PW.1.1 | Build software to meet security requirements | secure.c was written to fix the exact problems found in vulnerable.c. Every unsafe function was replaced with a safer one and each change is commented with the relevant rule. | task-a-vulnerability/secure.c

PW.4.1 | Check third-party components before using them | Syft was used to list every package inside the Docker image. Grype then checked that list against a public database of known vulnerabilities to see what needs fixing. | task-e-sbom-compliance/sbom.json, grype_output.txt

PW.5.1 | Write code securely | Semgrep scanned the whole codebase and the triage notes explain each finding, whether it is a real problem or a false alarm, and why. | task-c-static-analysis/semgrep_output.txt, triage_notes.md

RV.1.1 | Find vulnerabilities | Grype found 61 issues in the Docker image including several high severity ones in the SSL libraries and the Node runtime. The full list is saved in grype_output.txt. | task-e-sbom-compliance/grype_output.txt

RV.1.2 | Check if vulnerabilities can actually be exploited | The pentest log shows SQL injection and XSS being exploited against DVWA with the actual inputs used and screenshots from Burp Suite as proof. | task-d-dynamic-analysis/pentest_log.md

RV.2.1 | Fix vulnerabilities | vulnerable.c shows the broken code and secure.c shows the fixed version. The triage notes explain why each fix works. | task-a-vulnerability/vulnerable.c, secure.c


SSDF vs NCSC Secure Development Guidance

Both SSDF and the NCSC Secure Development and Deployment guidance are trying to get developers to think about security throughout the build process rather than just at the end. But they go about it differently.

SSDF gives everything a reference number like PO.1.1 so you can point to specific practices and show evidence against each one. That makes it useful when you need to prove to someone outside the team, like a client or an auditor, that you have done certain things. The downside is it takes some work to translate the practice descriptions into actual tasks because they are written to be generic enough to apply to any technology.

The NCSC guidance is more like a set of tips written in plain English. It covers similar ground but reads more like advice than a checklist. A developer who has never done a security course could pick it up and understand what to do. The tradeoff is that it is harder to use as formal evidence because there are no IDs to reference.

In practice you would probably use both. The NCSC guidance helps the team make good decisions day to day. The SSDF gives you the structure to show your work to anyone who needs to audit it. For something like an NHS system both would be relevant because you have to satisfy both clinical safety requirements and data protection rules at the same time.
