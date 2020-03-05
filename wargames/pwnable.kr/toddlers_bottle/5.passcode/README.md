In this challenge, we need to login to an application with two passcodes. In description of game, it's said that they ignored compiler warning (due to no reference passed to scanf) <br/>
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
can be used to manipulate behaviour of login. So this is the stack frame of `welcome` function
![](images/image.png)

* Yellow: $ebp and ret_addr pair
* Green: Stack canary
* Red: Bytes in the range between two red bars are name[100]
* Purple: Stack of of `login`.
    * `passcode1` overlaps with the last 4 bytes of name[100] in previous stack
    * `passcode2` overlaps with the stack canary in previous stack. Notice that there's no stack canary in stack of `login`. It seems that compiler does generate it when there's no risk of overflowing.

If there was only the first and single passcode check, we could pass this challenge by:
* Handcrafting a 100 bytes payload in `welcome` where the last 4 bytes are same as `passcode1`.
* Not providing an integer input for `scanf("%d", passcode1);` so that scanf will not try to update the value pointed by `passcode1`. As said in the beginning, this is the programming error compiler warns. If you provide an integer other than the address of a writable memory region, to those `scanf`s, there will be a segmentation fault.
