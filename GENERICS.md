# Generic containers

Each of these modules is independently importable through the compiler's
named generic import support. The containers are **methods on the type**:
importing the type is all it takes -- its methods come with it, and there
is no per-function import and no explicit `<T>` at the call site.

```mln
import { Vec } from "vec.mln";

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
| `Arena<T>` | `init(storage, cap)`, `reset()`, `alloc(count)` → `T *` or 0 |
| `RingBuffer<T>` | `init(storage, cap)`, `capacity()`, `count()`, `has()`, `full()`, `push(v)`, `pop(&out)`, `peek(&out)`, `clear()` |
| `HashMap<T>` | `init(keys, values, cap)`, `put(key, v)` → bool, `get(key, &out)` → bool |
| `IntrusiveList<T>` | `init()`, `push_front(node, &node->next)`, `pop_front(&head->next)` |

| Module | Contents |
| --- | --- |
| `slice.mln` | `Slice<T>` |
| `vec.mln` | `Vec<T>` |
| `arena.mln` | `Arena<T>` |
| `ringbuf.mln` | `RingBuffer<T>` (also the serial input queue's implementation) |
| `intrusive_list.mln` | `IntrusiveList<T>` |
| `option.mln` | payload-enum `Option<T>` (`Some(T)` / `None`) |
| `result.mln` | payload-enum `Result<T,E>` (`Ok(T)` / `Err(E)`) |
| `hashmap.mln` | string-keyed `HashMap<T>` |
| `assert.mln` | condition assertions plus generic `assert_eq<T>` / `assert_ne<T>` and `Result<T,E>` checks |

`Slice<T>`, `Vec<T>`, `Arena<T>`, and `RingBuffer<T>` operate on storage passed
by the caller. `IntrusiveList<T>` is singly linked; each push/pop receives the
address of the payload node's next-link field. `Option<T>` and `Result<T,E>`
are real payload enums built with `Some(value)` / `None` and `Ok(value)` /
`Err(error)`, matched with `case ... of { ... }`; `option_take()` and
`result_take_ok()`/`result_take_err()` are convenience wrappers for the
common "check and consume" shape.
`HashMap<T>` maps borrowed NUL-terminated string keys to `T` values using
caller-provided `i32` key-address and value arrays; it has insertion, replacement,
and lookup but not deletion or automatic growth. Its capacity must be a power of two.

`assert.mln` requires each runtime to provide `assert_fail(char* message)`. It
uses the existing `==` / `!=` operators, so `assert_eq<T>` and `assert_ne<T>`
are valid for types those operators support after generic instantiation.

Current code generation is reliable for scalar and pointer element types.
Passing aggregate values by value remains outside the container API contract.
