Previous: [[Lowest Common Ancestor of a Binary Tree (M)]] | Next: [[Minimum and Maximum Value of a Binary Tree (LOL)]]

https://leetcode.com/problems/balanced-binary-tree/

Given a binary tree, determine if it is height-balanced (a binary tree in which the depth of the two subtrees of every node never differs by more than one).

## Solution
```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left),
 * right(right) {}
 * };
 */
class Solution {
private:
    bool is_balanced;
    int dfs(TreeNode* n) {
        if (!n) {
            return 0;
        }

        int l = 0, r = 0;

        if (n->left) {
            l = dfs(n->left);
        }

        if (n->right) {
            r = dfs(n->right);
        }

        if (abs(l - r) > 1) {
            is_balanced = false;
        }

        return max(l, r) + 1;
    }

public:
    bool isBalanced(TreeNode* root) {
        is_balanced = true;
        dfs(root);

        return is_balanced;
    }
};
```