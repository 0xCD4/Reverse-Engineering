# ARM-Assembly-2 

Welcome back to my write-up by solving Pico CTF's RE challenge called ARMassembly-2.


First of all, in order to analyze the file, we are going to use *vim* editor, but you can use whatever editor you want.
You should check my write-up of ARMassembly-0 to understand this solution.

## Function-Analysis

```asm
func1:
        sub     sp,   sp, #32
        str     w0,  [sp, 12]  
        str     wzr, [sp, 24] 
        str     wzr, [sp, 28] 
        b       .L2
```

Everything looks same is it not ?, but there is something new called `wzr`. Let me do the explanation briefly:

The zero register (WZR/XZR) is used for a few encoding tricks. For example, there is no plain multiply encoding, just multiply-add. The instruction MUL W0, W1, W2 is identical to MADD W0, W1, W2, WZR which uses the zero register.

Thus it holds 0 register. Now you can write as follows:

```
*(stack+12) = 4189673334
*(stack+24) = 0
*(stack+28) = 0
```
We can solve this equatiuon mathematically, do not we ?


After this process we can see that `b`--> `branche` is being used which is in assembly x86 (jump) so it will be jumping to `.L2`

## L2 Function

```asm
ldr     w1, [sp, 28]  
ldr     w0, [sp, 12]  
cmp     w1, w0        
bcc     .L3
ldr     w0, [sp, 24]  
add     sp, sp, 32
ret
```

We can see that `0 loads to 1` and `our user's input loads W0` and now it compares. We can say; if loop

```c
if(!(w0<w1))
    L3()
```
After understanding this function it will be easy to understand `L3` lets dive into function `L3`




## L3 function

```asm
ldr     w0, [sp, 24]  ;w0 = *(stack+24)
add     w0, w0, 3     ;*(stack+24) = *(stack+24) + 3
str     w0, [sp, 24]
ldr     w0, [sp, 28]
add     w0, w0, 1     ;*(stack+28) = *(stack+28) + 1
str     w0, [sp, 28]
```
I have specifically added comments to make it understandable for the beginners. Let me explain the solution progress..

The loop tells us that the condition should be true to be returned to the main function, so as to make this true we need to understand `L3` 

L3 function tells us:

`Load the value from stack + 24 into w0 and add 3 to it. Store this value in stack + 24. Next load the value from stack + 28 into w0` 
`Add a1 to it. Store this value in stack + 28. And we will do this, until (stack + 28 > stack + 12).`

We can experiment that, we can do a multiplication process to the user's inputted value ` 4189673334` or you can just add 3 constantly to the added value by the user.

## Solution 

We are knowing that we need to do a multiplication to the added value by the user. In order to get the right answer need to do few things to be completed.

```
1. 4189673334 x 3 = 12,569,020,002
2. Translate this answer to hex.
3. we can get the right result if you calculate scientifically.
```

The flag: `PicoCTF{ED2C0662}`




