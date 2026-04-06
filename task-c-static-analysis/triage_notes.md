Semgrep Triage Notes

Finding 1
Rule: c.lang.security.insecure-use-gets-fn
File: task-a-vulnerability/vulnerable.c
Lines: 18 and 20
Verdict: True positive

gets() has no way to limit how much it reads. It will keep reading until it hits a newline regardless of how big the buffer is. On line 18 it reads into username which is only 32 bytes and on line 20 it does the same into password. Anything longer than the buffer will overflow into the stack. Semgrep is right to flag this.


Finding 2
Rule: c.lang.security.insecure-use-scanf-fn
File: task-a-vulnerability/vulnerable.c
Line: 33
Verdict: True positive

scanf("%s", name) on line 33 reads into a 16-byte buffer with no width limit in the format string. An attacker can type more than 16 characters and overflow the buffer. Semgrep is right here as well.


Finding 3
Rule: c.lang.security.insecure-use-scanf-fn
File: task-a-vulnerability/secure.c
Line: 49
Verdict: False positive

This one is debatable. The format string is "%15s" which tells scanf to read at most 15 characters, leaving one byte for the null terminator. The buffer is 16 bytes so there is no overflow possible here. Semgrep flags all use of scanf regardless of whether a width limit is present because the rule is written to catch the whole function, not just the unsafe uses of it. So the flag is technically incorrect in this specific case even though the reasoning behind the rule is sound in general.


What Semgrep cannot detect

Semgrep is a static analysis tool which means it only reads the code, it does not run it. This creates some blind spots.

Runtime behaviour. Semgrep cannot tell what values variables will actually hold when the program runs. It cannot detect a crash or a segfault, it can only spot patterns in the code that look dangerous. To actually see the program crash you need to run it, which is what GDB and Valgrind are for.

Logic flaws. If the program does the wrong thing for the wrong reason but uses safe functions, Semgrep will not catch it. For example if the authentication logic had a flaw that allowed any password to succeed, Semgrep would not flag that because no unsafe function is being called.

Cross-function data flow. Semgrep does basic pattern matching. It struggles to follow data from where it enters the program all the way through multiple functions to where it gets used unsafely. In vulnerable.c the user input comes in through scanf in main, gets passed to greet_user, and then gets copied with strcpy. A tool like CodeQL which builds a full data flow graph would trace that entire path. Semgrep mostly caught the individual dangerous calls rather than the full chain.

Dynamic analysis comparison. Tools like Valgrind and GDB complement Semgrep well. Semgrep finds suspicious patterns before the code even runs. Valgrind and GDB confirm the actual impact at runtime. Using both gives a much clearer picture than either tool alone.
