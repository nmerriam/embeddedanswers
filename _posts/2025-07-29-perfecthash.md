---
layout: post
title:  "Perfect hashing"
author: nick
categories: [ C, tutorial ]
# tags: [red, yellow]
image: assets/images/hash.png
description: "Perfect hashing"
featured: true
hidden: true
comments: false
---

Perfect hashing is often overlooked as a way to organize data in
a hard real-time, embedded system. This might be because its
application is quite limited. Nevertheless, it is an extremely
effective solution for a certain class of design problems.

### What is a hash function?

A hash table allows data to be indexed by an arbitrary key. A hash function
maps a key value to a bucket, which contains a number of elements of the
table. A hash table provides good average performance for a look-up,
typically faster than binary search and much faster than linear search.

A hash collision occurs when two keys map to the same bucket. The bucket
must be searched for the correct table element. Having a few such searches
is not harmful to average performance but it creates unpredictable timing,
which can be problematic for a hard real-time system with timing
guarantees. The worst case, of course, is when all the keys map to the
same bucket.

### What is a perfect hash function?

A [perfect hash](https://en.wikipedia.org/wiki/Perfect_hash_function)
is simply a hash table with no collisions. With no collisions,
timing behavior is very predictable, so perfect hashing is
ideally suited to some applications in real-time systems.

## Where is perfect hashing useful?

Creating a small, fast, hash function with no collisions is
typically very expensive. It involves searching the space of
hash function parameters and checking that the current set of keys
give no collisions. Therefore perfect hashing works best for
pre-configured data, where the hash function can be derived
at build time, rather than at run time.

### Decoding command IDs

Suppose our real-time, embedded system receives 32-bit command
IDs and reacts by carrying out the appropriate command. There
are 400 different commands and the IDs are not arranged from
zero to 399, but rather as 400 distinct, pseudo-random values.
This design reduces the chance that an invalid command ID is
interpreted as a real command.

When a command ID is received, the system needs to determine
whether or not it is a valid command ID and, if valid, which
command to execute. Perfect hashing provides a convenient
way to quickly map from the received command ID to a bucket that
contains at most one command. On receipt of a command ID,
the following steps are performed:
- Run the hash function to get the hash value
 - The execution time of the hash function might vary due
   to integer divide instructions varying in the number of
   cycles required, but the worst-case timing can be
   determined.
- Look in the appropriate bucket to check that the valid
  command ID matches the received command ID
 - The execution time for this array look-up will vary due
   to varying memory performance, but the worst-case timing
   can be determined.
In any event, only one fetch from one bucket is needed. This
will typically be from slow flash memory, so having only
one access allows perfect hashing to far outperform linear
or binary search.

## Disadvantages of perfect hashing

### Increased build time

The time required to derive a perfect hash function can be
quite large, depending on the set of keys. If there are a
large number of functions to be determined, this can have
a noticeable effect on the build time. And if the keys are
addresses of symbols from a linked and located ELF file,
we have to link twice. The first link contains only a
placeholder hash function and array of buckets and is
used to determine the address keys. Then we derive the
hash function and bucket array and re-link, checking that
the key addresses do not change. Linking twice can be
a painful delay when building a large system. And if
the second linking changes key addresses, the process can,
in theory, require a third linking, or even more.

### Number of buckets

When searching for hash function parameters, if collisions
cannot be avoided then we are forced to add more buckets
and search again. Apart from taking time to find a solution
without collisions, the end result might require twice as
much storage as would be required for a more conventional
array and search approach.

Our example of 400 32-bit command IDs might increase the
storage from a packed 1600 bytes to 3200 bytes. Consuming
an additional 1600 bytes for the timing benefit might be
a good trade-off. If we had 40000 keys, we might be less
willing to have 160KB of empty buckets.
