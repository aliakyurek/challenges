In this basic challange, all you need to know is that, file descriptor for stdin is 0.
So when you provide decimal value of 0x1234 as first argument to fd binary, you can send "LETMEWIN" as an input to it and get the flag.

Along with the script, challange can be solved with the following one-liner as well:
```
fd@prowl:~$ printf "LETMEWIN\n" | ./fd `printf %d 0x1234`
good job :)
mommy! I think I know what a file descriptor is!!
```
