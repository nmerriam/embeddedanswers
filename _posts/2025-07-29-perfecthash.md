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
a hard real-time, embedded system. This is probably because its
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
contains at most one command. 

