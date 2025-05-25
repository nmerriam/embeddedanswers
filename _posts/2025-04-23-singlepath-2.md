---
layout: post
title:  "Single-Path Code (2)"
author: nick
categories: [ C, tutorial ]
# tags: [red, yellow]
image: assets/images/singlePath1.png
description: "Single-path code (2)"
featured: true
hidden: true
comments: false
---

Single-path code, or branchless code, has been an important development in
performance computing. It turns out to be even more interesting in the
domain of hard real-time, embedded systems. In this second of three articles,
let us see what modern compilers do to reduce the number of taken branches
on a given code path.

## Saturation

In this first example, we will see how a compiler can find optimal,
branchless code with no tweaking of the source code.

Consider a simple function that replaces negative values with zero,
cf. [saturation](https://en.wikipedia.org/wiki/Saturation_arithmetic).

```c
int NoNegative1( int x )
{
    if( x < 0 )
    {
        x = 0;
    }
    return x;
}
```

There is a well-known, single-path implementation, where we take the 2s-complement
sign bit and use that to create a mask that, when inverted, generates a zero from
any negative input. Rewriting NoNegative with this approach, we get the following.

```c
int NoNegative2( int x )
{
    x &= ~( x >> 31 );
    return x;
}
```

We might well expect that NoNegative2 compiles to branchless code. It turns
out that also NoNegative1 compiles to branchless code, with the latest
Arm CLANG compiler. In fact, they [compile to exactly the same, single
instruction](https://godbolt.org/z/cP9devdKx).

This underlines the message of the
[Graphics Programming Black Book](https://github.com/jagregory/abrash-black-book)
by Michael Abrash, always
to check our expectations with actual measurement. I think that most
programmers would agree that the meaning of NoNegative1 is easier to
understand and check, from reading the source code, compared with
NoNegative2.

## Symmetric Saturation

In the second example, we will see how the branchless paradigm can be applied not
only with a single condition, but also naturally extended to multiple conditions.

```c
int SignedSaturate1( int x )
{
    if( x < -127 )     x = -127; // Lower limit
    else if( 127 < x ) x =  127; // Upper limit
    return x;
}
```

This time, there is no simple transformation to unconditional code. Rather, the
compiler uses [conditional instructions](https://godbolt.org/z/nhPbxsG6j).
These always execute but their behavior
depends on a condition, typically captured in a condition flag. You are probably
familiar with conditional branch instructions, and other instructions can be made
conditional in the same way. The Arm A32 instruction set made all instructions
available either as unconditional or conditional instructions. In fact, the
unconditional instructions have the same opcode, but with the condition field set to
select an unconditional "condition".

Other instruction sets (for example Infineon TriCore and PowerPC VLE) include
conditional instruction sets, after the value of conditional instructions and
branchless code was proven with Arm architectures. The Arm T32 (Thumb2) allows
any instruction to be made conditional using an IT instruction prefix.
Interestingly, the Arm A64 instruction set removes the ability to make any instruction
conditional. It turns out that making every single instruction conditional gives a
superscalar pipeline instruction scheduler a hard time, so only the conditional
instructions most useful for branchless code are implemented in A64.

If you are familiar with the Arm instruction set, or the TriCore instruction set,
you might well be thinking that the compiler should be using the dedicated saturate
instruction. That will saturate to the range [-128..127] but then we have only one
value to correct, as -128 would need to become -127. In the next article, we will
look at exactly this question.
