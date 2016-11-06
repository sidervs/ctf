
Running this file gives us an output:
`Poor Bernie.`

Looking at the main function we can see that this is really all it does:
```
0000000000400b3d <main>:
  400b3d:	55                   	push   rbp
  400b3e:	48 89 e5             	mov    rbp,rsp
  400b41:	48 83 ec 10          	sub    rsp,0x10
  400b45:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
  400b48:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
  400b4c:	bf 11 0c 40 00       	mov    edi,0x400c11
  400b51:	e8 8a f9 ff ff       	call   4004e0 <puts@plt>
  400b56:	b8 00 00 00 00       	mov    eax,0x0
  400b5b:	c9                   	leave  
  400b5c:	c3                   	ret    
  400b5d:	90                   	nop
  400b5e:	90                   	nop
  400b5f:	90                   	nop
```  
Besides the main function, there are a few functions which will never be called : "help", "fake_help", "real_help", "don't_call_me" 
and also c1, c1_, c2, c3, c4, c5, c8 and c55.
  
calling these functions using gdb reveals that most of them are using 
```
 mov rbx, m0
  call rbx
```
which gives us a Segmentaion fault.
 
"real_help" also has an interesting string printed :

```
Leonardo De Pisa? Who's that–The next president?
```
Leonardo De Pisa is Fibonacci's real name - which immediately clicks with the "c" functions
but calling c1, c1_, c2, c3, c5, c8 in succesion will result in a segmentation fault for the above reason.

c55 contains an infinite loop, but if it's condition were to be false we would replace `m0` with `0x400520` (malloc) 
```
0000000000400a3a <c55>:
  400a3a:	55                   	push   rbp
  400a3b:	48 89 e5             	mov    rbp,rsp
  400a3e:	48 83 ec 10          	sub    rsp,0x10
  400a42:	c7 45 fc 00 00 00 00 	mov    DWORD PTR [rbp-0x4],0x0
  400a49:	eb 19                	jmp    400a64 <c55+0x2a>
  400a4b:	b8 00 00 00 00       	mov    eax,0x0
  400a50:	e8 d3 fd ff ff       	call   400828 <c4>
  400a55:	bf 64 00 00 00       	mov    edi,0x64
  400a5a:	b8 00 00 00 00       	mov    eax,0x0
  400a5f:	e8 cc fa ff ff       	call   400530 <usleep@plt>
  400a64:	83 7d fc 0e          	cmp    DWORD PTR [rbp-0x4],0xe
  400a68:	7e e1                	jle    400a4b <c55+0x11>
  400a6a:	48 c7 05 f3 08 20 00 	mov    QWORD PTR [rip+0x2008f3],0x400520        # 601368 <m0>
  400a71:	20 05 40 00 
  400a75:	c9                   	leave  
  400a76:	c3                   	ret 
```  
so, by calling c55 first, but changing jle to jg we will change m0 to malloc and then we get the string: 

`flag{write_in_bernie!}`

This can be done with a .gdbinit file:
```
define push
	set $rsp -= 8
	set *(long*)$rsp = $arg0
end

b main
r
push c8
push c5
push c3
push c2
push c1_
push c1

set $rip = c55
set *(char*) 0x400a68 = 0x7f
c
```
