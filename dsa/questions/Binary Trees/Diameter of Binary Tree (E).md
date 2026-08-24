Previous: [[Path Sum (E)]] | Next: [[Invert Binary Tree (E)]]

https://leetcode.com/problems/diameter-of-binary-tree/

The diameter of a binary tree is defined as the length of the longest path between any two nodes within the tree. The path does not necessarily have to pass through the root.

The length of a path between two nodes in a binary tree is the number of edges between the nodes. Note that the path can not include the same node twice.

Given the root of a binary tree root, return the diameter of the tree.

## Solution

This is tricky one. Here we will recursively traverse till the leaf nodes. Now we return the depth. At each return, we will return the max depth, but we keep a global counter. Basically we add both left and right depths at each points and compare with the global max_diameter. This way, we can keep returning depths but also keep and calculate max_diameter.

```C++
class Solution {
   private:
    int max_diameter;
    int get_depth(TreeNode* n) {
        int left_depth = 0;
        int right_depth = 0;

        if (n->left != nullptr) {
            left_depth = get_depth(n->left) + 1;
        }

        if (n->right != nullptr) {
            right_depth = get_depth(n->right) + 1;
        }

        int diameter = left_depth + right_depth;
        max_diameter = max(max_diameter, diameter);

        return max(left_depth, right_depth);
    }

   public:
    int diameterOfBinaryTree(TreeNode* root) {
        max_diameter = 0;
        if(root != nullptr) {
            get_depth(root);
        }

        return max_diameter;
    }
};

```
