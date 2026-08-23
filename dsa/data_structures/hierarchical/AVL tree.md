# AVL Tree

## Overview
An AVL Tree (named after inventors Adelson-Velsky and Landis) is a self-balancing Binary Search Tree (BST) where the height difference between left and right subtrees of any node is at most 1.

## Balance Factor ($BF$)
For any node $N$ in an AVL tree:
$$\text{BF}(N) = \text{Height}(\text{Left Subtree}) - \text{Height}(\text{Right Subtree}) \in \{-1, 0, 1\}$$

If $|\text{BF}(N)| > 1$, the node is unbalanced and must be restored using **tree rotations**.

## Tree Rotations (Rebalancing)
| Imbalance Type | Description | Rebalancing Action |
| :--- | :--- | :--- |
| **LL Case** | Inserted in left subtree of left child | Single **Right Rotation** |
| **RR Case** | Inserted in right subtree of right child | Single **Left Rotation** |
| **LR Case** | Inserted in right subtree of left child | **Left Rotation** on left child, then **Right Rotation** on root |
| **RL Case** | Inserted in left subtree of right child | **Right Rotation** on right child, then **Left Rotation** on root |

## Time & Space Complexities
| Operation | Average Time | Worst Case | Space Complexity |
| :--- | :--- | :--- | :--- |
| Search | $O(\log N)$ | $O(\log N)$ | $O(\log N)$ |
| Insertion | $O(\log N)$ | $O(\log N)$ | $O(\log N)$ |
| Deletion | $O(\log N)$ | $O(\log N)$ | $O(\log N)$ |

## Comparison with Red-Black Trees
- **AVL Tree**: More strictly balanced ($h \le 1.44 \log_2 N$). Faster lookup speeds; slightly slower insertions/deletions due to frequent rebalancing rotations.
- **Red-Black Tree**: Slightly looser balancing ($h \le 2 \log_2 N$). Faster insertions/deletions (fewer rotations); standard choice for C++ STL (`std::set`, `std::map`).

## C++ Implementation Snippet (Rotations)
```cpp
#include <iostream>
#include <algorithm>

struct AVLNode {
    int key, height;
    AVLNode *left, *right;
    AVLNode(int k) : key(k), height(1), left(nullptr), right(nullptr) {}
};

int getHeight(AVLNode* n) { return n ? n->height : 0; }
int getBalance(AVLNode* n) { return n ? getHeight(n->left) - getHeight(n->right) : 0; }

AVLNode* rightRotate(AVLNode* y) {
    AVLNode* x = y->left;
    AVLNode* T2 = x->right;

    x->right = y;
    y->left = T2;

    y->height = std::max(getHeight(y->left), getHeight(y->right)) + 1;
    x->height = std::max(getHeight(x->left), getHeight(x->right)) + 1;

    return x; // New root
}

AVLNode* leftRotate(AVLNode* x) {
    AVLNode* y = x->right;
    AVLNode* T2 = y->left;

    y->left = x;
    x->right = T2;

    x->height = std::max(getHeight(x->left), getHeight(x->right)) + 1;
    y->height = std::max(getHeight(y->left), getHeight(y->right)) + 1;

    return y; // New root
}
```
