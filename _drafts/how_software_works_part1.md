---
layout: post
title: "How software works part Ⅰ"
---

I started my journey in software writing Javascript code in the browser. I later
graduated to writing backend servers in Ruby, talking to databases and other APIs,
delivering real software projects. I was able to write complex software but at the
same time I had no understanding of how my software was actually executed on the CPU.
In high level languages you don't even need to know the difference between registers,
the stack or the heap. This series of blogs is my attempt to create a primer for this
information. It is not meant to serve as a complete description of how computers work,
rather I'd like to build a good understanding of what abstractions
lie beneath the applications we write as regular programmers.

Some parts of this overview are going to be simplified and there will be gaps in the whole picture.
In addition, this series is called 'How software works' rather than 'How computers work'.
I have a very superficial knowledge of computer hardware which is why, in this series, I will
take hardware for granted and stay on the software side of computing.

### 00101010

Everyone knows that software is all ones and zeroes in the end. What does that mean?
How do my ruby objects get translated to binary? What's the big deal with ones and zeroes anyways?

The binary nature of our computers is largely hidden from even fairly low-level programs. When
you get to the lowest level of software, however, you do need to understand binary and why computers
represent data using ones and zeroes.

Why encode information in base 2 instead of base 10? The simple answer is that it is just simpler
and more cost-effective at the hardware level. It is much easier to build electrical components
that have two discrete states instead of ten. As an analogy, it is much easier
to build an ON/OFF light switch than it is to build a knob that dims the light gradually.
Instead, computers work like an array of ON/OFF switches that together can represent 2^n discrete
states where n is the number of switches.

These switches are called bits in computing. A bit can be either a 0 or 1. On their own, bits are
not that useful because they can encode very little information. To be able to represent some natural
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

When we multiply these numbers together, we get something like this:

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
                                     42
```

This method of storing different numbers means that 8 bits can take at most

`2^7 + 2^6 + 2^5 + 2^4 + 2^3 + 2^2 + 2^1 + 2^0 = 2^8 = 256`

discrete states.

This system can only represent positive integers.
We need add a twist to the system to be able to represent negative integers. The change is
to reserve the left-most bit (also called the most significant bit) to signal whether the
number is positive or negative. This means that instead of being able to represent numbers `0 to 256`
our range changes to `-128 to 128`.

Both representations are used often in programming. They're called _signed_ and _unsigned_
integers respectively. Choosing between the two is a tradeoff – signed numbers can go negative
but the maximum number drops to half.

I didn't choose 8 bits randomly: a string of 8 bits forms what we call a _byte_.
While bytes are small and sufficient for a surprisingly large number of tasks,
they are still very limited in terms of size. 256 numbers is not nearly enough to
run complex calculations or show, for example, a bank account balance. On modern computers,
you can choose to use more than 8 bits to represent an integers for these purposes. This is
called the integer _width_.

Good. We have a way to encode human-friendly numbers to a format that our hardware can work with.
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
get to what all of that means but for now let's just assume that the hardware is there.

The architecture must provide a mechanism to run programs. From the time computers were invented to
the 70s the main format to store programs was punchcards. Since physically punching cards is not
the most convenient way to write programs, a better way had to be invented. Nowadays every program that runs directly
on the hardware is stored as a binary file, literally as ones and zeroes. These files contain what is called
machine code because they are sent directly to the machine to be executed.

This is where learning about binary pays off. While the machine code is stored as ones and zeroes,
the CPU reads it one byte at a time. A sequence of bytes form what is called an instruction. These instructions
conform to a certain format that the CPU is wired to read and execute. For example, let's make up an instruction
set with the following format:

```
1 byte | 1 byte | 1 byte
-------+--------+-------
OPCODE | NUMBER | NUMBER
```

The word opcode refers to what type of operation we want the CPU to execute on the given arguments. Now let's define
what our imaginary architecture can do and what opcodes one would use to invoke those routines.

```
Opcode | Instruction name
-------+-----------------
0      | HALT
1      | ADD
2      | SUB
3      | MUL
4      | DIV
```

So, our computer can add, substract, multiply and divide two numbers. There's also a code for halting the execution
for when the program has finished. A simple program for this machine would look like this:

`00000001 00001010 00010100 00000000`

To make this more human-readable, let's convert the binary digits back to decimal:

`1 10 20 0`

When we execute this machine code the computer reads in the first byte and sees that it needs to execute the
`ADD` instruction, taking arguments `10` and `20`. Once that is done, it sees that it has to `HALT` the execution.
The CPU halts, concluding our first program on our imaginary computer.

I hope this section showed you how computers read _binary_ programs in a certain format and execute the
specified instructions on _binary_ data. Now, in reality no-one really writes machine code by hand. It is
just way too arduous to type out numbers with no mneumonics associated with them. This is why assembly was invented.

### ADD 10 20 HALT

Assembly, often abbreviated as _asm_, is the lowest-level code that humans usually write. It is a very small abstraction on top of machine code
to make life a bit more convenient for us programmers. Here's how the same program would look like in assembly:

```
ADD 10 20
HALT
```

This code will go through something called the _assembler_ which replaces the word `ADD` with the number 1
and `HALT` with 0, producing the same binary file that we started with.

Assembler is the first time we've encountered a _compiler_ on our journey to understand software. Compilers play a big
role in being able to write code in higher-level languages. The job of a compiler is to transform a program from one
language or format to another one. In this case, the assembler does that by replacing each occurence of `ADD` or `HALT`
with their corresponding binary version that the CPU can understand. It is not however limited to just doing that job.
For example, the assembler could analyse the code we've written and let us know if we've made a mistake:

```
ADD 10
HALT
```

Seeing a program like that, the assembler might refuse to convert it into the binary equivalent because the
code we've written does not conform to the specified opcode format.


### Next steps

At this point, you may be questioning the usefulness of the imaginary computer we have. It can only
add constant integers and there's no way to use the output of one computation in another computation.
Stay tuned for the next part of this blog series where we'll talk about memory in computers and
how computation results are kept track of.
