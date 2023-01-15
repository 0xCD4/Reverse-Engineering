# Let's get dynamic RE

Hello InfoSec,

welcome back to my another blog about reversing challenge. Today, i am going to be writing a write-up called *Let's dynamic* challenge on PicoCTF.

As usual, in order to complete this challenge we need to analyze the file as well as checking what kind of instructions are being used. So let's dive into it.


## Analyzing the file

With the help of given hint: 

`Running this in a debugger would be helpful` this means we need to compile the file so as to run and check the behaviour of the file. Let me compile, and check what the file does.
But, I want to read the written instructions obviously.


```asm
main:
.LFB5:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	pushq	%rbx
	subq	$296, %rsp
	.cfi_offset 3, -24
	movl	%edi, -292(%rbp)
	movq	%rsi, -304(%rbp)
	movq	%fs:40, %rax
	movq	%rax, -24(%rbp)
	xorl	%eax, %eax
	movabsq	$4137700413143496212, %rax
	movabsq	$3668774195188830448, %rdx
	movq	%rax, -144(%rbp)
	movq	%rdx, -136(%rbp)
	movabsq	$-3415231997387159298, %rax
	movabsq	$3180240096696696075, %rdx
	movq	%rax, -128(%rbp)
	movq	%rdx, -120(%rbp)
	movabsq	$-5717177924950513641, %rax
	movabsq	$-3967246834314051972, %rdx
	movq	%rax, -112(%rbp)
	movq	%rdx, -104(%rbp)
	movw	$97, -96(%rbp)
	movabsq	$6214777055764401527, %rax
	movabsq	$8184225536171504527, %rdx
	movq	%rax, -80(%rbp)
	movq	%rdx, -72(%rbp)
	movabsq	$-8364134581669616439, %rax
	movabsq	$5916610601309242417, %rdx
	movq	%rax, -64(%rbp)
	movq	%rdx, -56(%rbp)
	movabsq	$-2598080388612165765, %rax
	movabsq	$-4252370736625094538, %rdx
	movq	%rax, -48(%rbp)
	movq	%rdx, -40(%rbp)
	movw	$63, -32(%rbp)
	movq	stdin(%rip), %rdx
	leaq	-208(%rbp), %rax
	movl	$49, %esi
	movq	%rax, %rdi
	call	fgets@PLT
	movl	$0, -276(%rbp)
	jmp	.L2
  
 
.L3:
	movl	-276(%rbp), %eax
	cltq
	movzbl	-144(%rbp,%rax), %edx
	movl	-276(%rbp), %eax
	cltq
	movzbl	-80(%rbp,%rax), %eax
	xorl	%eax, %edx
	movl	-276(%rbp), %eax
	xorl	%edx, %eax
	xorl	$19, %eax
	movl	%eax, %edx
	movl	-276(%rbp), %eax
	cltq
	movb	%dl, -272(%rbp,%rax)
	addl	$1, -276(%rbp)
.L2:
	movl	-276(%rbp), %eax
	movslq	%eax, %rbx
	leaq	-144(%rbp), %rax
	movq	%rax, %rdi
	call	strlen@PLT
	cmpq	%rax, %rbx
	jb	.L3
	leaq	-272(%rbp), %rcx
	leaq	-208(%rbp), %rax
	movl	$49, %edx
	movq	%rcx, %rsi
	movq	%rax, %rdi
	call	memcmp@PLT
	testl	%eax, %eax
	je	.L4
	leaq	.LC0(%rip), %rdi
	call	puts@PLT
	movl	$0, %eax
	jmp	.L6
.L4:
	leaq	.LC1(%rip), %rdi
	call	puts@PLT
	movl	$1, %eax
.L6:
	movq	-24(%rbp), %rcx
	xorq	%fs:40, %rcx
	je	.L7
	call	__stack_chk_fail@PLT
.L7:
	addq	$296, %rsp
	popq	%rbx
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE5:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0"
	.section	.note.GNU-stack,"",@progbits

```
As we can see from this source code that we are not able to read the instructions properly, so it will be useful that i drop the file to *gdb*.

## Using gdb

GDB would always be useful to disassemble functions. We will get started to disassemble main fucnctions. In orde to view written functions you can use:

```asm
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
0x0000000000001000  _init
0x0000000000001030  puts@plt
0x0000000000001040  strlen@plt
0x0000000000001050  __stack_chk_fail@plt
0x0000000000001060  memcmp@plt
0x0000000000001070  fgets@plt
0x0000000000001080  __cxa_finalize@plt
0x0000000000001090  _start
0x00000000000010c0  deregister_tm_clones
0x00000000000010f0  register_tm_clones
0x0000000000001130  __do_global_dtors_aux
0x0000000000001170  frame_dummy
0x0000000000001179  main
0x0000000000001348  _fini

```

As you can see there are interesting functions such as:

```c
- main()
- memcmp()
```

I want to check `main` function.

