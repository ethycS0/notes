Previous: [[Average of Levels in Binary Tree (E)]] | Next: [[Maximum Depth of Binary Tree (E)]]

https://leetcode.com/problems/minimum-depth-of-binary-tree/

Given a binary tree, find its minimum depth.

The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.

Note: A leaf is a node with no children.

## Solution

Pretty similar to the previous question. BFS. Here we just maintain a queue that takes TreeNode pointer and its level. When we find a node that does not ahve children, we return the node level + 1.

```C++
class Solution {
public:
    int minDepth(TreeNode* root) {
        int level = 0;

        if (root == nullptr) {
            return level;
        }

        queue<pair<TreeNode*, int>> q;
        q.push({root, level});

        while (!q.empty()) {
            pair<TreeNode*, int> n = q.front();
            q.pop();

            if (n.first->left == nullptr && n.first->right == nullptr) {
                return n.second + 1;
            }

            if (n.first->left != nullptr) {
                q.push({n.first->left, n.second + 1});
            }

            if (n.first->right != nullptr) {
                q.push({n.first->right, n.second + 1});
            }
        }

        return 0;
    }
};
```
