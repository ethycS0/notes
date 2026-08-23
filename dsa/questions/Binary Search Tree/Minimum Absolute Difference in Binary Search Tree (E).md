Previous: [[Lowest Common Ancestor in Binary Search Tree (M)]] | Next: [[Balance a Binary Search Tree (M)]]

Given the root of a Binary Search Tree (BST), return the minimum absolute difference between the values of any two different nodes in the tree.

## Solution

This requires inorder traversal. You may feel that this may be solved with comparison between children or nodes but nope, always inorder and compare with the previous. This is a result of the BST design, closest numbers with Minimum Absolute Difference are inorder search.

```C++
class Solution {
private:
    int min_diff = INT_MAX;
    int prev = -1;

    void inorder(TreeNode* node) {
        if (!node) {
            return;
        }

        inorder(node->left);

        if (prev != -1) {
            min_diff = std::min(min_diff, node->val - prev);
        }
        prev = node->val;

        inorder(node->right);
    }

public:
    int getMinimumDifference(TreeNode* root) {
        inorder(root);
        return min_diff;
    }
};
```

