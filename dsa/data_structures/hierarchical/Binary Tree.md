# Binary Tree

**C++ Reference**: [std::queue](https://en.cppreference.com/w/cpp/container/queue)

## Overview
A Binary Tree is a non-linear hierarchical data structure in which each node has at most two children, referred to as the **left child** and **right child**.

## Core Terminology & Properties
- **Root**: Topmost node of the tree.
- **Leaf Node**: Node with no children.
- **Height ($h$)**: Number of edges on the longest path from root to a leaf.
- **Max Nodes at Level $l$**: $2^l$ (assuming root is level 0).
- **Max Total Nodes**: $2^{h+1} - 1$ for a tree of height $h$.

## Types of Binary Trees
1. **Full Binary Tree**: Every node has either 0 or 2 children (no node has 1 child).
2. **Complete Binary Tree**: All levels are completely filled except possibly the last level, which is filled from left to right.
3. **Perfect Binary Tree**: All internal nodes have 2 children and all leaf nodes are at the same level.
4. **Balanced Binary Tree**: Height of left and right subtrees of every node differs by at most 1 (guarantees $O(\log N)$ height).
5. **Degenerate / Skewed Tree**: Every parent node has only one child, behaving like a singly linked list ($O(N)$ height).

## Traversals
- **Depth-First Search (DFS)**:
  - *Inorder (Left, Root, Right)*: Produces sorted order in BSTs.
  - *Preorder (Root, Left, Right)*: Used for tree serialization/copying.
  - *Postorder (Left, Right, Root)*: Used for tree deletion and bottom-up evaluation.
- **Breadth-First Search (BFS)**:
  - *Level Order*: Traverses level by level using a Queue ($O(N)$ time, $O(W)$ max-width space).

## C++ Representation & STL Usage
C++ standard library does not provide a direct `std::binary_tree` container. Binary trees are implemented using a custom `Node` structure or represented via `std::vector` (for complete trees).

### Operations & Complexities
| Operation | Time Complexity | Space Complexity |
| :--- | :--- | :--- |
| Traversal (DFS/BFS) | $O(N)$ | $O(H)$ or $O(W)$ |
| Search (Unordered) | $O(N)$ | $O(H)$ |
| Insertion (at available leaf) | $O(N)$ | $O(N)$ |

### Code Usage
```cpp
#include <iostream>
#include <queue>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// Level Order Traversal using std::queue
void levelOrder(TreeNode* root) {
    if (!root) return;
    std::queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        TreeNode* curr = q.front();
        q.pop();
        std::cout << curr->val << " ";

        if (curr->left)  q.push(curr->left);
        if (curr->right) q.push(curr->right);
    }
}
```
