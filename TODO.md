# TODO

What the standard library still owes the compiler, in the order it is done.

The library lives here: `common/` holds the classes every system has, and `windows/`, `linux/` and `mac/`
hold the parts of each system. Each of them becomes a repository of its own later, so nothing here depends on
the compiler's repository, and the compiler names no class of the library.

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

## How the library moves

**One class at a time, as the compiler needs it.** The compiler reads a class when a program names it, so a class
is moved when the compiler asks for it, and not before. A class that has not moved yet is not in the library the
compiler reads, and the programs the compiler compiles do not name it yet.

**The library is reached through a junction.** The compiler's folder holds `.spite/library`, and that is a
junction to this repository, so a program loads the library by the name `.spite/library` and the new library
is the one it reads. The old compiler still reads the library of its own release, because it does not know the
rooms and the two methods that read and write a byte yet, so the old library stays where it is until the old
compiler is retired.

**Each class is rewritten as it moves.** The class that arrives in `common/` is the new explicit version, not a
copy of the old one. The old version stays in the first repository (archived), in its `library/` folder, until the old
compiler is retired, because the old compiler's rules are written against the paths there.

## The list

- [x] **The memory of an object.** The compiler gives an object a block of memory and reads and writes the
      numbers in it. This is the first thing the compiler needs, and the rest of the list waits for it.
- [x] **`Buffer`.** The class that holds bytes. A class says how much room it needs with
      `var _buffer = Buffer(bytes)`, and the compiler lays that room out at the front of the class's own
      block. The two methods that read and write a byte have no body, and the compiler writes them, because
      reading a byte of memory is not something Spite can say. The byte order belongs to each read and each
      write, and is named after the number (`read_integer_little_endian`, `read_integer_big_endian_at`), so
      a buffer holds bytes of both orders. The room is copied with `copy_first_bytes`, which answers a
      buffer that holds those bytes as items, and two buffers are compared with `compare_bytes`. What is
      not written yet: `copy_to`, `text`, and the ones that read and write a `Float` or a `Double`: the
      compiler has no float arithmetic yet, so a buffer cannot read one.
- [x] **`Boolean`.** One byte, read from the room with `_buffer[0]`, the buffer's own public read. `this` is
      the place the room is kept, and never the value. The methods are written here: `to_string`,
      `to_debug`, `to_buffer`, `to_boolean`, the casts to the numbers, and the comparisons `equals`,
      `less_than` and `greater_than`.
- [x] **The numbers.** `Integer`, `Byte`, `Short`, `Long`, `Tiny`, `UnsignedInteger`,
      `UnsignedLong`, `UnsignedShort`, `Float` and `Double` each hold a `Buffer` with the room they need, and the
      compiler reads the size of the number from it. Each one reads its value with the buffer's own read
      (`read_byte_at`, `read_unsigned_short_big_endian_at`, `read_unsigned_integer_big_endian_at`), and
      every method is written here: `to_buffer`, the casts to every other number (`to_integer`,
      `to_boolean`, `to_tiny`, `to_byte`, `to_short`, `to_unsigned_short`, `to_unsigned_integer`,
      `to_long`, `to_unsigned_long`), `equals`, `less_than` and `greater_than`. The maths is written here
      too: `absolute`, `minimum`, `maximum`, `sum`, `subtract`, `multiply`, `divide` and `remainder`. What
      is not written
      yet: the bitwise functions, the `to_string` and `to_debug`, the casts of a `Float` and
      a `Double`, and the 64-bit and float arithmetic, which the compiler does not have yet.
- [x] **`String`.** Its characters live in a `Buffer`, and the string keeps the count of them. The
      methods are written here: `length`, `code_at`, `append_code`, `character_at`, `slice`, `sum`,
      `equals`, `less_than`, `greater_than`, `starts_with`, `ends_with`, `contains`, `split`,
      `to_long`, `to_integer`, `to_string` and `to_debug`. What is not written yet: the text of a
      buffer, and the characters of a string that are not one byte each.
- [ ] **`Memory.Address`.** It goes away: the library names the memory it needs with a `Buffer`.
- [ ] **`Memory.Heap` and `Memory.Arena`.** The allocator is Spite here, in one file, and it reaches the C
      library through `DynamicLibrary`, so the compiler needs to call a function in a DLL by name and nothing
      else.
- [x] **`List` and `Dictionary`.** They hold their items in a `Buffer`, and an item is of any type: the
      two methods that read and write an item, `_read_value` and `_write_value`, have no body and the
      compiler writes them. What the compiler still owes: the memory of a value of any type, and the
      monomorphisation of a generic class, so that the size of an item is known. The dictionary is a hash table over its two
      lists ([the dictionary](docs/memory.md#the-dictionary)).
- [ ] **`Vector` and `Items`.** They hold their items in a `Buffer`, the way the list does.
- [x] **The classes that reach the operating system.** `Console`, `Program` and `Directory` are written
      here, and they reach the C library through `DynamicLibrary`. The parts each system has are in the
      system's folder, and they reopen the classes of `common/`: the `windows` folder reopens the
      `console` and the `program` with the reading of the console and the sleeping and the paths of the
      system. What is not written yet: `File`, `Clock`, the sockets and the rest.
- [ ] **The rest of the classes**, one at a time, as the compiler needs them.