```asm
   0x00000000000012d7 <+350>:   call   0x1040 <strlen@plt>
   0x00000000000012dc <+355>:   cmp    rbx,rax
   0x00000000000012df <+358>:   jb     0x1282 <main+265>
   0x00000000000012e1 <+360>:   lea    rcx,[rbp-0x110]
   0x00000000000012e8 <+367>:   lea    rax,[rbp-0xd0]
   0x00000000000012ef <+374>:   mov    edx,0x31
   0x00000000000012f4 <+379>:   mov    rsi,rcx
   0x00000000000012f7 <+382>:   mov    rdi,rax
   0x00000000000012fa <+385>:   call   0x1060 <memcmp@plt>     <---
   0x00000000000012ff <+390>:   test   eax,eax
   0x0000000000001301 <+392>:   je     0x1316 <main+413>
   0x0000000000001303 <+394>:   lea    rdi,[rip+0xcfe]        # 0x2008
   0x000000000000130a <+401>:   call   0x1030 <puts@plt>
   0x000000000000130f <+406>:   mov    eax,0x0
   0x0000000000001314 <+411>:   jmp    0x1327 <main+430>
   0x0000000000001316 <+413>:   lea    rdi,[rip+0xd0a]        # 0x2027
   0x000000000000131d <+420>:   call   0x1030 <puts@plt>
   0x0000000000001322 <+425>:   mov    eax,0x1
   0x0000000000001327 <+430>:   mov    rcx,QWORD PTR [rbp-0x18]
   0x000000000000132b <+434>:   xor    rcx,QWORD PTR fs:0x28
   0x0000000000001334 <+443>:   je     0x133b <main+450>
   0x0000000000001336 <+445>:   call   0x1050 <__stack_chk_fail@plt>
   0x000000000000133b <+450>:   add    rsp,0x128
   0x0000000000001342 <+457>:   pop    rbx
   0x0000000000001343 <+458>:   pop    rbp
   0x0000000000001344 <+459>:   ret

```

I would like to check `memcmp`, because lets read the definition.

memcmp --> 
``` 
The C library function int memcmp(const void *str1, const void *str2, size_t n)) compares the first n bytes of memory area str1 and memory area str2.


    str1 − This is the pointer to a block of memory.

    str2 − This is the pointer to a block of memory.

    n − This is the number of bytes to be compared.
```
Awesome. I reckon that it takes input from the user and compare it with being holded string in the memory. In order to run this file, we need to return to the terninal.



## Running the file

```bash
virus@trojan:~/reverse/PicoCTF/LetsDynamic$ ./chall
aaaa
Correct! You entered the flag.

```
We already knew that the output is not correct, because we are expecting from the output `the flag` however, it did not give any output.
And We already seen that the file has been written with `memcmp` I am going to use `ltrace` command to see what it takes.


```asm
virus@trojan:~/reverse/PicoCTF/LetsDynamic$ ltrace ./chall
fgets(aaaaaa
"aaaaaa\n", 49, 0x7f763b6e6aa0)                                     = 0x7ffc45fc4b90
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49

strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
strlen("\024\266gp\232\020l9\360\220YI\261\032\3522\376\3100\322\227\250\232\320\v}\017\260\204{","...) = 49
memcmp(0x7ffc45fc4b90, 0x7ffc45fc4b50, 49, 0x7ffc45fc4b50)                = 0xfffffff1
puts("Correct! You entered the flag."Correct! You entered the flag.
)                                    = 31
+++ exited (status 0) +++

```

Interesting. We see a memory address in the `memcmp` so as to check what the memory holds we can use `gdb`



## Finding the flag

We know everything about this file, only what we need to do, is checking the memory address. You can follow these steps to get the flag :]


```asm
   0x00005555555552fa <+385>:   call   0x555555555060 <memcmp@plt>     <---- Important
   0x00005555555552ff <+390>:   test   eax,eax
   0x0000555555555301 <+392>:   je     0x555555555316 <main+413>
   0x0000555555555303 <+394>:   lea    rdi,[rip+0xcfe]        # 0x555555556008
   0x000055555555530a <+401>:   call   0x555555555030 <puts@plt>
   0x000055555555530f <+406>:   mov    eax,0x0
   0x0000555555555314 <+411>:   jmp    0x555555555327 <main+430>
   0x0000555555555316 <+413>:   lea    rdi,[rip+0xd0a]        # 0x555555556027
   0x000055555555531d <+420>:   call   0x555555555030 <puts@plt>

```

We are willing to access the memory of `memcmp` so we will try.

```asm
gdb-peda$ disassemble memcmp
Dump of assembler code for function memcmp@plt:
   0x0000000000001060 <+0>:     jmp    QWORD PTR [rip+0x2f62]        # 0x3fc8 <memcmp@got.plt>
   0x0000000000001066 <+6>:     push   0x3
   0x000000000000106b <+11>:    jmp    0x1020
End of assembler dump.
```

Awesome! Now we are going to access the memory address. In order to get access we type `info functions`

