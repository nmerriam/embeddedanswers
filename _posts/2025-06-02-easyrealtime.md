---
layout: post
title:  "Why hard real-time is easy"
author: nick
categories: [ C, tutorial ]
# tags: [red, yellow]
image: assets/images/easyTime.png
description: "Why hard real-time is easy"
featured: true
hidden: true
comments: false
---

Hard real-time means software running in a system that only functions correctly
if results are delivered before hard deadlines. In a hard real-time system, every
action has a specified timing behavior, typically expressed as  when an event may be
recognized and the deadline for the reaction to that event.

For example, we might
require that our system respond to a given received CAN message by sending a CAN
message in reply within 10ms, and that our system will meet this deadline when there
is at least a 20ms separation between them. And if the incoming messages arrive too
frequently, messages will either be ignored completely, or the response will be
within 10ms.

## Composition

Hard real-time behavior is relatively easy to de-compose into component parts and
re-compose into larger sub-systems because all the timing is clearly defined. In
our CAN reaction example, we might break down the deadline into
- 2ms for recognizing the incoming message, computing the result and queuing it for
  transmission, and
- 8ms for transmission on the CAN bus.
In turn, we can take the 8ms CAN bus deadline and break it down into the time for
actually transmitting the message and the worst-case queuing delay due to higher
priority messages, plus the blocking from one lower priority CAN message that might
already have started when we queue our outgoing message.

As long as we are working in the domain of hard guarantees (and worst-case behavior),
the times are simply divided up on de-composition, and added together on
re-composition. This is far easier than working with a soft real-time system, where
the timing behavior has no hard deadlines and can only be modelled with some kind
of probability. Combining these probabilities is far less straightforward, not least
because assumptions of independence are fraught with difficulty.

In a system with no timing specification, we nevertheless require time-outs to
allow the system to fail gracefully, for example in the event of a network drop-out.
It is impossible to know how long to make the time-out. Empirical observations and
some guesswork must be done by the developers. The result is that
time-outs get longer and longer, to try to avoid false positives when the system was
only running a bit slowly, and not actually failing. And then the system takes a
very long time to detect and respond to actual failure.

## Debugging

Debugging the timing behavior of a hard real-time system is relatively simple,
assuming that adequate timing measurement is available. Either a task exceeds its
budget or it does not. Either an interrupt arrives too frequently or it does not.
We can quickly localize deviations from the timing specification.

It might not be
easy to repair the timing issue. In some cases, a change to the specification may be
appropriate. Consider a task that has a budget of 4ms. This means that each time the
task is required to respond to an event, it can use 4ms of CPU time. Imagine that
the task comprises of 2 modules, each of which is given a budget of 2ms. It might
be the case that, when debugging, we observe that one module requires at most 1ms,
while the other module regularly exceeds 2ms, and sometimes exceeds 3ms. Optimizing
that module to run with about half the CPU time might be very expensive. And we
might choose to change the specification, to limit the first module to 1ms CPU time,
and to give the second module a budget of 3ms.

Return to the timing requirements might seem a bit unwieldy, but imagine
how much simpler this is than attempting to debug a system without timing
requirements, where we cannot even clearly identify what is correct, compared with
what is too slow or too fast.

