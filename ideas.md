- Polling latency
- Preemptive v. non-preemtive scheduling
- Base+offset addressing
- Real-time thread-safety
- Floating point, and how to use it in different contexts without lots of expensive register set save/restore
- Pipeline optimization
- Inlining
- Tail-call optimization
- Why hard real-time is easy
- Why hard real-time is difficult



And in a safety environment, where
[MC/DC coverage](https://en.wikipedia.org/wiki/Modified_condition/decision_coverage)
is an important
test parameter, single-path code makes a mockery of MC/DC metrics.

## Saturation

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
