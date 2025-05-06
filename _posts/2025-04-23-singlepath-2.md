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
to check our expectations with actual measurement.

## Symmetric Saturation

In the second example, we will see how a small change to the source code
helps the compiler to generate better, branchless code.

The ability of a compiler to surprise the user of a high-level programming
language is emphasized again by the following example. This is based
on a real example, where the control algorithms required that results of
integer arithmetic be saturated to symmetric ranges, such as -15 to +15,
or -127 to 127. Always the limit was two to the power of n minus one. I asked
why they had not exploited the Arm SSAT instruction, and they patiently
explained that SSAT performs an asymmetric saturation, for example in the
range -128 to +127. And that this made it impossible to use SSAT, they claimed.
Rather, they wrote code like SignedSaturate1.

```
int SignedSaturate1( int x )
{
    if( x < -127 )     x = -127; // Lower limit
    else if( 127 < x ) x =  127; // Upper limit
    return x;
}
```

The current Arm CLANG compiler generates branchless code from this
example, first using a conditional instruction to truncate on the
lower limit, and then a second conditional instruction to truncate on
the upper limit.

But this overlooks the fact that SSAT does nearly the correct
operation. Only one value needs to be corrected. And the compiler
recognizes when C code corresponds to SSAT, so that the following,
longer, C code actually generates
[one instruction fewer](https://godbolt.org/z/3j49bqzE6). And one of
those instructions is unnecessary and would most likely be elided when the
function is inlined, as was the case in the real project.

```
int SignedSaturate2( int x )
{
    if( x < -128 )     x = -128; // Lower limit minus 1
    else if( 127 < x ) x =  127; // Upper limit
    if( -128 == x )    x = -127; // Correct lower limit
    return x;
}
```

## Likelihood hints




```
int Func1( int x );

int Func2( int x )
{
    if( x < 0 )
    {
        x *= (x / 2);
    }
    else
    {
        x = Func1( x );
    }

    x = Func1( x + 1 );
    x = Func1( x + 2 );

    return x;
}
```

```
#define LIKELY( condition_ )   ( __builtin_expect( ( condition_ ), 1 ) )
#define UNLIKELY( condition_ ) ( __builtin_expect( ( condition_ ), 0 ) )

int Func1( int x );

int Func3( int x )
{
    if( UNLIKELY( x < 0 ) )
    {
        x *= (x / 2);
    }
    else
    {
        x = Func1( x );
    }

    x = Func1( x + 1 );
    x = Func1( x + 2 );

    return x;
}
```