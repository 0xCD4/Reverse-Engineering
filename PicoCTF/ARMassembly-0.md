# Reverse ARMassembly-0 

Hello back again me!! Today we are going to be reversing ARM-Assembly. I should say that; I am not so familiar with ARM but with the help of documentation, we will solve it.
There will be a serie about these concepts, thus stay tuned!!



First of all, let me read the challenge.

Description:
```

What integer does this program print with arguments 182476535 and 3742084308? File:
chall.S Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})
```

There are two integers: 
1. 182476535
2. 3742084308

Awesome!

You can check this challenge on [PicoCTF-ARM](https://play.picoctf.org/practice/challenge/160?category=3&page=1&search=ARM)


## Analyzing the file

We can wget this file to my terminal. In order to analyze this file we can use vim or other text editor, and we know that this file was written in Assembly.

Let me open the file to be analyzed:

```asm
main:
        stp     x29, x30, [sp, -48]!
        add     x29, sp, 0
        str     x19, [sp, 16]
        str     w0, [x29, 44]
        str     x1, [x29, 32]
        ldr     x0, [x29, 32]
        add     x0, x0, 8
        ldr     x0, [x0]
        bl      atoi
        mov     w19, w0
        ldr     x0, [x29, 32]
        add     x0, x0, 16
        ldr     x0, [x0]
        bl      atoi
        mov     w1, w0       # W1 ---> 182476535
        mov     w0, w19      # w0 ---> 3742084308
        bl      func1
        mov     w1, w0
        adrp    x0, .LC0
        add     x0, x0, :lo12:.LC0
        bl      printf
        mov     w0, 0
        ldr     x19, [sp, 16]
        ldp     x29, x30, [sp], 48
        ret

```
Luckily, we have main function which makes it easier for us to analyze the instructions even though, we know there were given two integers. 

I had already commented two integers.

```
In the reverse order the integers were already given.
```

```asm
mov w1, w0  ; the first integer has been thrown to w1
mov w0, w19 ; the second intger has been thrown to w0
```

Awesome! Now, we are seeing that after the instruction being called 

```asm
bl, func ; bl ---> branch and link

```

Let me analyze func1!


```asm
func1:
        sub     sp, sp, #16
        str     w0, [sp, 12]
        str     w1, [sp, 8]
        ldr     w1, [sp, 12]
        ldr     w0, [sp, 8]
        cmp     w1, w0
        bls     .L2
        ldr     w0, [sp, 12]
        b       .L3
.L2:
        ldr     w0, [sp, 8]
.L3:
        add     sp, sp, 16
        ret

```

```
sp --> SP (or R13) is the stack pointer
```


Let me analyze these instructions ASM

```asm
 str     w0, [sp, 12]  ;W0 --> 3742084308
 str     w1, [sp, 8]   ;W1 --> 182476535
 ldr     w1, [sp, 12]
 ldr     w0, [sp, 8]
 cmp     w1, w0
 bls     .L2
 ldr     w0, [sp, 12]
```

We know that *W0* is the input integer from the uer *str* --> store , thus this means that 12 bytes offset is stored to *W0*. It also stores 8 bytes to *W1*
After the instruction we can see that *ldr* ---> Load a data in specified address (label) into register. `It loads 12 bytes to *W1*, and it loads 8 bytes to *W0*` when the process is finished
it compares between two integers. Comparison between  *W1* and *W0*

Do not forget!!

`load the value at offset 12 on the stack into w1` is equal `to ldr w1, [sp, 12]`
`load the value at offset 8  on the stack into w0` is equal `to ldr w0, [sp,8]`

If you are familiar with `x32/86` Assembly, you just need to research this!


There is another function called `L2` let me overview this in my mind. This function will give us the solution.

```asm
.L2:
        ldr     w0, [sp, 8] ; ldr --> 	Load a data in specified address (label) into register
.L3:
        add     sp, sp, 16
        ret

```


At `.L2` we load a value from the stack at `offset 8` into the variable w0 and continue execution back in func1. In .L3 we just add `16 back to sp`,
ie. we fill the stack back again. 

When we do a comparison we can see that the greatest value shall be called, but let me check...

```asm
 bls     .L2 ; Branch on Lower than or Same 
 ldr      w0, [sp, 12] 
 b       .L3 ; b --> branch(moves in x86 assembly )
```
It branches .L3 as return and it filles 16 bytes as offset.


W0 will be loaded and printed out!!

W0 --- > 3742084308, we need to convert this in 32 bits hex as we read before.


We also should denote that `32 bits` We can use bit python knowledge.

```python

value = 3742084308
print(f"The answer is 0x8{value}")

```

The output:

```
The answer is 0xdf0bacd4
```

You should consider that the answer should be given as non 0x ---> picoCTF{df0bacd4}



Thank you so much for reading this write up. It was great to learn new stuff.

Seeyaa



