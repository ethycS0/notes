## Move Semantics

- The problem: returning objects by value in C++98 invokes the copy constructor, which deep copies heap-allocated data — O(n) and expensive.
- RVO (Return Value Optimization): the compiler constructs the return object directly into the caller's memory slot, bypassing copy entirely. No constructor called for the transfer. Available since C++98 but only guaranteed in C++17.
- Move semantics are the fallback when RVO cannot apply (e.g. multiple possible return paths). Instead of copying heap data, the move constructor steals the internal heap pointer from the dying temporary and nulls it out — O(1).
- In memory terms: the stack addresses of both objects stay fixed. What changes is that this->ptr is set to the temporary's heap address, and the temporary's ptr is nulled. The stack slot of the temporary still dies — but you've already taken its heap allocation.
- Priority order for return values: Elision (RVO) > Move > Copy.

## rvalue References

- An rvalue is any temporary value you cannot take the address of: literals, arithmetic expressions (x + 5), or returned temporaries. The defining rule — if &x is illegal, it's an rvalue.
- rvalue references (&&) add no new memory mechanic over raw pointers. Their value is entirely in the type system: they encode "this object is dying" as a contract, enabling two things:
    1. Automatic overload resolution — the compiler picks MyObject(MyObject&&) over MyObject(const MyObject&) when the argument is a temporary.
    2. Safety — && cannot bind to a live lvalue, preventing accidental gutting of objects still in use.
- Reference qualifiers (& and && on member functions) apply the same idea to this, restricting which value category an object must be to call a method.
- Member function reference qualifiers (& and && after the parameter list) overload a method based on the value category of the object itself (*this). A & qualifier restricts the call to lvalue instances; && restricts it to rvalue/temporary instances. Useful when a method should behave differently — or be outright disabled — depending on whether the object is alive or dying.

## auto

- auto variables must be initialized, are generally immune to type mismatches that can lead to portability or efficiency problems, can ease the process of refactoring, and typically require less typing than variables with explicitly specified types.

## Most Vexing Parse

- The most vexing parse is a counterintuitive syntax ambiguity resolution rule in C++. Under this rule, anything that can be interpreted by the compiler as a function declaration will be interpreted as a function declaration, even if the programmer intended it to be an object initialization.

## Uniform/Braced Initialization

- Braced initialization is the most widely usable initialization syntax, it prevents narrowing conversions, and it’s immune to C++’s most vexing parse.
- During constructor overload resolution, braced initializers are matched to std::initializer_list parameters if at all possible, even if other construc‐ tors offer seemingly better matches.
- An example of where the choice between parentheses and braces can make a significant difference is creating a std::vector with two arguments.
- Choosing between parentheses and braces for object creation inside templates can be challenging.

## Static

- Inside a function: Static Local Variable
- File/Namspace Scope: Per File internal linkage
- Inside a class: Static Member Variables
- Inside a class: Static Member Function

## CRTP

- The Curiously Recurring Template Pattern (CRTP) is a C++ idiom where a class inherits from a template class, using itself as the template argument. 
- It is primarily used to achieve static polymorphism (compile-time method dispatch), eliminating the runtime overhead, vtables, and pointer-indirection penalties associated with traditional virtual functions.
## RAII

- Resource Acquisition Is Initialization (RAII) is a core C++ programming idiom that binds the lifecycle of a limited resource to the lifetime of a local stack object. 
- Under RAII, a resource is automatically acquired in the object's constructor and automatically released in its destructor. 
- This design pattern guarantees that resources are never leaked, even if a function returns early or throws an exception.
## PIMPL

- PIMPL stands for Pointer to IMPLementation. It is a design pattern used in C++ to hide a class's internal data members and private methods behind a forward-declared pointer.