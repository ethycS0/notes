# Min Heap

**C++ Reference**: [std::priority_queue](https://en.cppreference.com/w/cpp/container/priority_queue) | [std::make_heap](https://en.cppreference.com/w/cpp/algorithm/make_heap)

## Overview
A Min Heap is a Complete Binary Tree where the key at the parent node is less than or equal to the keys of its children.

- **Root Value**: Contains the minimum element of the dataset.
- **Array Indexing (0-indexed)**:
  - Parent: `(i - 1) / 2`
  - Left Child: `2 * i + 1`
  - Right Child: `2 * i + 2`

## C++ STL Implementation (`std::priority_queue`)

- **Header**: `#include <queue>`
- **Declaration**: `std::priority_queue<T, std::vector<T>, std::greater<T>> min_pq;`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Peek Min | `min_pq.top()` | $O(1)$ | $O(1)$ |
| Push (Insert) | `min_pq.push(val)`, `min_pq.emplace(val)` | $O(\log N)$ | $O(1)$ |
| Pop Min (Delete) | `min_pq.pop()` | $O(\log N)$ | $O(1)$ |
| Build Heap | Constructor with range / `std::make_heap` | $O(N)$ | $O(1)$ |
| Size / Empty | `min_pq.size()`, `min_pq.empty()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <queue>
#include <vector>

int main() {
    // Min Heap Declaration
    std::priority_queue<int, std::vector<int>, std::greater<int>> min_heap;

    min_heap.push(30);
    min_heap.push(10);
    min_heap.push(20);

    // Min element at top
    std::cout << "Min Element: " << min_heap.top() << "\n"; // 10

    min_heap.pop(); // Removes 10
    std::cout << "Next Min: " << min_heap.top() << "\n";    // 20
}
```

---

# Max Heap

## Overview
A Max Heap is a Complete Binary Tree where the key at the parent node is greater than or equal to the keys of its children.

- **Root Value**: Contains the maximum element of the dataset.
- **Common Applications**: Priority queues, Heap Sort ($O(N \log N)$), Dijkstra's algorithm, Top $K$ elements problems.

## C++ STL Implementation (`std::priority_queue`)

- **Header**: `#include <queue>`
- **Declaration**: `std::priority_queue<T> max_pq;` (Default behavior)

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Peek Max | `max_pq.top()` | $O(1)$ | $O(1)$ |
| Push (Insert) | `max_pq.push(val)`, `max_pq.emplace(val)` | $O(\log N)$ | $O(1)$ |
| Pop Max (Delete) | `max_pq.pop()` | $O(\log N)$ | $O(1)$ |
| Build Heap | Constructor with range / `std::make_heap` | $O(N)$ | $O(1)$ |
| Size / Empty | `max_pq.size()`, `max_pq.empty()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <queue>

int main() {
    // Max Heap Declaration (Default)
    std::priority_queue<int> max_heap;

    max_heap.push(30);
    max_heap.push(10);
    max_heap.push(50);

    // Max element at top
    std::cout << "Max Element: " << max_heap.top() << "\n"; // 50

    max_heap.pop(); // Removes 50
    std::cout << "Next Max: " << max_heap.top() << "\n";    // 30
}
```
