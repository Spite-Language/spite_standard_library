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

The `Buffer` is the class that names memory. Its two methods with no body are the ones the compiler
writes, because reading a byte of memory is not something Spite can say:

```spite
func _read_byte_at(at: Integer): Integer

func _write_byte_at(at: Integer, value: Integer)
```

The receiver of both is the room: the place the bytes are kept. A class that holds a buffer reads its
own room through them. They are private, so a program reads the bytes of a buffer through the buffer's
own methods and never through these two.

The public way to read a byte of a buffer is its index, `buffer[0]`, which is the buffer's own `get_at`:
it answers the byte at that place of the memory, and `null` when the place is outside it. The write is
`set_at`, and it stops when the place is outside the memory. The bytes a buffer holds as items are the
first ones of that memory, and the methods that read and write items say so with their own checks.

A buffer copies the bytes of its room with `copy_the_room`, which answers a buffer that holds those bytes
as items, and it compares its bytes with the bytes of another buffer with `compare_bytes`, which answers a
number: less than zero when the first is smaller, zero when the bytes are the same, and more than zero
when the first is bigger.

## The byte order

The byte order belongs to each read and each write, not to the buffer. A buffer is made with a size and
no order, `Buffer(size)`, and it holds bytes of both orders, each read and written the way the format
that holds them says.

A number of two bytes or more is read and written through a function that names its order after the
number: `_little_endian`, the order of every machine Spite compiles for, or `_big_endian`, which puts the
most significant byte first. So a call says what it reads:

```spite
func read_integer_little_endian(): Integer?
func read_integer_big_endian_at(at: Integer): Integer
func append_short_little_endian(value: Integer)
func write_long_big_endian_at(at: Integer, value: Integer)
```

A byte and a tiny are one byte, and they have no order: `read_byte`, `read_tiny`, `append_tiny` and the
`write_byte_at` and `write_tiny_at` that go with them.

The two orders are the values of the enum `ByteOrder`, `'little_endian'` and `'big_endian'`.

## The boolean

A `Boolean` is one byte. It says so with its buffer, and the compiler reads the number and gives the
boolean its byte. The value is read from the room, and not from `this`, because `this` is the place the
value is kept and not the value:

```spite
var _buffer = Buffer(1)

func to_string(): String {
    if _buffer[0] != 0 {
        return "true"
    }
    return "false"
}
```

## The numbers

A number is a room too, of the width the number takes, and it reads its value with the buffer's own
read. The methods are written in the library, so a reader can see what a number is:

```spite
var _buffer = Buffer(4)

func to_buffer(): Buffer {
    return _buffer.copy_first_bytes(4)
}

func to_integer(): Integer {
    var mine = to_buffer()
    return mine.read_integer_little_endian_at(0)
}
```

`to_buffer` gives a copy of the room's bytes, and the copy is a buffer of its own, so a program reads
the bytes of a number without reaching into its room, and a number compares itself with another
through the two copies:

```spite
func less_than(other: Integer): Boolean {
    var mine = to_buffer()
    var theirs = other.to_buffer()
    var left = mine.read_integer_little_endian_at(0)
    var right = theirs.read_integer_little_endian_at(0)
    return left < right
}
```

The bytes are read with the order of the machine, the order the compiler writes a value in, so a
number that is an attribute of an object holds the same bytes the compiler wrote. A number that is
signed reads the sign with the read that knows it: `Tiny`, `Short`, `Integer` and `Long` are signed, and
`Byte`, `UnsignedShort`, `UnsignedInteger` and `UnsignedLong` are not.

The widths are the ones the language has: `Tiny` and `Byte` are one byte, `Short` and `UnsignedShort`
two, `Integer` and `UnsignedInteger` four, `Long` and `UnsignedLong` eight, and `Float` four and
`Double` eight.

The numbers answer their maths as well: `absolute`, `minimum`, `maximum`, `sum`, `subtract`, `multiply`,
`divide` and `remainder` read the value of both numbers and answer a whole number.

## The string

A string is a room of bytes, and the bytes are its characters in the order they were written. The
string says how much room it needs, and it keeps the count of the characters it holds:

```spite
var _buffer = Buffer(1024)
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

A dictionary is a list of keys and a list of values, and a key and its value are at the same place in
the two. The key is looked up in the list of the keys, and the value is taken from the list of the
values at the place the key was found.

The compiler turns an index into a call: `a[b]` is `a.get_at(b)`, and `a[b] = v` is `a.set_at(b, v)`,
so a dictionary answers `get_at` and `set_at` with a key and a value.

