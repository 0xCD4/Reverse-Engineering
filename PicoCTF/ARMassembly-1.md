# ARMassembly-1

Hey everyone, welcome back to my write-up! Today, i am going to be writing a blog about the solution of ARMassembly-1 on PicoCTF. If you are ready lets get started.

Description:
```
For what argument does this program print `win` with variables 79, 7 and 3? File: chall_1.S Flag format: 
picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})
```

First of all, let me analyze the file properly.


## Analyze the file

```asm
main:
        stp     x29, x30, [sp, -48]!
        add     x29, sp, 0
        str     w0, [x29, 28]
        str     x1, [x29, 16]
        ldr     x0, [x29, 16]
        add     x0, x0, 8
        ldr     x0, [x0]
        bl      atoi
        str     w0, [x29, 44]
        ldr     w0, [x29, 44]
        bl      func
        cmp     w0, 0
        bne     .L4
        adrp    x0, .LC0
        add     x0, x0, :lo12:.LC0
        bl      puts
        b       .L6
.L4:
        adrp    x0, .LC1
        add     x0, x0, :lo12:.LC1
        bl      puts
.L6:
        nop
        ldp     x29, x30, [sp], 48
        ret
        .size   main, .-main
        .ident  "GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
        .section        .note.GNU-stack,"",@progbits

```

As usual, we are seeing the main function which will be important for us.

*atoi* --> atoi is a function in the C programming language that converts a string into an integer numerical representation. atoi stands for ASCII to integer.


Let me analyze the funtion step by step:

```asm

func:
        sub     sp, sp, #32
        str     w0, [sp, 12]
        mov     w0, 79
        str     w0, [sp, 16] # *(stack+16) = 79
        mov     w0, 7
        str     w0, [sp, 20] # *(stack+20) = 7
        mov     w0, 3
        str     w0, [sp, 24] # *(stack+24) = 3
        ldr     w0, [sp, 20] # w0 = 7
        ldr     w1, [sp, 16] # w1 = 79
        lsl     w0, w1, w0   # w0, 79, 7 79 << 7
        str     w0, [sp, 28] # *(stack+28) = 10112
        ldr     w1, [sp, 28] # *(stack+28) = 10112
        ldr     w0, [sp, 24] # *(stack+24) = 3
        sdiv    w0, w1, w0   # w0, 10112, 3 10112//3
        str     w0, [sp, 28] #  *(stack+28) = 3370
        ldr     w1, [sp, 28] # w1 = 3370
        ldr     w0, [sp, 12] # w0 = x
        sub     w0, w1, w0   # w0 = 3370-x
        str     w0, [sp, 28]
        ldr     w0, [sp, 28]
        add     sp, sp, 32
        ret


```
You can see that the comments are being added next to the instructions respectively.


```asm
str     w0, [sp, 12]
mov     w0, 79
str     w0, [sp, 16] # *(stack+16) = 79
mov     w0, 7
str     w0, [sp, 20] # *(stack+20) = 7
mov     w0, 3
str     w0, [sp, 24] # *(stack+24) = 3
ldr     w0, [sp, 20] # w0 = 7
ldr     w1, [sp, 16] # w1 = 79
lsl     w0, w1, w0   # w0, 79, 7 79 << 7

```

So first we make space on the stack for the variables. Our user input is stored on the stack at offset 12, next the mov instruction is called.


```asm
str     w0, [sp, 12]
mov     w0, 79
str     w0, [sp, 16] # *(stack+16) = 79
mov     w0, 7
str     w0, [sp, 20] # *(stack+20) = 7
mov     w0, 3
str     w0, [sp, 24] # *(stack+24) = 3
```
We can understand from this code:

```
w0 - the input lets stay x = stack+12
stack+16 = 79
stack+20 = 7
stack+24 = 3
```

You can also think mathematically, every definition has its own declaration!

Now, lets move forward to the next instruction.


```asm
lsl     w0, w1, w0   # w0, 79, 7 79 << 7
str     w0, [sp, 28] # *(stack+28) = 10112
ldr     w1, [sp, 28] # *(stack+28) = 10112
ldr     w0, [sp, 24] # *(stack+24) = 3
sdiv    w0, w1, w0   # w0, 10112, 3 10112//3
str     w0, [sp, 28] #  *(stack+28) = 3370

```

This piece of code shall be important for us, because this will bring us the solution at all. Now, let me explain these steps by one-one


We need to understand what *lsl* is --> LSL is a logical shift left by 0 to 31 places. The vacated bits at the least significant end of the word are filled with zeros

```
w0->7
w1->79
```

This means that we are able to do our process.

```
w0-> 79 << 7 
```

The result:

```
>>> 79 << 7
10112
thus, *(stack+28)  will be stored to 10112
```
and we also know that:

```
w0 -> *(stack+24) = 3
```


## The final part 

Oke we have analyzed the file correctly, and evertyhing is being written well. We can now use our written note to solve the question properly.


```asm
sdiv    w0, w1, w0   # w0, 10112, 3 10112//3
str     w0, [sp, 28] #  *(stack+28) = 3370
```

sdiv means *Signed Divide and Unsigned Divide.* we can divide two integers together. We should consider that w1 -> 10112, and w0 -> 3. 


```
10112//3 = 3370 this will be stored to w0, and [stack+28] will be equal to 3370

```

The final part of this function:

```asm
 ldr     w1, [sp, 28] # w1 = 3370
 ldr     w0, [sp, 12] # w0 = x
 sub     w0, w1, w0   # w0 = 3370-x
 str     w0, [sp, 28]

```

ldr means in assembly ->  Load a data in specified address (label) into register. We do not anything about the user's input we can call as *x* (stack+12 = x)

we need to substract from 3370.

the final result should be 3370-x


```asm

cmp     w0, 0
bne     .L4
adrp    x0, .LC0
```

```asm
.LC0:
        .string "You win!"
        .align  3
```

we can understand from this code; if by comparison be 0 then call *adrp* with the call of x0, but when it does not then bne and this means --> (bne stands for Branch if Not Equal)
That is call of *You wind* so as to complete the challenge we should say that '3370-x' must be equal to 0


```
3370-x = 0 
x = 3370
```

In order to put the answer correctly, we should convert to 32 bits

3370 --> 00000D2A

THE FLAG : ```picoCTF{00000D2A}```

Thank you so much for reading this write-up. More challenges will be appeared stay tuned!!!

![image](https://user-images.githubusercontent.com/116346668/210439356-de787a95-2d2f-4749-8e6d-b0a3e394bb72.png)
