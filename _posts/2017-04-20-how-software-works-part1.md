---
layout: post
title: "How software works part Ⅰ"
---

My journey in software started with Javascript and Ruby, learning how to write web applications and APIs. Although I was able to write software at the application level I had no understanding of how software _actually works_.

How does the text in my editor end up being executed on the CPU? What are all these low level concepts like stack, heap, registers, CPU caches, etc? What are finite state machines good for?

  There’s a mountain of abstractions that application developers stand on that they may not even know about. This series of blog posts is an attempt to create a primer for this information, diving into the depths of low-level representation and execution of programs.

Some parts of this overview are going to be simplified and there will be gaps in the whole picture. We’ll go bottom-up, starting from number theory and binary representation, all the way to the internals of interpreted languages like Ruby and Javascript. Please note, this series is called 'How software works' not 'How computers work'. I have a very superficial knowledge of computer hardware so, in this series, I will take hardware for granted and stay on the software side of computing.

### 00101010

Everyone knows that software is all ones and zeroes in the end. What does that mean?
How do my classes and methods get translated to binary? What's the big deal with ones and zeroes anyways?

The binary nature of our computers is largely hidden from even fairly low-level programs. When
you get to the lowest level of software, however, you do need to understand binary and why computers represent data using ones and zeroes.

Why encode information in base 2 instead of base 10? The short answer is that it is just simpler
and more cost-effective at the hardware level. It is much easier to build electrical components
that have two discrete states instead of ten. As an analogy, it is much cheaper
to build an ON/OFF light switch than it is to build a knob that dims the light gradually.
Instead, computers work like an array of ON/OFF switches that together can represent `2^n` discrete
states where `n` is the number of switches.

These switches are called bits in computing. A bit can represent the number zero or one. On their own, bits are not that useful because they can encode very little information. To be able to represent some natural
numbers we need to use more bits together. Let's use, say, 8 bits to make a number:

```
00101010
```

To see what this makes in the decimal system, we have to multiply each bit by the number it
represents in the decimal system. Starting from the right, the decimal they represent is 2^n
where n is the position of the bit:

```
  0    0    1    0    1    0    1    0
  *    *    *    *    *    *    *    *
  2^7  2^6  2^5  2^4  2^3  2^2  2^1  2^0
```

Expanding the powers of two turns this into:

```
0    0    1    0    1    0    1    0
*    *    *    *    *    *    *    *
128  64   32   16   8    4    2    1
```

And now, performing the multiplication

```
0    0    1    0    1    0    1    0
*    *    *    *    *    *    *    *
128  64   32   16   8    4    2    1
↓    ↓    ↓    ↓    ↓    ↓    ↓    ↓
0    0    32   0    8    0    2    0
```

The last step is to add all the results together to get the number in decimal.

```
+  0    0    32   0    8    0    2    0
---------------------------------------
=                                    42
```

Once you know how to convert between number bases, you realise that binary 00101010
and decimal 42 are _the same thing_. They express exactly the same number just like
1km expresses the exact same distance as .621 miles.

There are some limits, however, that we must consider. Just like a 3-digit decimal number
can only express numbers between 0-999, there are constraints to binary numbers. For example,
an 8 bit binary number can express at most

`2^8 = 256`

discrete values.

Our current representation has another limitation: it can only express positive integers.
We need to add a little twist to be able to represent negative integers. The change is
to reserve the leftmost bit (also called the most significant bit) to signal whether the
number is positive or negative. For example:

```
00000001 == 2
```

```
10000001 == -2
```

This isn't a magic trick, however. An 8 bit number can still only represent 256 discrete
values, the range shifts from `0...256` to `-128...128`.

Both representations are used often in programming. They're called _signed_ and _unsigned_
integers depending on whether the most significant bit is treated as a sign bit or not.
Choosing between the two is a tradeoff – signed numbers can go negative but the maximum positive number drops to half.

I didn't choose the number 8 randomly: a string of 8 bits forms what we call a _byte_.
While bytes are small and sufficient for a surprisingly large number of tasks,
they are still very limited in size. 256 numbers is not nearly enough to
run complex calculations or show, for example, a bank account balance. On modern computers,
you can choose to use more than 8 bits to represent an integer for these purposes. This is
called the integer _width_. Computers are best at dealing with integers whose width is a power of
two, so most integers used in the wild have either 8, 16, 32, 64, or 128 bit width.

