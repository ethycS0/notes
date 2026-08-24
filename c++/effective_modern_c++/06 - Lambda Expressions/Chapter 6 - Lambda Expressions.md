
[[INDEX|← Back to Index]]

## Item 31: Avoid default capture modes

- Default by-reference capture can lead to dangling references.
- Default by-value capture is susceptible to dangling pointers (especially this), and it misleadingly suggests that lambdas are self-contained.

---

## Item 32: Use init capture to move objects into closures

- Use C++14’s init capture to move objects into closures.
- In C++11, emulate init capture via hand-written classes or `std::bind`.

---

## Item 33: Use decltype on auto&& parameters to std::forward them

- Use `decltype` on `auto&&` parameters to `std::forward` them.

---

## Item 34: Prefer lambdas to std::bind

- Lambdas are more readable, more expressive, and may be more efficient than using `std::bind`.
- In C++11 only, `std::bind` may be useful for implementing move capture or for binding objects with templatized function call operators. | Item 34

---

[[INDEX|← Index]] | [[Chapter 5 - Rvalue References Move Semantics and Perfect Forwarding|<- Previous Chapter]] | [[Chapter 7 - The Concurrency API|Next Chapter →]]
