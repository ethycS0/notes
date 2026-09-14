
[[c++/effective_modern_c++/INDEX|← Back to Index]]

## Item 18: Use std::unique_ptr for exclusive-ownership resource management

- `std::unique_ptr` is a small, fast, move-only smart pointer for managing resources with exclusive-ownership semantics.
- By default, resource destruction takes place via delete, but custom deleters can be specified. Stateful deleters and function pointers as deleters increase the size of `std::unique_ptr` objects.
- Converting a `std::unique_ptr` to a `std::shared_ptr` is easy. 

---

## Item 19: Use std::shared_ptr for shared-ownership resource management

- `std::shared_ptr`s offer convenience approaching that of garbage collection for the shared lifetime management of arbitrary resources.
- Compared to `std::unique_ptr`, `std::shared_ptr` objects are typically twice as big, incur overhead for control blocks, and require atomic reference count manipulations.
- Default resource destruction is via delete, but custom deleters are supported. The type of the deleter has no effect on the type of the `std::shared_ptr`.
- Avoid creating `std::shared_ptr`s from variables of raw pointer type.

---

## Item 20: Use std::weak_ptr for std::shared_ptr-like pointers that can dangle

- Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle.
- Potential use cases for `std::weak_ptr` include caching, observer lists, and the prevention of `std::shared_ptr` cycles.

---

## Item 21: Prefer std::make_unique and std::make_shared to direct use of new

- Compared to direct use of new, make functions eliminate source code duplication, improve exception safety, and, for `std::make_shared` and std::allo cate_shared, generate code that’s smaller and faster.
- Situations where use of make functions is inappropriate include the need to specify custom deleters and a desire to pass braced initializers.
- For `std::shared_ptr`s, additional situations where make functions may be ill-advised include (1) classes with custom memory management and (2) systems with memory concerns, very large objects, and std::weak_ptrs that outlive the corresponding `std::shared_ptr`s.

---

## Item 22: When using the Pimpl Idiom, define special member functions in the implementation file

- The Pimpl Idiom decreases build times by reducing compilation dependencies between class clients and class implementations.
- For `std::unique_ptr` pImpl pointers, declare special member functions in the class header, but implement them in the implementation file. Do this even if the default function implementations are acceptable.
- The above advice applies to `std::unique_ptr`, but not to `std::shared_ptr`.

---

[[c++/effective_modern_c++/INDEX|← Index]] | [[Chapter 3 - Moving to Modern C++|← Previous Chapter]] | [[Chapter 5 - Rvalue References Move Semantics and Perfect Forwarding|Next Chapter →]]
