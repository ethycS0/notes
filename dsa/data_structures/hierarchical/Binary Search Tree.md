# Binary Search Tree

## Overview
A Binary Search Tree (BST) is a node-based binary tree data structure maintaining the **BST Invariant**:
- Left subtree of a node contains only nodes with keys **less than** the node's key.
- Right subtree of a node contains only nodes with keys **greater than** the node's key.
- Left and right subtrees must also be binary search trees.
- **Inorder Traversal** of a BST produces sorted elements in ascending order.

## Deletion Cases
1. **Leaf Node**: Remove directly.
2. **Node with 1 Child**: Replace node with its child.
3. **Node with 2 Children**: Replace node's key with its **Inorder Successor** (smallest element in right subtree) or **Inorder Predecessor** (largest element in left subtree), then recursively delete that successor/predecessor node.

## Time & Space Complexities
| Operation | Average Time | Worst Case (Skewed) | Space Complexity |
| :--- | :--- | :--- | :--- |
| Search | $O(\log N)$ | $O(N)$ | $O(H)$ |
| Insertion | $O(\log N)$ | $O(N)$ | $O(H)$ |
| Deletion | $O(\log N)$ | $O(N)$ | $O(H)$ |
| Minimum / Maximum | $O(\log N)$ | $O(N)$ | $O(1)$ |

## C++ STL Implementation (`std::set`, `std::map`)
The C++ STL provides ordered associative containers implemented as self-balancing BSTs (**Red-Black Trees**):
- `std::set<T>`: Unique sorted keys.
- `std::map<Key, Value>`: Unique sorted key-value pairs.
- `std::multiset<T>` / `std::multimap<Key, Value>`: Allows duplicate keys.

- **Headers**: `#include <set>`, `#include <map>`

### Standard API & Complexities
| Operation | Function / Method | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| Insert | `st.insert(val)`, `mp[key] = val` | $O(\log N)$ | $O(1)$ |
| Find / Search | `st.find(val)`, `mp.find(key)` | $O(\log N)$ | $O(1)$ |
| Delete | `st.erase(val)`, `mp.erase(key)` | $O(\log N)$ | $O(1)$ |
| Bound Search | `st.lower_bound(val)`, `st.upper_bound(val)` | $O(\log N)$ | $O(1)$ |
| Min / Max Element | `*st.begin()`, `*st.rbegin()` | $O(1)$ | $O(1)$ |

### Code Usage
```cpp
#include <iostream>
#include <set>
#include <map>

int main() {
    std::set<int> bst;

    // Insertion
    bst.insert(50);
    bst.insert(30);
    bst.insert(70);

    // Lookup
    auto it = bst.find(30);
    if (it != bst.end()) {
        std::cout << "Found: " << *it << "\n";
    }

    // Range operations (Lower / Upper Bound)
    auto lb = bst.lower_bound(40); // Points to 50 (first element >= 40)
    std::cout << "Lower bound >= 40: " << *lb << "\n";

    // Deletion
    bst.erase(30);
}
```
