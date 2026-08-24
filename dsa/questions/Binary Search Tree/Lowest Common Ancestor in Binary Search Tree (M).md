Previous: [[Two Sum IV - Input is a BST (E)]] | Next: [[Minimum Absolute Difference in Binary Search Tree (E)]]

https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/

Given a binary search tree (BST) where all node values are unique, and two nodes from the tree p and q, return the lowest common ancestor (LCA) of the two nodes.

The lowest common ancestor between two nodes p and q is the lowest node in a tree T such that both p and q are descendants. The ancestor is allowed to be a descendant of itself.

## Solution

We can solve this question recursively with DFS similar to LCA of Binary Tree. But let try using the attributes of BST. This actually significantly simplifies the answer. It honestly feels wrong but, we basically move root if it is greater than or smaller than both the nodes. If there is a situation when it is in the middle we immedietely return root.

```C++
class Solution {
   public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (!root) {
            return nullptr;
        }
        
        while (true) {
            if (root->val > p->val && root->val > q->val) {
                root = root->left;
            } else if (root->val < p->val && root->val < q->val) {
                root = root->right;
            } else {
                return root;
            }
        }

        return nullptr;
    }
}
```
