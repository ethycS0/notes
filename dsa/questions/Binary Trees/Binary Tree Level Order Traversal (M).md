Previous: [[Maximum Depth of Binary Tree (E)]] | Next: [[Same Tree (E)]]

https://leetcode.com/problems/binary-tree-level-order-traversal/

Given a binary tree root, return the level order traversal of it as a nested list, where each sublist contains the values of nodes at a particular level in the tree, from left to right.

## Solution

This is also a pretty straight forward BFS solution. Just perform tree traaversal in root -> left -> right (append to queue) manner and append values. TO make sure levels are properly formatted, iterate through the q based on precomputed size for that level.

```C++
class Solution {
   public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> r;
        if(root == nullptr) {
            return r;
        }

        queue<TreeNode*> q;
        q.push(root);

        while (q.size() > 0) {
            vector<int> val;
            int n = q.size();

            for (int i = 0; i < n; i++) {
                TreeNode* node = q.front();
                q.pop();

                val.push_back(node->val);

                if (node->left != nullptr) {
                    q.push(node->left);
                }

                if (node->right != nullptr) {
                    q.push(node->right);
                }
            }

            r.push_back(val);
        }

        return r;
    }
}
```
