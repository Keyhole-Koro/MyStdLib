# MyStdLib

Shared MyLang standard library: generic containers (`Vec`, `Slice`, `Arena`,
`RingBuffer`, `IntrusiveList`, `HashMap`, `Option`, `Result`) plus string,
byte, bit-array, and assertion helpers (`str`, `bytes`, `bitset`, `strbuf`,
`assert`).

Used by [MyKernel](https://github.com/Keyhole-Koro/MyKernel),
[MyOS](https://github.com/Keyhole-Koro/MyOS), and exercised by
[MyLangCompiler](https://github.com/Keyhole-Koro/MyLangCompiler)'s generic
import tests. See [GENERICS.md](GENERICS.md) for the generic container API.

Every module is imported by explicit relative path (there is no implicit
prelude): `import { Vec, vec_init, vec_push, vec_pop } from "vec.mln";`.

`assert.mln` provides generic `assert_eq<T>` / `assert_ne<T>` plus condition,
string, byte-range, pointer, and `Result` assertions. Programs that import it
must provide `assert_fail(char* message)` for their runtime-specific failure
behaviour (for example, printing and halting in a kernel test).

```mln
import assert from "assert.mln";
import { assert_eq } from "assert.mln";

void assert_fail(char* message) {
    // runtime-specific reporting and termination
}

assert.assert_true(ready, "must be ready");
assert_eq<i32>(actual, expected, "unexpected value");
```
