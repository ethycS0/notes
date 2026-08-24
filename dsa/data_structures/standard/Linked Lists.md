# Singly Linked List

**C++ Reference**: [std::forward_list](https://en.cppreference.com/w/cpp/container/forward_list) | [std::list](https://en.cppreference.com/w/cpp/container/list)

## Overview
A linear data structure consisting of nodes where each node contains data and a pointer (`next`) to the next node in the sequence.

- **Unidirectional Traversal**: Traversed only from head to tail.
- **Dynamic Size**: Dynamic memory allocation; nodes do not need contiguous memory.
- **Efficient Insertions/Deletions**: $O(1)$ at head or given node pointer.

## C++ STL Implementation (`std::forward_list`)

- **Header**: `#include <forward_list>`
- **Declaration**: `std::forward_list<T> flist;`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Push Front | `flist.push_front(val)` | $O(1)$ | $O(1)$ |
| Pop Front | `flist.pop_front()` | $O(1)$ | $O(1)$ |
| Insert After Node | `flist.insert_after(pos, val)` | $O(1)$ | $O(1)$ |
| Erase After Node | `flist.erase_after(pos)` | $O(1)$ | $O(1)$ |
| Search / Access | `std::find(flist.begin(), flist.end(), val)` | $O(N)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <forward_list>
#include <algorithm>

int main() {
    std::forward_list<int> flist = {20, 30};

    // Insertions
    flist.push_front(10);                            // [10, 20, 30]
    auto it = flist.begin();                         // points to 10
    flist.insert_after(it, 15);                      // [10, 15, 20, 30]

    // Deletions
    flist.erase_after(it);                          // [10, 20, 30] (removes 15)
    flist.pop_front();                              // [20, 30]

    // Access Head
    int front_val = flist.front();                  // 20
}
```

---

# Doubly Linked List

## Overview
A linear data structure where each node contains data and two pointers (`prev` and `next`), allowing traversal in both directions.

- **Bidirectional Traversal**: Navigable forward and backward.
- **Constant Time Deletion**: Node deletion is $O(1)$ given a direct iterator/pointer to the node.
- **Memory Overhead**: Higher than singly linked list due to storing two pointers per node.

## C++ STL Implementation (`std::list`)

- **Header**: `#include <list>`
- **Declaration**: `std::list<T> lst;`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Push Front / Back | `lst.push_front(val)`, `lst.push_back(val)` | $O(1)$ | $O(1)$ |
| Pop Front / Back | `lst.pop_front()`, `lst.pop_back()` | $O(1)$ | $O(1)$ |
| Insert at Iterator | `lst.insert(pos, val)` | $O(1)$ | $O(1)$ |
| Erase at Iterator | `lst.erase(pos)` | $O(1)$ | $O(1)$ |
| Search / Access | `std::find(lst.begin(), lst.end(), val)` | $O(N)$ | $O(1)$ |
| Size | `lst.size()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <list>
#include <algorithm>

int main() {
    std::list<int> lst = {20, 30};

    // Push Front / Back
    lst.push_front(10);              // [10, 20, 30]
    lst.push_back(40);               // [10, 20, 30, 40]

    // Insert / Erase via Iterator
    auto it = std::find(lst.begin(), lst.end(), 30);
    if (it != lst.end()) {
        lst.insert(it, 25);          // [10, 20, 25, 30, 40]
        lst.erase(it);               // [10, 20, 25, 40] (removes 30)
    }

    // Access Ends
    int head = lst.front();          // 10
    int tail = lst.back();           // 40
}
```
