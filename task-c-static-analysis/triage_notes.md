Semgrep Triage Notes

Finding 1
Rule: c.lang.security.insecure-use-gets-fn
File: vulnerable.c lines 18 and 20
Verdict: True positive

gets() doesn't care how long the input is, it just keeps reading. Both lines correctly flagged.


Finding 2
Rule: c.lang.security.insecure-use-scanf-fn
File: vulnerable.c line 33
Verdict: True positive

No width limit on the scanf so it can overflow the buffer. Fair flag.


Finding 3
Rule: c.lang.security.insecure-use-scanf-fn
File: secure.c line 49
Verdict: False positive

This one is using "%15s" which already limits the input to fit the buffer. Semgrep flagged it anyway because the rule just sees scanf and raises alarm regardless. The code is actually fine here.


What Semgrep can't do

It only reads code, doesn't run it. So it won't tell you a crash actually happened, just that something looks off. That's why we also used GDB and Valgrind.

It also doesn't follow data across functions very well. The full path from user input in main all the way to the strcpy in greet_user, Semgrep only caught pieces of that. Something like CodeQL would trace the whole thing.

Logic bugs it won't catch at all. If the login function had a flaw that let anyone in, Semgrep would miss it completely. That's where dynamic testing comes in.
