# Generic containers

Each of these modules is independently importable through the compiler's
named generic import support. The containers are **methods on the type**:
importing the type is all it takes -- its methods come with it, and there
is no per-function import and no explicit `<T>` at the call site.

```mln
import { Vec } from "collections/vec.mln";

i32 main() {
    i32 storage[8];
    Vec<i32> values;
    values.init(&storage[0], storage.length);
    values.push(42);
    i32 value = 0;
    values.pop(&value);
    return value;
}
```

| Type | Methods |
| --- | --- |
| `Slice<T>` | `init(data, len)`, `len()`, `at(i)` → `T *` or 0 |
| `Vec<T>` | `init(storage, cap)`, `len()`, `push(v)` → bool, `pop(&out)` → bool, `at(i)` |
| `Arena<T>` | `init(storage, cap)`, `reset()`, `alloc(count)` → `Option<T *>` |
| `RingBuffer<T>` | `init(storage, cap)`, `capacity()`, `count()`, `has()`, `full()`, `push(v)`, `pop(&out)`, `peek(&out)`, `clear()` |
| `HashMap<K, V>` | `init(keys, values, used, cap)`, `put(key, v)` → bool, `get(&key, &out)` → bool |
| `IntrusiveList<T>` | `init()`, `push_front(node, &node->next)`, `pop_front(&head->next)` |

| Module | Contents |
| --- | --- |
| `memory/slice.mln` | `Slice<T>` |
| `collections/vec.mln` | `Vec<T>` |
| `memory/arena.mln` | `Arena<T>` |
| `collections/ringbuf.mln` | `RingBuffer<T>` (also the serial input queue's implementation) |
| `collections/intrusive_list.mln` | `IntrusiveList<T>` |
| `core/option.mln` | payload-enum `Option<T>` (`Some(T)` / `None`) |
| `core/result.mln` | payload-enum `Result<T,E>` (`Ok(T)` / `Err(E)`) |
| `collections/hashmap.mln` | `HashMap<K, V>` for `K: Hash + Eq` |
| `testing/assert.mln` | condition assertions plus generic `assert_eq<T>` / `assert_ne<T>` and `Result<T,E>` checks |

`Slice<T>`, `Vec<T>`, `Arena<T>`, and `RingBuffer<T>` operate on storage passed
by the caller. `IntrusiveList<T>` is singly linked; each push/pop receives the
address of the payload node's next-link field. `Option<T>` and `Result<T,E>`
are real payload enums built with `Some(value)` / `None` and `Ok(value)` /
`Err(error)`, and should be inspected with `case ... of { ... }`.
`HashMap<K, V>` maps keys implementing `Hash + Eq` to `V` values with
caller-provided key, value, and `u8` occupancy arrays. It has insertion,
replacement, and lookup but not deletion or automatic growth. Its capacity must be
a power of two. The separate occupancy array means zero remains a valid key.
`put` takes ownership of non-Copy keys and values; `get` reads its key through
a non-owning pointer.

`testing/assert.mln` requires each runtime to provide `assert_fail(char* message)`. It
uses the existing `==` / `!=` operators, so `assert_eq<T>` and `assert_ne<T>`
are valid for types those operators support after generic instantiation.

Current code generation is reliable for scalar and pointer element types.
Passing aggregate values by value remains outside the container API contract.
