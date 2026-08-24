Previous: [[Same Tree (E)]] | Next: [[Diameter of Binary Tree (E)]]

https://leetcode.com/problems/path-sum/

You are given the root of a binary tree and an integer targetSum, return true if the tree has a root-to-leaf path such that adding up all the values along the path equals targetSum.

A leaf is a node with no children.

## Solution

This was pretty easy, just make sure the checks are correct. Traverse the tree and add the values. Make pair queue to send sums down the tree and check sum check at the leaf nodes.

```C++
class Solution {
   public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }

        queue<pair<TreeNode*, int>> q;
        q.push({root, 0});

        while (!q.empty()) {
            pair<TreeNode*, int> n = q.front();
            q.pop();

            int sum = n.second + n.first->val;

            if (sum == targetSum && n.first->left == nullptr && n.first->right == nullptr) {
                return true;
            }

            if (n.first->left != nullptr) {
                q.push({n.first->left, sum});
            }

            if (n.first->right != nullptr) {
                q.push({n.first->right, sum});
            }
        }

        return false;
    }
};
```
