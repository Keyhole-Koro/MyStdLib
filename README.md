# MyStdLib

Shared MyLang standard library: generic containers (`Vec`, `Slice`, `Arena`,
`RingBuffer`, `IntrusiveList`, `HashMap`, `Option`, `Result`) plus string,
byte, and bit-array helpers (`str`, `bytes`, `bitset`, `strbuf`).

Used by [MyKernel](https://github.com/Keyhole-Koro/MyKernel),
[MyOS](https://github.com/Keyhole-Koro/MyOS), and exercised by
[MyLangCompiler](https://github.com/Keyhole-Koro/MyLangCompiler)'s generic
import tests. See [GENERICS.md](GENERICS.md) for the generic container API.

Every module is imported by explicit relative path (there is no implicit
prelude): `import { Vec, vec_init, vec_push, vec_pop } from "vec.mln";`.
