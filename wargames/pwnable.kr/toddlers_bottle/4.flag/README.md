In this challenge, you need to reverse a binary. First of all, dowload the flag binary to your lab, and make it executable.
```
wget http://pwnable.kr/bin/flag
chmod u+x flag
```
When checking the security properties, we see an interesting line "Packed with UPX". So the binary seems to be packed that we can't reverse it without unpacking first.
```
uzi@pwnpatrol:$ checksec flag
[*] '/home/uzi/flag/flag'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
    Packer:   Packed with UPX
```
If not installed, install upx compressor/decompressor and unpack the binary
```
uzi@pwnpatrol:$ upx-ucl -d flag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94        Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017
        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    883745 <-    335288   37.94%   linux/amd64   flag
Unpacked 1 file.
```
Now checksec again
```
uzi@pwnpatrol:$ checksec flag
[*] '/home/uzi/flag/flag'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Let's first check with strings to see if there's sth useful. But unfortunately, strings outputs thousand (~7667) of text. Reason is as the following
```
uzi@pwnpatrol:$ file flag 
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.24,
```
Since it's statically linked, there are many strings for symbols. So we can't find anything either with strings, objdump or similar. 
So we need to disassemble the binary. Open the binary with gdb and disas main.
```
uzi@pwnpatrol:$ gdb -q flag
Reading symbols from flag...(no debugging symbols found)...done.
(gdb) disassemble main
Dump of assembler code for function main:
=> 0x0000000000401164 <+0>:     push   rbp
   0x0000000000401165 <+1>:     mov    rbp,rsp
   0x0000000000401168 <+4>:     sub    rsp,0x10
   0x000000000040116c <+8>:     mov    edi,0x496658
   0x0000000000401171 <+13>:    call   0x402080 <puts>
   0x0000000000401176 <+18>:    mov    edi,0x64
   0x000000000040117b <+23>:    call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401184 <+32>:    mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
   0x000000000040118b <+39>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:    mov    rsi,rdx
   0x0000000000401192 <+46>:    mov    rdi,rax
   0x0000000000401195 <+49>:    call   0x400320
   0x000000000040119a <+54>:    mov    eax,0x0
   0x000000000040119f <+59>:    leave  
   0x00000000004011a0 <+60>:    ret    
End of assembler dump.
```
Disassembly shows that there's a flag pointer and value pointed by it, is copied to rdx. Check its content.
```
(gdb) info symbol 0x6c2070
flag in section .data
(gdb) x 0x6c2070
0x6c2070 <flag>:        0x00496628
(gdb) x 0x00496628
0x496628:       0x2e585055
```
So it seems that flag is a global pointer, pointing to somedata in .rodata since `info files` shows that 0x00496628 in rodata section. Furthermore, looking at
`x 0x00496628`, it looks like there're some characters here (based on experience:)). So dump it and we got the flag!

```
(gdb) x/s 0x00496628
0x496628:       "UPX...? sounds like a delivery service :)"
```

