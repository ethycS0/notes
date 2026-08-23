# Hash Tables

## Overview
A Hash Table is an associative data structure that maps keys to values using a **Hash Function** to compute an index into an array of buckets.

- **Fast Lookups**: Provides average $O(1)$ time complexity for search, insertion, and deletion.
- **Collision Resolution Strategies**:
  - **Separate Chaining**: Each bucket holds a linked list/bucket of entries that hash to the same index. (Used by C++ STL containers).
  - **Open Addressing**: Search alternative slots in the array upon collision (Linear Probing, Quadratic Probing, Double Hashing).

## C++ STL Implementation (`std::unordered_map`, `std::unordered_set`)

- **Headers**: `#include <unordered_map>`, `#include <unordered_set>`
- **Containers**:
  - `std::unordered_map<Key, Value>`: Stores key-value pairs with unique keys.
  - `std::unordered_set<Key>`: Stores unique keys.

### Standard API & Complexities
| Operation | Function / Method | Average Time | Worst Case Time | Space Complexity |
| :--- | :--- | :--- | :--- | :--- |
| Insertion | `mp[key] = val`, `mp.insert({k, v})`, `st.insert(val)` | $O(1)$ | $O(N)$ | $O(1)$ |
| Deletion | `mp.erase(key)`, `st.erase(val)` | $O(1)$ | $O(N)$ | $O(1)$ |
| Lookup / Find | `mp.find(key)`, `st.count(val)` | $O(1)$ | $O(N)$ | $O(1)$ |
| Element Access | `mp.at(key)`, `mp[key]` | $O(1)$ | $O(N)$ | $O(1)$ |
| Size | `mp.size()`, `st.size()` | $O(1)$ | $O(1)$ | $O(1)$ |

> **Note**: Worst-case $O(N)$ time occurs when hash collisions collapse all keys into a single bucket.

### Code Usage
```cpp
#include <iostream>
#include <unordered_map>
#include <unordered_set>

int main() {
    // --- Unordered Map ---
    std::unordered_map<std::string, int> freq;

    // Insertion & Access
    freq["apple"] = 5;
    freq["banana"] = 3;
    freq.insert({"cherry", 8});

    // Lookup
    auto it = freq.find("apple");
    if (it != freq.end()) {
        std::cout << "apple count: " << it->second << "\n"; // 5
    }

    // Erase
    freq.erase("banana");

    // --- Unordered Set ---
    std::unordered_set<int> visited;
    visited.insert(10);
    visited.insert(20);

    if (visited.count(10)) {
        std::cout << "10 is present in set\n";
    }
}
```
