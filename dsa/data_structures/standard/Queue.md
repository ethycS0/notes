# Queue

**C++ Reference**: [std::queue](https://en.cppreference.com/w/cpp/container/queue) | [std::deque](https://en.cppreference.com/w/cpp/container/deque)

## Overview
A Queue is a linear data structure following the **FIFO (First-In, First-Out)** principle. Elements are inserted at the **rear (back)** and removed from the **front**.

- **Two Access Points**: Enqueue at rear, Dequeue from front.
- **Common Applications**: Task/Job scheduling (CPU, Printer), Breadth-First Search (BFS), network packet buffering.

## C++ STL Implementation (`std::queue`)

- **Header**: `#include <queue>`
- **Declaration**: `std::queue<T>` (Container adapter; defaults to `std::deque<T>`).

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Enqueue (Push Back) | `q.push(val)`, `q.emplace(val)` | $O(1)$ | $O(1)$ |
| Dequeue (Pop Front) | `q.pop()` | $O(1)$ | $O(1)$ |
| Access Front | `q.front()` | $O(1)$ | $O(1)$ |
| Access Rear | `q.back()` | $O(1)$ | $O(1)$ |
| Check Empty / Size | `q.empty()`, `q.size()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;

    // Enqueue
    q.push(10);
    q.push(20);
    q.push(30);

    // Access ends
    std::cout << "Front: " << q.front() << ", Back: " << q.back() << "\n"; // Front: 10, Back: 30

    // Dequeue
    q.pop(); // Removes 10

    // Traverse Queue
    while (!q.empty()) {
        std::cout << q.front() << " ";
        q.pop();
    }
    // Output: 20 30
}
```

---

# Deque

## Overview
A **Deque (Double-Ended Queue)** allows efficient insertion and deletion at both the front and rear ends.

- **Hybrid Container**: Combines direct indexing capabilities of dynamic arrays with constant-time double-ended operations.
- **Memory Layout**: Implemented as a sequence of fixed-size chunks/pages managed by a central map array.

## C++ STL Implementation (`std::deque`)

- **Header**: `#include <deque>`
- **Declaration**: `std::deque<T> dq;`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Push Front / Back | `dq.push_front(val)`, `dq.push_back(val)` | $O(1)$ | $O(1)$ |
| Pop Front / Back | `dq.pop_front()`, `dq.pop_back()` | $O(1)$ | $O(1)$ |
| Random Access | `dq[i]`, `dq.at(i)` | $O(1)$ | $O(1)$ |
| Insert / Erase | `dq.insert(pos, val)`, `dq.erase(pos)` | $O(N)$ | $O(1)$ |
| Size | `dq.size()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> dq = {20, 30};

    // Insert at both ends
    dq.push_front(10); // [10, 20, 30]
    dq.push_back(40);  // [10, 20, 30, 40]

    // Random Access
    std::cout << "Element at index 2: " << dq[2] << "\n"; // 30

    // Remove from both ends
    dq.pop_front(); // [20, 30, 40]
    dq.pop_back();  // [20, 30]
}
```