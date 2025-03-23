---
layout: post
title:  "Constant Joy (1)"
author: Nick
categories: [ C, tutorial ]
# tags: [red, yellow]
image: assets/images/11.jpg
description: "What is means to apply const to a global variable in C."
featured: true
hidden: true
comments: false
rating: 3
---

The const keyword in C is a useful tool with special relevance to embedded software. It has different meanings in different contexts. Even experienced C programmers are often unclear about these different meanings, each of which I will describe in a short article. This is the first, and addresses const applied to a global or static variable outside any function scope.

A global variable is defined with a type and, optionally an initial value:
```c
int var = 1;
```
If that variable is intended to never change, and always to hold that initial value, we can add the const decorator like this:
```c
int const var = 1;
```

This has three important effects:
1. The compiler will raise an error if you try to write to ‘var’. This can save a lot of time, compared with debugging to find an unintended change to ‘var’.
2. The human reader of the code can see that this variable does not change. Integrated development environments (IDEs) can show the declaration, additionally, at every reference to ‘var’. And a naming convention might also be used. We can prefix global constants with ‘c_’, to make ‘c_var’, for example.
3. In an embedded environment, RAM is often scarce and it is common to read const variables direct from read-only (flash) memory without copying them to RAM at start-up. This is critical for allowing complex software, such as vehicle engine management, on a cost-effective processor. The same technique is used with code, which is also commonly executed direct from flash.

This last effect can hugely reduce RAM consumption in embedded software but there is a performance penalty on cache misses. Flash memory can be made to perform very well for sequential accesses, which are common when executing code. But flash memory is very slow for random accesses, which is what we are likely to get with const variables. The effect is even worse when the flash accesses collide with instruction fetches from the same flash. Therefore software typically performs better when well-chosen const variables are located in data RAM. This could be achieved by just removing the const qualifier, but then we lose the first two advantages. A better option is to place performance-relevant const variables in a different section and to locate that section in RAM. The memory protection unit (MPU) can be used to ensure that writes only occur when the start-up code initialises that RAM.

Even though I know of this effect, I continue to be surprised by the performance improvement achieved in real, embedded software by locating key const variables in RAM. For the modest effort required to identify these variables and add the relevant section pragma or attribute, I have seen around 2% reduction in CPU load. The same result could require ten times as much effort with generic, code-based optimization.

In the next article, I will look at applying the const keyword to variables within function scope.
