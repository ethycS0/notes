
[[c++/effective_modern_c++/INDEX|← Back to Index]]

## Item 1: Understand template type deduction

- During template type deduction, arguments that are references are treated as non-references, i.e., their reference-ness is ignored.
- When deducing types for universal reference parameters, lvalue arguments get special treatment.
- When deducing types for by-value parameters, const and/or `volatile` arguments are treated as non-const and non-`volatile`.
- During template type deduction, arguments that are array or function names decay to pointers, unless they’re used to initialize references.

---

## Item 2: Understand auto type deduction

- `auto` type deduction is usually the same as template type deduction, but `auto` type deduction assumes that a braced initializer represents a `std::initializer_list`, and template type deduction doesn’t.
- `auto` in a function return type or a lambda parameter implies template type deduction, not `auto` type deduction.

---

## Item 3: Understand decltype

- `decltype` almost always yields the type of a variable or expression without any modifications.
- For lvalue expressions of type T other than names, `decltype` always reports a type of T&.
- C++14 supports `decltype(auto)`, which, like `auto`, deduces a type from its initializer, but it performs the type deduction using the `decltype` rules.

---

## Item 4: Know how to view deduced types

- Deduced types can often be seen using IDE editors, compiler error messages, and the Boost TypeIndex library.
- The results of some tools may be neither helpful nor accurate, so an understanding of C++’s type deduction rules remains essential.

---

[[c++/effective_modern_c++/INDEX|← Index]] | [[Chapter 2 - auto|Next Chapter →]]
