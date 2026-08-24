Previous: [[Invert Binary Tree (E)]] | Next: [[Balanced Binary Tree (E)]]

https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/

You are given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes p and q as the lowest node in T that has both p and q as descendants (where we allow a node to be a descendant of itself).”

In this context, "lowest" means the deepest node in the tree, not the node with the smallest value.

## Solution

We can solve this question recursively by returning true if p or q is found. We then return true back up the tree till we find the LCA where either both left and right will return true or LCA itself is p or q and left or right has returned true.

```C++
class Solution {
   private:
    TreeNode* lca;
    bool find_ancestor(TreeNode* node, TreeNode* p, TreeNode* q) {
        bool left = false, right = false;

        if (node->left != nullptr) {
            left = find_ancestor(node->left, p, q);
        }

        if (node->right != nullptr) {
            right = find_ancestor(node->right, p, q);
        }

        bool x = left || right;

        if (left == true && right == true) {
            lca = node;
        } else if(x && node == p) {
            lca = node;
        } else if(x && node == q){
            lca = node;
        }

        return (x || node == p || node == q);
    }

   public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        lca = root;
        if(root) {
            find_ancestor(root, p, q);
        }

        return lca;
    }
};
```
