# Software-Security-Assessment-7021SCN

Opeyemi Caleb Opeyemi
MSc Advanced Software Engineering
Student ID: 16980763
Software Security Module Assessment


## Portfolio Contents

| Folder | What it contains | Start here |
|---|---|---|
| task-a-vulnerability | Deliberately vulnerable C program, secure fixed version, and exploit notes covering stack buffer overflow | vulnerable.c, secure.c |
| task-b-threat-model | STRIDE threat model for an NHS appointment booking system with Mermaid DFD, threat table, security requirements, and STRIDE limitations | stride_model.md |
| task-c-static-analysis | Semgrep scan output and triage notes classifying each finding as true or false positive | semgrep_output.txt, triage_notes.md |
| task-d-dynamic-analysis | OWASP ZAP report, Burp Suite screenshots of SQL injection and XSS attacks against DVWA, and a pentest log | zap-report.html, pentest_log.md |
| task-e-sbom-compliance | Dockerfile, Syft-generated CycloneDX SBOM, Grype CVE scan output, and SSDF compliance mapping | sbom.json, grype_output.txt, compliance_notes.md |


## Task A - Stack Buffer Overflow

Exploit Notes - Stack Buffer Overflow

1. Vulnerability Description

CWE-121 - Stack-based Buffer Overflow

The bug is in greet_user() at line 7 of vulnerable.c. The function allocates a 64-byte buffer on the stack and copies user input into it with strcpy(), which does not check how long the input is. Anything over 64 bytes spills into adjacent stack memory and overwrites the saved base pointer and return address. The same problem exists in login() where gets() reads into username[32] and password[32] with no limit at all.


2. Input That Triggers the Overflow

    python3 -c "print('A' * 100)" | ./vulnerable

100 A's are passed as the name. The buffer is 64 bytes so the extra 36 bytes corrupt the stack frame above it.


3. GDB Register Output

    rax            0x6e                110
    rbx            0x7fffffffddd8      140737488346584
    rcx            0x0                 0
    rdx            0x0                 0
    rsi            0x5555555592a0      93824992252576
    rdi            0x7fffffffda50      140737488345680
    rbp            0x4141414141414141  0x4141414141414141
    rsp            0x7fffffffdc88      0x7fffffffdc88
    r8             0x73                115

The important line is rbp = 0x4141414141414141. The hex value 0x41 is ASCII for 'A', so the saved base pointer has been completely overwritten with our input. In a real exploit you would put a valid memory address there instead of A's. If rip (the instruction pointer) gets overwritten the same way, the CPU jumps to wherever the attacker points it when the function returns - that is full control of execution.


4. Valgrind Output

    ==7485== Source and destination overlap in strcpy(0x1ffefffc10, 0x1ffefffc70)
    ==7485==    at 0x484F413: strcpy (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
    ==7485==    by 0x10920B: greet_user (vulnerable.c:7)

Valgrind caught the strcpy writing past the end of buf. It traces it back to line 7 which is exactly where the overflow happens.

    ==7485== Jump to the invalid address stated on the next line
    ==7485==    at 0x4141414141414141: ???

When greet_user tried to return, the CPU tried to jump to 0x4141414141414141 because that is what was sitting in the return address slot on the stack. That address does not exist so it crashes.

    ==7485==  Address 0x4141414141414141 is not stack'd, malloc'd or (recently) free'd
    ==7485==  Bad permissions for mapped region at address 0x4141414141414141

Valgrind confirms the address has no valid mapping and no execute permissions.

    ==7485== ERROR SUMMARY: 2 errors from 2 contexts

Two errors total - the unsafe strcpy and the invalid jump. Both come from the same root cause.


5. Real-World Impact

In a real attack the input would not be a hundred A's. An attacker would work out the exact offset to the return address and put a pointer to their shellcode there instead. The shellcode would be injected earlier in the buffer or somewhere else in memory and when the function returns the CPU executes it with whatever privileges the process has. If this were a network service running as root that would mean full system compromise. This is how some of the oldest and most damaging exploits worked, including the Morris Worm in 1988. Stack canaries, ASLR and the NX bit exist specifically to make this harder, which is why they were turned off when compiling this binary.


## CI/CD Pipeline - Task C

The file .github/workflows/semgrep.yml runs a Semgrep security scan automatically on every push to main. It installs Semgrep, scans the repository using the p/c and OWASP Top Ten rule sets, and saves the output as a downloadable artifact. This means every code change is automatically checked for security issues. The scan found two true positive findings in vulnerable.c (gets() and unbounded scanf) and one false positive in secure.c (width-limited scanf flagged by a rule that catches all scanf usage).
