# Easy RE 

Hello everyone, 

welcome back to my blog-post about reversing an easy RE challenge. I will share my RE write-ups in this repository.

In order to get useful information of the file, we need to use some useful tools to analyze the architecture etc..

Let me get started with *file* command.

```bash
remnux@remnux:~/binary-exploit/crackme$ file gugus.out 
gugus.out: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=0f8486979b8d039a21132108fe86f4ed982d070d, for GNU/Linux 3.2.0, not stripped

```

We are seeing that the file has not been stripped also its 32-BIT. We are going to run the file to see what the output will be.

```bash
remnux@remnux:~/binary-exploit/crackme$ ./gugus.out aaa
...Good morning...
Sorry....
Try again

```  

hmm it seems that we input wrong password. We do not give up , i am willing to use *ltrace* command.

What does ltrace command do?

You can surely search with a bit google tricks, but let me shortly introduce what it does.
```
ltrace is a program that simply runs the specified command until
it exits.  It intercepts and records the dynamic library calls
which are called by the executed process and the signals which
are received by that process.  It can also intercept and print
the system calls executed by the program.

```

Thus, it will be useful to use ltrace command to see the functions being used.

```bash 
remnux@remnux:~/binary-exploit/crackme$ ltrace ./gugus.out aaa
__libc_start_main(0x5659e1bd, 2, 0xffb25894, 0x5659e2a0 <unfinished ...>
puts("...Good morning..."...Good morning...
)                                                                    = 19
strcmp("gu!gu?s", "aaa")                                                                      = 1
puts("Sorry...."Sorry....
)                                                                             = 10
puts("Try again"Try again
)                                                                             = 10
+++ exited (status 0) +++

```

You can also use *string* command to see the functions being used luckily. If it was being stripped, we could not see the functions and syscalls.



```bash

remnux@remnux:~/binary-exploit/crackme$ strings gugus.out 
td` 
/lib/ld-linux.so.2
_IO_stdin_used
exit
puts
__cxa_finalize
strcmp
__libc_start_main
libc.so.6
GLIBC_2.1.3
GLIBC_2.0
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
gu!g
Y[^]
[^_]
missing argument
...Good morning...
Access granted
Congratulation
Sorry....
Try again
;*2$"
GCC: (Debian 11.2.0-10) 11.2.0
Scrt1.o

```

Awesome! *strcmp* is being used while coding this file. I am going to check once whether this password is correct or not.

```
gu!g
Y[^]
[^_]

```

The password *gu!gu?s*

Let me give a try:

```bash
remnux@remnux:~/binary-exploit/crackme$ ./gugus.out gu!gu?s
bash: !gu?s: event not found
```

hmm event not found...? 

It seems we need to escape special characters with *\* You can get more information from stackoverflow [stackoverflow-escape]{https://stackoverflow.com/questions/9590838/python-escape-special-characters-in-sys-argv}

I almost forget to say that; Our input is not taken by scanf but sys.argv thus one of the reason that it could not find the event.


I will try with *\*:

```bash
remnux@remnux:~/binary-exploit/crackme$ ltrace ./gugus.out gu\!\gu\?\s
__libc_start_main(0x566421bd, 2, 0xff8e67b4, 0x566422a0 <unfinished ...>
puts("...Good morning..."...Good morning...
)                                                                    = 19
strcmp("gu!gu?s", "gu!gu?s")                                                                  = 0
puts("Access granted"Access granted
)                                                                        = 15
puts("Congratulation"Congratulation
)                                                                        = 15
+++ exited (status 0) +++
remnux@remnux:~/binary-exploit/crackme$ ./gugus.out gu\!\gu\?\s
...Good morning...
Access granted
Congratulation

```

You can also check which syscalls have been used:

```assembly
   0x00001223 <+102>:	add    eax,0x4
   0x00001226 <+105>:	mov    eax,DWORD PTR [eax]
   0x00001228 <+107>:	sub    esp,0x8
   0x0000122b <+110>:	push   eax
   0x0000122c <+111>:	lea    eax,[ebp-0x2b]
   0x0000122f <+114>:	push   eax
   0x00001230 <+115>:	call   0x1030 <strcmp@plt>
   0x00001235 <+120>:	add    esp,0x10
   0x00001238 <+123>:	mov    DWORD PTR [ebp-0x1c],eax
   0x0000123b <+126>:	cmp    DWORD PTR [ebp-0x1c],0x0
   0x0000123f <+130>:	jne    0x1267 <main+170>
   0x00001241 <+132>:	sub    esp,0xc
   0x00001244 <+135>:	lea    eax,[ebx-0x1fd3]
```

We have successfully reversed this file. This challenge was enough to understand how to debug and use ltrace command. The most important part was to escape the special symbolic characters.

Thank you for spending your valuable time to read this blog. More RE challenges will be written (easy,medium,hard). See you soon!!!    
