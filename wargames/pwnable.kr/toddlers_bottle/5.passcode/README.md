In this challenge, we need to login to an application with two passcodes.<br/>
We check the security properties and see that there's stack buffer overflow protection and code execution prevention.
```
uzi@pwnpatrol:$ checksec passcode
[*] '/home/uzi/challenges/wargames/pwnable.kr/toddlers_bottle/5.passcode/passcode'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```
Looking at the source code, the functions `welcome` and `login` are at the same level meaning that the stack we prepare in welcome,
can be used to manipulate behaviour of login.
![](images/image.png)
