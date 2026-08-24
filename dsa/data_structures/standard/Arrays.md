# Static Arrays

**C++ Reference**: [std::array](https://en.cppreference.com/w/cpp/container/array) | [std::vector](https://en.cppreference.com/w/cpp/container/vector)

## Overview
A static array is a contiguous memory allocation storing elements of identical data types with a fixed size determined at compile time.

- **Direct Index Access**: $O(1)$ via memory offset computation ($\text{address} = \text{base} + i \times \text{element\_size}$).
- **Memory Overhead**: Minimal (no pointer overhead).
- **Size**: Fixed upon creation; cannot be resized dynamically.

## C++ STL Implementation (`std::array`)

- **Header**: `#include <array>`
- **Declaration**: `std::array<T, N> arr;`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Random Access | `arr[i]`, `arr.at(i)` | $O(1)$ | $O(1)$ |
| Access First/Last | `arr.front()`, `arr.back()` | $O(1)$ | $O(1)$ |
| Size | `arr.size()` | $O(1)$ | $O(1)$ |
| Search | `std::find(arr.begin(), arr.end(), val)` | $O(N)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <array>
#include <algorithm>

int main() {
    std::array<int, 5> arr = {10, 20, 30, 40, 50};

    // Access
    int val = arr[2];          // 30 (no bounds check)
    int safe_val = arr.at(2);   // 30 (throws std::out_of_range)

    // Linear Search
    auto it = std::find(arr.begin(), arr.end(), 30);
    if (it != arr.end()) {
        std::cout << "Found at index: " << std::distance(arr.begin(), it) << "\n";
    }
}
```

---

# Dynamic Arrays

## Overview
A dynamic array is a contiguous block of memory that automatically resizes when capacity is exceeded (typically doubling capacity $N \to 2N$).

- **Dynamic Resizing**: Automatic reallocation and element copying when full.
- **Amortized Constant Insertion**: Inserting at the end is amortized $O(1)$.
- **Cache Friendly**: High spatial locality due to contiguous memory layout.

## C++ STL Implementation (`std::vector`)

- **Header**: `#include <vector>`
- **Declaration**: `std::vector<T> vec;`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Random Access | `vec[i]`, `vec.at(i)` | $O(1)$ | $O(1)$ |
| Insert at End | `vec.push_back(val)`, `vec.emplace_back(val)` | Amortized $O(1)$ | $O(1)$ |
| Delete from End | `vec.pop_back()` | $O(1)$ | $O(1)$ |
| Insert Arbitrary | `vec.insert(pos, val)` | $O(N)$ | $O(1)$ |
| Delete Arbitrary | `vec.erase(pos)` | $O(N)$ | $O(1)$ |
| Linear Search | `std::find(vec.begin(), vec.end(), val)` | $O(N)$ | $O(1)$ |
| Size / Capacity | `vec.size()`, `vec.capacity()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {10, 20, 30};

    // Insert & Delete
    vec.push_back(40);                // [10, 20, 30, 40]
    vec.insert(vec.begin() + 1, 15);  // [10, 15, 20, 30, 40]
    vec.erase(vec.begin() + 1);       // [10, 20, 30, 40]
    vec.pop_back();                   // [10, 20, 30]

    // Find
    auto it = std::find(vec.begin(), vec.end(), 20);
    if (it != vec.end()) {
        std::cout << "Found 20 at index " << (it - vec.begin()) << "\n";
    }
}
```