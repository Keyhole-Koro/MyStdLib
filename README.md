# MyStdLib

Shared MyLang standard library: generic containers (`Vec`, `Slice`, `Arena`,
`RingBuffer`, `IntrusiveList`, `HashMap`, `Option`, `Result`) plus string,
byte, bit-array, and assertion helpers (`str`, `InlineString<N>`, `bytes`,
`bitset`, `assert`), and the readers of what the toolchain leaves in the image:
`memory/section.mln` (a linker-collected section by name, as a `Slice<T>`),
`format/mbin.mln` (an MBIN executable image in a buffer: header fields, the
section directory, virtual-address translation) and `meta/annotations.mln`
(the compiler's annotation metadata rows, as an iterator:
`annotations.named("app")`, `it.next()`, `it.fn()`, ...; `in_image(buf)`
reads the table of an executable on disk; see
`docs/design/toolchain-collected-sections.md` in MyComputer).

Core traits (see `docs/traits.md` in MyLangCompiler):

| trait | module | language feature |
| --- | --- | --- |
| `From<S>` | `core/convert.mln` | `?` across error types, `Type::from(v)` |
| `Eq` | `core/cmp.mln` | `==` / `!=` |
| `Hash` | `core/hash.mln` | `HashMap<K, V>` keys |
| `Copy` | `core/copy.mln` | explicit copy semantics for safe structs |
| `Iterator<T>` | `core/iter.mln` | `for (T x in it)` |
| `Drop` | `core/drop.mln` | automatic cleanup (`String` implements it) |
| `Describe` | `core/describe.mln` | `describe()`: a short message; the error contracts implement it |

`Option` and `Result` methods: `is_some()` / `is_none()`, `is_ok()` /
`is_err()` (borrowing), and `unwrap_or(fallback)`, `ok()`, `err()`
(consuming). They work on a call result too: `ui.window_x(w).unwrap_or(0)`.

Iterating the collections (`for` needs `Option` imported):

| call | yields |
| --- | --- |
| `slice.iter()`, `vec.iter()` | `T *` for each element, front to back (`SliceIter<T>`) |
| `ring.drain()` | each queued `T`, popping it (`RingDrain<T>`) |
| `map.entries()` | `MapEntry<K, V>` (`key`, `value` pointers) for each stored pair |

Lookups that can miss return the element's address as an Option:
`vec.at(i)` and `slice.at(i)` (None out of range) and `map.get(&key)` (None
when absent). Read with a default as `*v.at(i).unwrap_or(&fallback)`, or
propagate with `?` in a function returning Option. The collections have no
`[]`: nothing in the standard library panics.

Debugging:

| module | role |
| --- | --- |
| `debug/error_trace.mln` | error return trace: where an `Err` was created and each `?` it returned through, recorded by code built with `mlc --error-trace` (`depth()`, `site(i)`, `format(buf, cap)`) |

Hosted modules:

| module | role |
| --- | --- |
| `hosted/fs.mln` | `Result`-based filesystem client API |
| `hosted/process.mln` | generic process client API; `spawn()` returns `Result<i32, SpawnError>` |
| `hosted/log.mln` | hosted diagnostic output |
| `platform/myos/syscall.masm` | MyOS syscall trap binding |

Shared contracts used by hosted modules:

| contract | role |
| --- | --- |
| `contracts/io/fs.contract.mln` | `FsError` and `SeekWhence` |
| `contracts/myos/services.contract.mln` | OS_CALL service numbers |

Used by [MyKernel](https://github.com/Keyhole-Koro/MyKernel),
[MyOS](https://github.com/Keyhole-Koro/MyOS), and exercised by
[MyLangCompiler](https://github.com/Keyhole-Koro/MyLangCompiler)'s generic
import tests. See [GENERICS.md](GENERICS.md) for the generic container API.

Every module is imported by explicit relative path (there is no implicit
prelude): `import { Vec } from "collections/vec.mln";` -- a type's exported methods
(`v.push(x)`, `s.len()`) travel with it. `text/str.mln` supplies the compiler-known
borrowed `str` view (`{ char* data; i32 length; }`) and its bridge to legacy
`char*` buffers:

```mln
import str from "text/str.mln";

str name = "MyOS";
if (name.starts_with("My")) {
    char* c_name = name.as_c_str();
}
```

String literals retain a trailing NUL for compatibility, but `str.len()` uses
the stored byte length and therefore also handles embedded NULs. Use
`as_c_str()` explicitly for an arbitrary view; only literals retain the old
implicit conversion to `char*`/`char[]`.

`text/inline_string.mln` provides a fixed-capacity, allocation-free owned
string. `N` is the full byte capacity: contents are length-aware and do not
reserve a NUL slot. Appends are atomic and return false without changing the
value when the complete input does not fit. A named struct literal supplies
the empty zero value, so no separate initialization call is needed.

```mln
import str from "text/str.mln";
import { InlineString } from "text/inline_string.mln";

InlineString<128> line = InlineString<128> {};
line.append("pid=");
InlineString<12> number = 42.to_string();
line.append(number.as_str());
```

`str` is only a borrowed view; it is not writable storage. Use
`InlineString<N>` for bounded local/field ownership and, once allocation is
needed, the heap-owned `String` API. NUL-terminated pointers belong at explicit
foreign-system boundaries rather than in ordinary text code.

`text/string.mln` provides growable ownership. Its allocator is selected once
by the executable through `memory/allocator.mln`; MyOS installs the kernel heap
and MyAppFramework installs its process-local `sbrk` heap. A moved String is not
freed by the source binding, and the compiler calls `drop()` for the final
owner automatically.

```mln
import string from "text/string.mln";
import { String } from "text/string.mln";

String line = string.with_capacity(128);
line.append("pid=");
InlineString<12> pid = 42.to_string();
line.append(pid.as_str());
```

`reserve`, `append`, and `push` return false on allocation failure and leave
the existing text intact. `String` is length-aware and does not reserve a NUL
terminator; use `InlineCString<N>` at a `char *` boundary.

`testing/assert.mln` provides generic `assert_eq<T>` / `assert_ne<T>` plus condition,
string, byte-range, pointer, and `Result` assertions. Programs that import it
must provide `assert_fail(char* message)` for their runtime-specific failure
behaviour (for example, printing and halting in a kernel test).

```mln
import assert from "testing/assert.mln";
import { assert_eq } from "testing/assert.mln";

void assert_fail(char* message) {
    // runtime-specific reporting and termination
}

assert.assert_true(ready, "must be ready");
assert_eq<i32>(actual, expected, "unexpected value");
```
