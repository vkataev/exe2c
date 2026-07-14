# exe2c
Agent harness for smooth transition of any executable to a single .c file

Executable formats like LE, PE, and ELF describe how to build a process in memory.

---

# 1. A raw binary

Suppose you have a file:

```text
program.bin

Offset   Bytes
------   ----------------
0000     B8 2A 00 00 00
0005     C3
```

which is

```asm
mov eax,42
ret
```

There is **no header**. The CPU cannot execute a file. It executes **memory**. So someone has to decide:

> "I'm going to copy this file into RAM at address 0x100000."

Memory becomes

```text
Memory

100000  B8 2A 00 00 00
100005  C3
```

and execution begins at `0x100000`.

---

# 2. What is a loader?

The program that copies the file into memory is called the **loader**.

Its job is basically

```text
Open file
↓

Allocate memory
↓

Copy bytes
↓

Jump to program
```

DOS has an MZ loader. Windows has a PE loader. Linux has an ELF loader. DOS4GW has an LE loader.

They're all doing the same general job.

---

# 3. Why not just copy the whole file?

Imagine the file contains

```text
HEADER
CODE
DATA
DEBUG INFO
IMPORT TABLE
RESOURCE TABLE
```

Only **code and data** belong in RAM. The header is just instructions for the loader. Therefore the loader doesn't simply copy the file. It interprets it.

---

# 4. Memory map

The executable says something like

```text
Code
size = 300 KB
load at 0x000000

Data
size = 20 KB
load at 0x0050000

BSS
size = 100 KB
allocate only
```

This is called the **memory map**. It describes what the process's memory should look like.

Example:

```text
Memory

00000000  Code
0004B000  More code
00050000  Data
00055000  BSS
00070000  Heap
```

Notice this has nothing to do with where things were inside the file.

BSS is the **block starting symbol**, it contains statically allocated variables that are **declared but have not been assigned a value** yet.

---

# 5. Why separate code and data?

Suppose your source is

```c
int counter;

int square(int x)
{
    return x*x;
}
```

The compiler produces

```text
Code
square()

Data
counter

BSS
counter if initially zero
```

They have different purposes.

---

# 6. Virtual addresses

Inside the program the compiler may generate

```asm
mov eax,[00500120h]
```

That means

> Read memory from address `0x500120`.

It doesn't care where in the file that variable came from. It only cares where it will live **after loading**.

---

# 7. Why relocations exist

Now imagine this program.

```c
int value=123;

int main()
{
    printf("%d",value);
}
```

The compiler generates something like

```asm
mov eax,[????????]
```

But during compilation it doesn't yet know exactly where `value` will end up. So it leaves a placeholder.

For example

```asm
mov eax,[00000000]
```

Then it records

> At offset `0x1234`, this number must later become the address of `value`.

That record is called a **relocation** (or in LE terminology, a **fixup**).

---

# 8. The loader applies relocations

Suppose the loader decides

```text
Code begins

00000000

Data begins

00100000
```

It now patches the instruction

From

```asm
mov eax,[00000000]
```

to

```asm
mov eax,[00100000]
```

Now it works.

---

# 9. Why DOS4GW needs objects

A DOS4GW executable isn't one big chunk.

It says

```text
Object 1
Code

Object 2
Read-only data

Object 3
Writable data

Object 4
BSS
```

Each object has

* size
* memory address
* pages

The loader builds memory from these objects.

---

# 10. Pages

Instead of storing one huge block, LE stores pages.

Usually

```text
4096 bytes
```

each.

Imagine

```text
Object 1

Page 1
Page 2
Page 3
Page 4
```

Each page says

```text
I'm stored at file offset XXXXX
```

The loader copies every page into RAM.

---

# 11. Why not store objects directly?

Because pages allow

* demand loading
* compression
* sharing
* easier swapping

OS/2 made heavy use of this.

DOS4GW inherited the format.

---

# 12. Putting it all together

When DOS4GW loads an LE executable it roughly does

```text
Read header

↓

Read object table

↓

Allocate RAM for every object

↓

Read page map

↓

Copy pages

↓

Zero BSS

↓

Apply relocations

↓

Set ESP

↓

Jump to EIP
```

After that, the CPU simply executes normal 32-bit instructions. The loader's work is finished.

---

## Why reverse engineers care

When you open an executable in Ghidra or IDA, the disassembler is essentially acting as a **loader** first. It has to recreate the program's memory layout before it can make sense of the instructions.

If it gets that layout wrong—for example, by placing the code or data at the wrong addresses—then addresses embedded in instructions won't point to the right places. Cross-references break, functions don't get connected to the data they use, and the analysis becomes much harder.

That's why understanding the LE header matters. It's not just file metadata; it's the recipe the original DOS4GW loader used to build the program in memory. A disassembler that follows the same recipe will "see" almost the same memory image the CPU saw when the program originally ran.

Once these ideas click, formats like MZ, LE, PE, and ELF start to look like different dialects of the same language: each one tells a loader **what memory to allocate, what bytes to copy there, what addresses to fix up, and where execution begins**.
