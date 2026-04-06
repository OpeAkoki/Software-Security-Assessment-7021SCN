SSDF Compliance Mapping

This document maps portfolio artifacts to NIST Secure Software Development Framework (SSDF) practices.

SSDF Practice ID | Practice Name | How the portfolio satisfies it | File location
---|---|---|---
PO.1.1 | Define security requirements | The STRIDE threat model defines security requirements for the NHS appointment system across all six threat categories. Fourteen formal security requirements (SR-01 to SR-14) were derived directly from the threat analysis and each traces back to a specific threat row. | task-b-threat-model/stride_model.md
PO.3.1 | Create and maintain a secure development environment | GitHub Actions runs Semgrep on every push to main, ensuring the pipeline automatically checks for insecure code patterns before any changes are merged. | .github/workflows/semgrep.yml
PW.1.1 | Design software to meet security requirements | secure.c was written to address the exact vulnerabilities identified in vulnerable.c, replacing gets() with fgets(), strcpy() with strncpy(), and unbounded scanf() with a width-limited format specifier. Each fix references CERT C STR31-C. | task-a-vulnerability/secure.c
PW.4.1 | Reuse secured software components | Syft was used to generate a full CycloneDX SBOM of the Docker image, listing every dependency and its version. Grype then scanned the SBOM against the NVD to identify known vulnerable components, satisfying the requirement to verify third-party software before use. | task-e-sbom-compliance/sbom.json, grype_output.txt
PW.5.1 | Create source code by following secure coding practices | Semgrep scanned the codebase using the p/c and OWASP Top Ten rule sets. The triage notes document each finding, classify true and false positives, and explain the root cause of each issue. | task-c-static-analysis/semgrep_output.txt, triage_notes.md
RV.1.1 | Identify and confirm vulnerabilities | Grype scanned the SBOM and identified 61 vulnerabilities across the Docker image including High severity CVEs in libcrypto3, libssl3, and the node binary itself. The output is saved and committed as evidence. | task-e-sbom-compliance/grype_output.txt
RV.1.2 | Assess the exploitability of vulnerabilities | The pentest log documents SQL injection and reflected XSS findings against DVWA with proof of concept inputs, Burp Suite evidence, and CWE references, demonstrating active exploitation of known vulnerability classes. | task-d-dynamic-analysis/pentest_log.md
RV.2.1 | Fix vulnerabilities and track their resolution | vulnerable.c documents the insecure code and secure.c provides the fixed version. The Semgrep triage notes explain why each fix addresses the flagged issue and reference the relevant CERT C rule. | task-a-vulnerability/vulnerable.c, secure.c


Comparison of SSDF and NCSC Secure Development Guidance

The NIST SSDF and the NCSC Secure Development and Deployment guidance share the same goal of building security into the development process rather than bolting it on at the end, but they take different approaches in how they present that goal.

The SSDF is structured around discrete, auditable practices with unique IDs like PO.1.1 and RV.1.1. Each practice has a clear description and expected outputs which makes it straightforward to map evidence against specific requirements. This makes SSDF well suited to formal compliance exercises and procurement contexts where an organisation needs to demonstrate exactly how it meets each requirement. The tradeoff is that it can feel bureaucratic and the practice descriptions are deliberately technology-agnostic, which means teams sometimes need to do a lot of interpretation before the guidance becomes actionable.

The NCSC guidance is more principles-based and written in plain language. It covers similar ground including threat modelling, dependency management, secure configuration, and vulnerability disclosure but frames everything as practical advice rather than auditable controls. It is easier to read and follow for a development team that does not have a compliance background, but it is harder to use as evidence in a formal audit because there are no practice IDs to map against.

In practice the two are complementary. NCSC guidance is useful for day to day secure development decisions while SSDF provides the structure needed to demonstrate compliance to a regulator or client. For an NHS-facing system both would be relevant given the combination of clinical safety requirements and the need to satisfy NHS Digital and ICO scrutiny.
