---
title: "JVM"
date: 2022-02-28T08:36:14Z
draft: false
tags: ["java","jvm"]
categories: ["note"]
---

## Java Memory

### Runtime Data Field

Program Counter Register. Thread private. Save current executed bytecode address.  If program is executing Java method, the counter record virtual machine bytecode address. Else if *native* method, the counter's value is *Undefined*.

Java Virtual Machine Stack. Thread private. Every time executing a *method*, JVM create a **Stack Frame** saving local variable table, operator number stack, dynamic link, method out, and so on. Local variable table save JVM basic data type in compile time, object reference. If thread request stack depth larger than virtual machine permitted, throw StackOverflowError; If JVM stack can expend dynamically and can't apply more memory, throw OutOfMemoryError.

Native Method Stack. Like JVM Stack, but serve for native method. HotSpot combine JVM Stack with Native Method Stack.

Java Heap. It allocate memory for almost all objects in Java. It's alias is GC heap (Garbage Collected Heap).

Method Area. Thread shared. It saves type info, constant, static variable, code cache, etc. If method area can't satisfy need for memory, throw OutOfMemoryError.

Runtime Constant Pool. A part of Method Area.

Direct Memory. Not a part of JVM runtime data. Java NIO. Allocate memory out of heap.

### Object

Object Memory Allocation Method:

1. Bump The Pointer
2. Free List
3. Thread Local Allocation

Object Header saves meta-data,hash code,GC age, etc.

memory layout: *Header*, *Instance Data*, *Padding*.

Header includes two types, object runtime data and type pointer.

Instance data save object's real data.

Access object: handle pointer and direct pointer.

## Garbage Collection

### Determining whether an object is alive or not

Reference Counting. (not in java)

Reachability Analysis.(Java C#)

GC Roots include

Fixed as GC Roots:

- Objects in vm stack. Parameters, local variables, temporary variables.
- static reference object in method area.
- constant reference object in method area. Reference in String Table.
- JNI in local method stack.
- Reference in JVM. Basic data types' Class Object, permanent Exception Object, and system class loader.
- Object held by synchronized.
- Callbacks register in JMXBean, JVMTI, local code cache.

And other temporary object according to GC and memory area.

#### Reference

- Strongly Reference
- Soft Reference
- Weak Reference
- Phantom Reference

#### Alive or not

method `finalize()` (not recommended)

#### Collect method area

Deprecated constants and Type no longer in use.

Judge if a type is no longer in use:

- The class's all examples are collected.
- The class's loader is collected.
- The java.lang.Class object for the class counterpart isn't referenced.

### GC Algorithms

#### Generational Collection Theory

1. Weak Generational Hypothesis: The vast majority of objects are born and die.
2. Strong Generational Hypothesis: The more times an object survives the rubbish collection process, the more difficult it is to die.
3. Intergenerational Reference Hypothesis: Compared to same generation reference, Intergenerational reference is only a minority.

Consistent design principles: Collector should divide heap into different areas. And assign objects to different areas according to their age.

In commercial JVM, designer usually divide heap into at least two areas, Young Generation and Old Generation.

Partial GC:

- Minor GC/Young GC
- Major GC/Old GC
- Mixed GC

FUll GC: collect all the heap

#### Algorithms

**Mark-Sweep** Mark all need collect or alive objects, then collect objects. It's the most basic collection algorithms. It has two main disadvantages: First it's execution efficiency is unstable, Second is memory fragmentation.

**Mark-Copy** Disadvantage is only half memory is usable. Most of commercial JVM use this algorithm to collect young generation. HotSpot vm's Serial\ParNew young collector use a kind of more efficient method, "Appel Collection". This method divide young generation into a big Eden space, and two small Survivor space. When collecting, copy alive objects in Eden and one of Survivors to other survivor. The ratio of Eden and Survivor is 8:1. So the available memory is 90%.Object priority allocation in Eden.

**Mark-Compact** Used in Old generation.

### HotSpot Algorithms Details Implement

#### GC Roots Enumeration

By now every collector need pause user threads when GC roots enumeration.

#### Safepoint

## Class File Structure

Class file is a binary stream based on 8 bytes.

Class file is a structure like C language, which has only two data types: *unsigned number* and *table*.

- Unsigned number is basic data type, u1 u2 u4 u8 express 1 byte, 2 byte2, 4 bytes, 8 bytes. Unsigned number can be used to describe number, index, count, and build string as UTF-8.
- Table is composed by unsigned number or other table. All tables are named ending with "_info".