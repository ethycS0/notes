Previous: [[Diameter of Binary Tree (E)]] | Next: [[Lowest Common Ancestor of a Binary Tree (M)]]

https://leetcode.com/problems/invert-binary-tree/

You are given the root of a binary tree root. Invert the binary tree and return its root.

## Solution

This is easier both iteratively and recursively. Just traverse the tree, push the nodesm and switch them for root. Return root.

```C++
class Solution {
   private:
    void invert(TreeNode* n) {
        if (n->left != nullptr) {
            invert(n->left);
        }

        if (n->right != nullptr) {
            invert(n->right);
        }

        TreeNode* temp = n->left;

        n->left = n->right;
        n->right = temp;
    }

   public:
    TreeNode* invertTree(TreeNode* root) {
        // if (root != nullptr) { invert(root); }   // For Recursive Solution

        if (root == nullptr) {
            return nullptr;
        }

        queue<TreeNode*> q;
        q.push(root);

        while (!q.empty()) {
            TreeNode* n = q.front();
            q.pop();

            if (n->left != nullptr) {
                q.push(n->left);
            }

            if (n->right != nullptr) {
                q.push(n->right);
            }

            TreeNode* temp = n->left;
            n->left = n->right;
            n->right = temp;
        }

        return root;
    }
};

```
