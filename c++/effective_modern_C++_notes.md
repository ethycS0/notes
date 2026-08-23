## Move Semantics

- The problem: returning objects by value in C++98 invokes the copy constructor,
  which deep copies heap-allocated data — O(n) and expensive.
- RVO (Return Value Optimization): the compiler constructs the return object
  directly into the caller's memory slot, bypassing copy entirely. No constructor
  called for the transfer. Available since C++98 but only guaranteed in C++17.
- Move semantics are the fallback when RVO cannot apply (e.g. multiple possible
  return paths). Instead of copying heap data, the move constructor steals the
  internal heap pointer from the dying temporary and nulls it out — O(1).
- In memory terms: the stack addresses of both objects stay fixed. What changes
  is that this->ptr is set to the temporary's heap address, and the temporary's
  ptr is nulled. The stack slot of the temporary still dies — but you've already
  taken its heap allocation.
- Priority order for return values: Elision (RVO) > Move > Copy.

## rvalue References

- An rvalue is any temporary value you cannot take the address of: literals,
  arithmetic expressions (x + 5), or returned temporaries. The defining rule —
  if &x is illegal, it's an rvalue.
- rvalue references (&&) add no new memory mechanic over raw pointers. Their
  value is entirely in the type system: they encode "this object is dying" as a
  contract, enabling two things:
  1. Automatic overload resolution — the compiler picks MyObject(MyObject&&)
     over MyObject(const MyObject&) when the argument is a temporary.
  2. Safety — && cannot bind to a live lvalue, preventing accidental gutting
     of objects still in use.
- Reference qualifiers (& and && on member functions) apply the same idea to
  \*this, restricting which value category an object must be to call a method.
- Member function reference qualifiers (& and && after the parameter list) overload
  a method based on the value category of the object itself (\*this). A & qualifier
  restricts the call to lvalue instances; && restricts it to rvalue/temporary
  instances. Useful when a method should behave differently — or be outright
  disabled — depending on whether the object is alive or dying.

## Auto

- auto variables must be initialized, are generally immune to type mismatches
  that can lead to portability or efficiency problems, can ease the process of
  refactoring, and typically require less typing than variables with explicitly
  specified types.
- auto-typed variables are subject to the pitfalls described in Items 2 and 6.

## Uniform/Braced Initialization

- Braced initialization is the most widely usable initialization syntax, it prevents
  narrowing conversions, and it’s immune to C++’s most vexing parse.
- During constructor overload resolution, braced initializers are matched to
  std::initializer_list parameters if at all possible, even if other construc‐
  tors offer seemingly better matches.
- An example of where the choice between parentheses and braces can make a
  significant difference is creating a std::vector<numeric type> with two
  arguments.
- Choosing between parentheses and braces for object creation inside templates
  can be challenging.

## Overloading

- Cannot overload ont return types alone

## nullptr

- Prefer nullptr to 0 and NULL
- Avoid overloading on integral and pointer types.

## Scoped Enums

- C++98-style enums are now known as unscoped enums.
- Enumerators of scoped enums are visible only within the enum. They convert
  to other types only with a cast.
- Both scoped and unscoped enums support specification of the underlying type.
  The default underlying type for scoped enums is int. Unscoped enums have no
  default underlying type.
- Scoped enums may always be forward-declared. Unscoped enums may be
  forward-declared only if their declaration specifies an underlying type.

## Virtual & Runtime Polymorphism

- Polymorphism: Enables a base pointer to call derived-specific behavior; must
  happen at runtime because the actual object type is often decided by unpredictable
  runtime logic.
- virtual Keyword: Tells the compiler to bypass static binding (fixed compile-time
  addresses) and defer function resolution to runtime (dynamic binding).
  = VTable (Virtual Table): A single compiler-generated lookup table per class containing
  pointers to its virtual function overrides.
- VPTR (Virtual Pointer): A hidden pointer added to every object instance that links it
  to its class's VTable, increasing object size by one pointer.
- The Lookup Method: On a virtual call, the program grabs the object's VPTR, finds
  the function's index in the VTable, and jumps to that specific address.

## override

- Declare overriding functions override.
- Member function reference qualifiers make it possible to treat lvalue and
  rvalue objects (\*this) differently

## unique_ptr

- A raw pointer wrapped in a struct whose destructor calls delete automatically
  when it goes out of scope. Zero overhead — same size as a raw pointer on the stack.
- Single ownership enforced at compile time — copy constructor is deleted, only
  std::move() transfers ownership, nulling the source.
- Scope follows standard C rules — any {}. Destructor fires the moment execution
  leaves the declaring block: functions, if/for/while bodies, class, namespace, global.
- Early returns and exceptions no longer cause leaks — destructor fires regardless
  of how scope is exited.
- std::unique_ptr is a small, fast, move-only smart pointer for managing
  resources with exclusive-ownership semantics.
- By default, resource destruction takes place via delete, but custom deleters
  can be specified. Stateful deleters and function pointers as deleters increase the
  size of std::unique_ptr objects.
- Converting a std::unique_ptr to a std::shared_ptr is easy

# Items Done

5, 7, 8, 10, 12, 18, 21

# Important Words

- Narrowing Conversions
- Most Vexing Parse
- Type Inference
- CRTP
- RVO
- Factory Function
- RAII
