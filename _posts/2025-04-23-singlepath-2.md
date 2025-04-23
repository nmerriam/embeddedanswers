---
layout: post
title:  "Single-Path Code (1)"
author: Nick
categories: [ C, tutorial ]
# tags: [red, yellow]
image: assets/images/singlePath1.png
description: "Single-path code (1)"
featured: true
hidden: true
comments: false
---

Single-path code, or branchless code, has been an important development in
performance computing. The hardware acceleration features of modern
processors are optimized for code without branches. The smooth flow
of pre-fetching instructions to cache and pre-fecthing their arguments
to registers is disrupted by branches. Good compilers try hard to
genereate fewer branches, especially as you request that optimization
is more focuseed on speed. In addition, the way that you write your
source code can have a big impact on the number of executed branches.

In a series of three short articles, I will explain
- why branchless code is also important in embedded software,
  especially in hard real-time systems,
- how compilers can generate code with fewer branches from
  your unchanged source code, and
- how source code in a high-level language can be constructed to
  minimize branches.

In addition, single-path code has a special role to play in hard
real-time systems. While there can still be small variations due to
pipeline and cache performance, single-path code has very stable
execution time, which helps to make the timing more predictable.
This benefit can be so important, that it justifies make the code
slower.

And in a safety environment, where
[MC/DC coverage](https://en.wikipedia.org/wiki/Modified_condition/decision_coverage)
is an important
test parameter, single-path code makes a mockery of MC/DC metrics.

## Saturation

Consider a simple function that relaces negative values with zero,
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
sign bit and use that to create a mask. Rewriting NoNegative with this approach,
we get the following.

```c
int NoNegative2( int x )
{
    x &= ~( x >> 31 );
    return x;
}
```

In fact, this implementation is so well-known that a compiler
will often recognize that these are equivalent and generate
the same code for both implementations,
see [https://godbolt.org/z/MKG4de3j6](https://godbolt.org/z/MKG4de3j6).