```asm
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
0x0000555555555000  _init
0x0000555555555030  puts@plt
0x0000555555555040  strlen@plt
0x0000555555555050  __stack_chk_fail@plt
0x0000555555555060  memcmp@plt      <---- important
0x0000555555555070  fgets@plt
0x0000555555555080  __cxa_finalize@plt
0x0000555555555090  _start
0x00005555555550c0  deregister_tm_clones
0x00005555555550f0  register_tm_clones
0x0000555555555130  __do_global_dtors_aux
0x0000555555555170  frame_dummy
0x0000555555555179  main
0x0000555555555348  _fini
gdb-peda$

```

Lets check the result:


```asm
gdb-peda$ r
Starting program: /home/virus/reverse/PicoCTF/LetsDynamic/chall
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
aaaaa
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffe010 --> 0xa6161616161 ('aaaaa\n')
RBX: 0x31 ('1')
RCX: 0x7fffffffdfd0 ("picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}")
RDX: 0x31 ('1')
RSI: 0x7fffffffdfd0 ("picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}")
RDI: 0x7fffffffe010 --> 0xa6161616161 ('aaaaa\n')
RBP: 0x7fffffffe0e0 --> 0x1
RSP: 0x7fffffffdfa8 --> 0x5555555552ff (<main+390>:     test   eax,eax)
RIP: 0x555555555060 (<memcmp@plt>:      jmp    QWORD PTR [rip+0x2f62]        # 0x555555557fc8 <memcmp@got.plt>)
R8 : 0x0
R9 : 0x5555555592a0 --> 0xa6161616161 ('aaaaa\n')
R10: 0x77 ('w')
R11: 0x246
R12: 0x7fffffffe1f8 --> 0x7fffffffe462 ("/home/virus/reverse/PicoCTF/LetsDynamic/chall")
R13: 0x555555555179 (<main>:    push   rbp)
R14: 0x555555557da0 --> 0x555555555130 (<__do_global_dtors_aux>:        endbr64)
R15: 0x7ffff7ffd040 --> 0x7ffff7ffe2e0 --> 0x555555554000 --> 0x10102464c457f
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x555555555050 <__stack_chk_fail@plt>:       jmp    QWORD PTR [rip+0x2f6a]        # 0x555555557fc0 <__stack_chk_fail@got.plt>
   0x555555555056 <__stack_chk_fail@plt+6>:     push   0x2
   0x55555555505b <__stack_chk_fail@plt+11>:    jmp    0x555555555020
=> 0x555555555060 <memcmp@plt>: jmp    QWORD PTR [rip+0x2f62]        # 0x555555557fc8 <memcmp@got.plt>
 | 0x555555555066 <memcmp@plt+6>:       push   0x3
 | 0x55555555506b <memcmp@plt+11>:      jmp    0x555555555020
 | 0x555555555070 <fgets@plt>:  jmp    QWORD PTR [rip+0x2f5a]        # 0x555555557fd0 <fgets@got.plt>
 | 0x555555555076 <fgets@plt+6>:        push   0x4
 |->   0x7ffff7f24c00 <__memcmp_avx2_movbe>:    endbr64
       0x7ffff7f24c04 <__memcmp_avx2_movbe+4>:  cmp    rdx,0x20
       0x7ffff7f24c08 <__memcmp_avx2_movbe+8>:  jb     0x7ffff7f24ee0 <__memcmp_avx2_movbe+736>
       0x7ffff7f24c0e <__memcmp_avx2_movbe+14>: vmovdqu ymm1,YMMWORD PTR [rsi]
                                                                  JUMP is taken
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdfa8 --> 0x5555555552ff (<main+390>:    test   eax,eax)
0008| 0x7fffffffdfb0 --> 0x7fffffffe1f8 --> 0x7fffffffe462 ("/home/virus/reverse/PicoCTF/LetsDynamic/chall")
0016| 0x7fffffffdfb8 --> 0x100002000
0024| 0x7fffffffdfc0 --> 0x8000
0032| 0x7fffffffdfc8 --> 0x3100000040 ('@')
0040| 0x7fffffffdfd0 ("picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}")
0048| 0x7fffffffdfd8 ("dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}")
0056| 0x7fffffffdfe0 ("4n4ly1s_1s_5up3r_us3ful_14bfa700}")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 4, 0x0000555555555060 in memcmp@plt ()

```

To make it more understandable:

```asm
gdb-peda$ x/s $rcx
0x7fffffffdfd0: "picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}"
gdb-peda$ x/s $rax
0x7fffffffe010: "aaaaa\n"
gdb-peda$

```

`rax` ---> rax is the 64-bit, "long" size register.
`rcx` ---> Typical scratch register.  Some instructions also use it as a counter.


## The flag

Wohoee!! you did it !! we can get the flag by reading the registers.

The flag `picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}`

I hope you had fun with this challenge. More write-ups/blogs will be written in this repository. Thank you so much for spending your valuable time to read this write-up. 

Ahmet | Reverse-Engineer | Binary-Exploitation | Mathematician

![image](https://user-images.githubusercontent.com/116346668/212536288-1bf3e23a-b326-4ed9-a195-9a61b01e6ab2.png)
