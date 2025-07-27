- Polling latency
- Preemptive v. non-preemtive scheduling
- Base+offset addressing
- Real-time thread-safety
- Floating point, and how to use it in different contexts without lots of expensive register set save/restore
- Pipeline optimization
- Inlining
- Tail-call optimization



### Pipeline

Executing one instruction can be broken down into multiple steps. Different
architectures identify different steps, but the following make up a representative
example:
- Fetch instruction
- Decode instruction
- Fetch data from memory (if required)
- Compute instruction
- Write data to memory (if required)

Rather obviously, the processor does not have to wait until one instruction has
finished all of its steps before starting to fetch the next instruction. It is not
entirely straightforward, since branch prediction might fail, and our speculative
fetch of what was predicted to be the next instruction might turn out to be incorrect.
Nevertheless, it is clear that we can make a much faster processor if we can overcome
these challenges and implement a
[pipeline](https://en.wikipedia.org/wiki/Instruction_pipelining).

### Cache

A modern processor can execute instructions faster than they can be fetched
main code memory and faster than data can be fetched from main data memory.
To avoid the processor spending most of its time stalled, waiting for memory,
instruction and data caches can be provided that do not help with the first
fetch but greatly accelerate repeated accesses to the same address. If we
have a loop, the first iteration of the loop might be very slow, but every
subsequent iteration finds the instructions for the loop already in fast
cache memory.

#### Burst Memory Accesses

### Multi-core and DMA

### Branch Prediction
