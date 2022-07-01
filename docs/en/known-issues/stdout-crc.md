# Invalid message sent to STDOUT

This message `validation failed on the message sent to STDOUT: see: ..., invalid message ...` means that you or some application sent a non-correct message (raw) to the STDOUT.  
Process `STDOUT` is reserved for the RR communication with the PHP process via the `goridge` protocol (`v3`).

How to fix that?  

1. Check your application. All dependencies should send their messages to the `STDERR` instead of `STDOUT`. You may see the output after the `invalid message`. It might be shrunk on `windows`.
2. Check `dd` , `echo` inserted by you. PHP workers can redirect `echo` automatically from the `STDOUT` to `STDERR` but only after the worker is initialized fully.
