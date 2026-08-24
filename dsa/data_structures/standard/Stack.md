# Stack

**C++ Reference**: [std::stack](https://en.cppreference.com/w/cpp/container/stack)

## Overview
A Stack is a linear data structure operating on the **LIFO (Last-In, First-Out)** principle. Elements are added (pushed) and removed (popped) from the same end, referred to as the **top**.

- **Single Access Point**: Elements can only be inserted, accessed, or deleted from the top.
- **Common Applications**: Function call stack, backtracking (DFS), expression evaluation & parsing, undo/redo mechanisms.

## C++ STL Implementation (`std::stack`)

- **Header**: `#include <stack>`
- **Declaration**: `std::stack<T>` (Container adapter; defaults to `std::deque<T>`, can also adapt `std::vector<T>` or `std::list<T>`).

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Push Top | `stk.push(val)`, `stk.emplace(val)` | $O(1)$ | $O(1)$ |
| Pop Top | `stk.pop()` | $O(1)$ | $O(1)$ |
| Read Top Element | `stk.top()` | $O(1)$ | $O(1)$ |
| Check Empty | `stk.empty()` | $O(1)$ | $O(1)$ |
| Element Count | `stk.size()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> stk;

    // Push elements
    stk.push(10);
    stk.push(20);
    stk.push(30); // Stack top is 30

    // Inspect Top
    std::cout << "Top element: " << stk.top() << "\n"; // 30

    // Pop element
    stk.pop(); // Removes 30; top becomes 20

    // Process Stack
    while (!stk.empty()) {
        std::cout << stk.top() << " ";
        stk.pop();
    }
    // Output: 20 10
}
```