Good! We have a way to encode human-friendly numbers to a format that our hardware can work with.
You might be asking at this point how useful it is to just being able to store numbers in binary.
Surely the data that we need to work with these days is more complex than that.

Hold that thought while we dive into how integers are used to encode the lowest-level form
of software: Machine code.

### 1 10 20 0

From this point on, we will assume that a device exists which can run machine code.
It has a CPU, RAM memory, buses to transfer data etc. Generally what one would call a computer
architecture. For example, I'm writing this blog on a MacBook Pro which has one of the most common
computer architectures called x86. The architecture has intrinsic, hardware-level routines to
do very simple computations, move data between RAM and the CPU, manipulate registers etc. We will
get to what that means later. For now let's just assume that the hardware is available.

The architecture must provide a mechanism to run programs. From the time computers were invented to
the 70s the main format to store programs was [punchcards](https://en.wikipedia.org/wiki/Punched_card) ). Since physically punching cards is not
the most convenient way to write programs, a better way had to be invented. Nowadays every program that runs directly on the hardware is stored as a binary file, literally as ones and zeroes. These files contain what is called machine code because they are sent directly to the machine to be executed.

This is where learning about binary pays off. While the machine code is stored as ones and zeroes,
the CPU reads it one byte (8 bits) at a time. A sequence of bytes form what is called an instruction. These instructions conform to a certain format that the CPU is wired to read and execute. For example, let's make up an instruction set with the following format:

```
1 byte | 1 byte | 1 byte
-------+--------+-------
OPCODE | NUMBER | NUMBER
```

In this format, one instruction consists of three bytes: one byte for the opcode, one for the first operand, and one for the second operand.

The word [opcode](https://en.wikipedia.org/wiki/Opcode) refers to what type of operation we want the CPU to execute. Think of it like a function call, except the function is a hardwired circuit in the processor. Now let's define what our imaginary architecture can do and what opcodes one would use to invoke those routines.

```
Opcode | Instruction name
-------+-----------------
0      | HALT
1      | ADD
2      | SUB
3      | MUL
4      | DIV
```

This table defines the [instruction set](https://en.wikipedia.org/wiki/Instruction_set) of our made-up architecture.

So, our computer can add, subtract, multiply and divide two numbers. There's also a code for halting the execution for when the program has finished. A simple program for this machine would look like this:

`00000001 00001010 00010100 00000000`

Remember, computers always see data in binary.  To make this program more human-readable, let's convert the it back to decimal:

`1 10 20 0`

When we execute this machine code the computer reads in the first byte and sees that it needs to execute the `ADD` instruction, taking arguments `10` and `20`. Once that is done, it sees that it has to `HALT` the execution. The CPU halts, concluding our first program on our imaginary computer.

This section showed you how computers read _binary_ programs in a certain format and execute the
specified instructions on _binary_ data. Nowadays no one really writes machine code by hand. It is
just way too arduous to type out numbers with no mnemonics associated with them. This is why assembly was invented.

### ADD 10 20 HALT

Assembly, often abbreviated as _asm_, is the lowest-level code that humans usually write. It is a very small abstraction on top of machine code
to make life a bit more convenient for us programmers. Here's how the same program would look like in assembly:

```
ADD 10 20
HALT
```

This code will go through something called the _Assembler_ which replaces the word `ADD`  with the number 1 and `HALT`  with 0, producing the same binary file that we started with.

_Assembler_ is the first time we've encountered a _compiler_ on our journey to understand software. Compilers play a big role in being able to write code in higher-level languages. The job of a compiler is to transform a program from one
language or format to another one. In this case, the assembler does that by replacing each occurrence of `ADD` or `HALT`
with their corresponding binary version that the CPU can understand.

Compilers aren’t limited to just converting programs though. They often analyse the code and help the programmer avoid mistakes before transforming the code. For example, the assembler could analyse the code we've written and let us know if we've made a mistake:

```
ADD 10
HALT
```

Seeing a program like that, the assembler might refuse to convert it into the binary equivalent because the code we've written does not conform to the specified opcode format (_the second argument needed in the addition is missing_).


### Next steps

At this point, you may be questioning the usefulness of the imaginary computer we have. It can only
add constant integers and there's no way to use the output of one computation in another computation.

Stay tuned for the next part of this blog series where we'll talk about memory in computers and
how computation results are kept track of.
