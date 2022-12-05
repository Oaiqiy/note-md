---
title: "Promela"
date: 2021-12-09T06:41:12Z
draft: false
tags: ["promela","spin","Protocol verification"]
categories: ["class"]
---

PROMELA = *Protocol/Process Meta Language*
SPIN = *Simple Promela Interpreter*

## Overview

Promela programs consist of precesses, message channels, and variables.

- Promela processes execute concurrently.
- Non-deterministic scheduling of the precesses.
- Processes are interleaved.
- All statements atomic.
- Each process may have several different possible actions enable at each point of execution.

## Executing a Promela system

A system state of a Promela system comprises the value of **the global variables, the contents of the channel buffers** and for each process instance,the **values of its location counter, its local variables** and **local channel buffers**.

Initially, *all global (non-initialized) variables* contain the value 0 and the channel buffers are empty.

**The existing process instances** are the  ones created through the **active modifier** on the proctype declarations together with the **init** and **never** processes, if present.

The location counters of these instances are at their first statements.

A system state of a Promela system uniquely identifies the enabled statements in every (active) process instance.

So, process instances execute asynchronously.

In case the selected statement was an *un-buffered send (receive)*, a corresponding *receive (send) statement* was executed simultaneously.

If the system does have a *never claim*, an *execution step* is a combined.

A *never claim* blocks execution of the system. in those situations in which the never claim has no enabled statements (in the current state).

## Executability

In Promela there is no difference between **conditions** and **statements**, even isolated boolean conditions can be used as statements.

The execution of every statement is conditional on its executability.

```pml
    while (a!=b)
        skip
```

equals `(a==b)`

## Promela Model

consist of:

- **type** declarations
- **channel** declarations
- **variable** declarations
- **process** declarations
- \[**init** process]

## Processes

A process type (proctype) consist of

- a **name**
- a list of formal parameters
- local variable declarations
- body

communicate with other processes

- using global variables
- using channels

Processes are **created** using the **run** statement (which returns the process id ).

Processes can be created at **any point** in the execution (within any precess).

Processes start executing **after** the **run** statement.

```pml
init{
    int pid2 = run Foo(2);
    run Foo(27);
}
```

Processes can **also** be created by adding **active** in front of the **proctype** declaration.

```pml
active[3] proctype Bar(){
    ...
}
```

## Variables and Types

Five different (integer)

- bit \[0..1]
- bool \[0..1]
- byte \[0.255]
- short \[-2^16-1..2^16-2]
- int \[-2^32-1..2^32-1]
  
Arrays  
Records(structs)

**Type conflicts** are detected at runtime.

**Default initial value** of basic variables (local and global) is **0**.

## Statements

The body of a process consists of a sequence of **statements**. A statement is either

- **executable**: the statement can be executed **immediately**.
- **blocked**: the statement **cannot** be executed.

An **assignment** is always executable.

An **expression** is also a statement; it is evaluates to **non-zero**.

The **skip** statement is **always executable**.

A **run** statement is **only executable** if a new process can be created.

A **printf** statement is **always executable**.

assert **assert(\<expr>)**

## Mutual Exclusion

## if-statement

```pml
if
:: choice1 -> s1.1; s1.2;
:: choice2 -> s2.1; s2.2;
:: ...
:: choice n -> sn.2; sn.2;
fi;
```

## do-statement

```pml
do
:: choice1 -> state1.1;

...

od;
```

## Communication

function:

- message passing
- rendezvous synchronization (handshake)

`chan <name> = [<dim>] of {<t1>,<t2>,...<tn>};`  
dum: number of elements in the channel

Sending - putting a message into a channel

`ch ! <expr1>, <expr2>, ... <expr n>;`

Receiving - getting a message out of a channel  
`ch ? <var1> ...`  
`ch > <const1> ...`

## Alternating Bit Protocol

- To every message, the **sender** adds a bit.
- The receiver acknowledges each message by sending the received bit back.
- To receiver only excepts messages with a bit that is excepted to receive.
- If the sender is sure that the receiver has correctly received the previous message, it sends a new message and it alternates the accompanying bit.

## Atomic

```pml
atomic {stat1; ...}
```

## d_step

- more efficient version of atomic: no intermediate states are generated and stored.

## timeout

- timeout models a global timeout.
- timeout provides an escape from deadlock states
- beware of statements that are always executable

## goto

go label

## Invariance

[] P where P is a state property
