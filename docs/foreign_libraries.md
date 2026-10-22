# Foreign libraries

A `DynamicLibrary` calls the functions a library written in C exports. The library names the file, how
its functions are named, and the header its constants come from.

## Calling back into Spite

A `ForeignCallback` hands C a function of the program. It keeps the function, and so the object the
function belongs to, for as long as it lives, and answers `address`, the function pointer C calls, and
`context`, the number C passes back so the call finds its object:

```spite
var ticking = ForeignCallback(ticker.ticked, 'context_last')
library.keep_tick(ticking.address, ticking.context)
```

The compiler writes the function C calls, beside each `ForeignCallback(...)`, and fills `address` and
`context` there. When the `ForeignCallback` is dropped, `_let_go` empties its place, and a call C makes
after that stops the program, naming the function.
