# The Spite standard library

The classes every Spite program reaches by path: `load "../spite_standard_library/windows"` from a folder beside
this one. `common/` holds the classes every system has, and `windows/`, `linux/` and `mac/` the parts of each
system, which reopen the classes of `common/`. The compiler names no class of the library.

What is left is in the [issues](https://github.com/Spite-Language/spite_standard_library/issues) of this
repository, and the language's in the [compiler's](https://github.com/Spite-Language/spite_compiler/issues).
Work here follows [`spite_compiler/AGENTS.md`](https://github.com/Spite-Language/spite_compiler/blob/master/AGENTS.md): a change a program can see
updates the specification and the docs in the same push.

## The rules the library keeps

**Nothing is implicit.** A method the compiler used to supply (a function with no body, or a call the compiler
knows by name) is written in the library, in Spite, where a reader can see it. The compiler keeps only what it
cannot do in Spite: the machine code, and the memory a value is kept in.

**The library does not touch memory.** A class that needs memory holds a `Buffer`, and says how much room it
needs. The compiler places the buffer and optimizes it from what it proves, so a program never thinks about
memory: `Integer` is a whole number that happens to be four bytes wide, and the compiler keeps it in a
register. A value is read from its room and never through `this`: `this` is the place the room is kept, and
the read is what says what the value is.

**Every name says what it holds.** No abbreviations, and no em dashes.

**There is no `List<Byte>`.** A run of bytes is a `Buffer`, written or made, and the compiler refuses a list of
bytes anywhere (D626 of the language). No class here answers or takes one.
