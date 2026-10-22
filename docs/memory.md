# Memory

The memory of a value is the compiler's work, and the library only names it. A class that holds memory
holds a `Buffer`, and says how many bytes it needs:

```spite
var _buffer = Buffer(1)
```

The compiler reads that number, gives the class a block of memory, and lays the room out at the front of
the block. The room is where the value is kept: `this` is the place the room is kept, and never the
value itself. A value is read from the room, and written to it, and the read and the write are named.

## The buffer

The `Buffer` is the class that names memory. Its methods with no body are the ones the compiler writes,
because reading memory is not something Spite can say: `_resize_room(size)`, and a read and a write for
each whole number, in the order of the machine:

```spite
func _read_byte_at(at: Integer): Byte

func _write_byte_at(at: Integer, value: Byte)

func _read_integer_at(at: Integer): Integer

func _write_integer_at(at: Integer, value: Integer)
```

The compiler reads the type of the answer, or of the value, and makes each one a single load or store of
that width. They check nothing, so a class reads its own room through them only where it has proven the
place, and a program reads a buffer through its public methods.

The public way to read a byte of a buffer is its index, `buffer[0]`, which is the buffer's own `get_at`:
it answers the byte, and `null` outside the bytes the buffer holds. The write is `set_at`, and it stops
outside them. `Buffer(size)` is room for `size` bytes; appending past it doubles the room.

## The byte order

The byte order belongs to each read and each write, not to the buffer. A number of two bytes or more is
read and written through a function that names its order after the number: `_little_endian`, the order
of every machine Spite compiles for, or `_big_endian`, which puts the most significant byte first:

```spite
func read_integer_little_endian(): Integer?
func read_integer_big_endian_at(at: Integer): Integer?
func append_short_little_endian(value: Short)
func write_long_big_endian_at(at: Integer, value: Long)
```

A read answers `null` when the bytes are not all in the buffer, and a write stops. A byte and a tiny are
one byte, and they have no order.

Each `_big_endian` read and write is one load or one store too: the buffer declares a read and a write of
each width with no body, `_read_integer_big_endian_at(at)` and `_write_integer_big_endian_at(at, value)`,
and the compiler makes each one the load or the store of that width with the bytes swapped in the register,
so a big-endian number costs what a little-endian one costs and nothing is copied to read it.

## The numbers

A number is a room too, of the width the number takes, and it reads its value from the room once:

```spite
var _buffer = Buffer(4)

func _value(): Integer {
    return _buffer._read_integer_at(0)
}

func less_than(other: Integer): Boolean {
    var left = _value()
    var right = other._value()
    return left < right
}
```

A conversion answers the value in the other type, and the language checks it: a value that does not fit
the narrower type stops the program instead of wrapping. A `Boolean` is one byte read the same way.

The widths are the ones the language has: `Tiny` and `Byte` are one byte, `Short` and `UnsignedShort`
two, `Integer` and `UnsignedInteger` four, `Long` and `UnsignedLong` eight, and `Float` four and
`Double` eight. The numbers answer their maths as well: `absolute`, `minimum`, `maximum`, `sum`,
`subtract`, `multiply`, `divide` and `remainder`, and `to_string`.

## The string

A string is a room of bytes, and the bytes are its characters in the order they were written. The
string says how much room it needs, and it keeps the count of the characters it holds:

```spite
var _buffer = Buffer(16)
var _length = 0

func length(): Integer {
    return _length
}

func code_at(index: Integer): Integer {
    return _buffer._read_byte_at(index)
}
```

A string reads the byte of its own room through the two methods the compiler writes, the way every
class that holds a buffer does. The characters are added with `append_code`, and the text of another
string is added with `sum`, which is the way two strings are put together.

A string compares itself with another through the characters, one at a time, and answers `equals`,
`less_than` and `greater_than`. It splits itself with `split`, which answers the pieces between the
separators, and it reads a number out of itself with `to_long` and `to_integer`.

## The list

A list holds its items in a room, and an item is of any type, so the two methods that read and write
an item have no body and the compiler writes them:

```spite
func _read_value(index: Integer): $element_type

func _write_value(index: Integer, value: $element_type)
```

The list keeps the count of the items it holds and the room they are in, and it answers `count` and
`is_empty`. An item is added with `append`, `prepend` and `insert`, and read with `get_at` and
`set_at`; `first` and `last` answer the item at the ends, and `remove_at`, `remove_first` and
`remove_last` take one away. `contains` and `index_of` look for an item, `join` puts the items
together with a separator, `copy` answers a list of the same items, and `each` calls a function with
every item.

## The dictionary

A dictionary is a list of keys and a list of values, in the order the keys were first set, and a key and
its value are at the same place in the two. Over them is a table of slots, a `Buffer` with a power of two
of them: each slot is eight bytes, the place of an entry plus one (zero for an empty slot) and then 32 bits
of the key's hash. The table holds at most half as many entries as slots, and an empty dictionary has none.

A key is hashed once for each lookup, set and removal. A whole number, a `Boolean`, an enum value and an
object are hashed as their bits (an object's bits are its identity), through `_key_bits(key)`, which has
no body: the compiler answers the key's own register. A `Float` or a `Double` is hashed as its bits once
both zeros are one zero and every not-a-number is one value, and a `String` by its bytes, eight at a time.
The bits are multiplied by a large odd number, and the top 32 bits of the product are the slot's hash; its
top bits again are the slot the key starts at. A lookup walks the slots from there to the first empty one,
compares a slot's hash bits before it reads the key, and reads the key from the list only when they match.

Growing the table doubles it and moves each slot by the hash bits it keeps, so no key is hashed again.
Removing a key empties its slot and moves back each entry after it that may move, then takes one from the
place of every entry after the removed one: removing takes time in proportion to the table.

The compiler turns an index into a call: `a[b]` is `a.get_at(b)`, and `a[b] = v` is `a.set_at(b, v)`,
so a dictionary answers `get_at` and `set_at` with a key and a value.


## The copy table

A deep copy of a structure whose references can lead back to an object already being copied
remembers every object it copies in a `Spite.Internal.CopyTable`, so each object is copied once.
A reference the compiler keeps uncounted, such as a child's `parent`, is copied as the original
object first, and the table settles it with `later` and `settle` once every object is copied: it
then holds the copy when the same deep copy made one, and the original object otherwise.

