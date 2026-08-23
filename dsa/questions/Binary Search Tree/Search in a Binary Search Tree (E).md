Previous: [[Lowest Common Ancestor of a Binary Tree (M)]] | Next: [[Insert into a Binary Search Tree (M)]]

You are given the root of a binary search tree (BST) and an integer val.

Find the node in the BST that the node's value equals val and return the subtree rooted with that node. If such a node does not exist, return null.

## Solution

Pretty simple and the main charm of BST. Return if val found, go right if node->val is smaller than val else go right. If we hit null, then return null.

```C++
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        while (root != nullptr) {
            int n = root->val;

            if (n == val) {
                return root;
            } else if (n < val) {
                root = root->right;
            } else {
                root = root->left;
            }
        }
        return nullptr;
    }
}
```