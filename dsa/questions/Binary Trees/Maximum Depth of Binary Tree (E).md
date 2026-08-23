Previous: [[Minimum Depth of Binary Tree (E)]] | Next: [[Minimum and Maximum Value of a Binary Tree (LOL)]]

Given the root of a binary tree, return its depth.

The depth of a binary tree is defined as the number of nodes along the longest path from the root node down to the farthest leaf node.

## Solution

This can be done with BFS or DFS. I did with DFS, basically we recursively call maxdepth on left and right nodes of a node. Then we add + 1 to the returned value which is the max of left and right node. Leaf nodes start the chain with returning 1 and all the nodes till the root filter out the maximum values upto the root where we find the max depth.

```C++

class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root == nullptr) {
            return 0;
        }

        int left_depth = 0, right_depth = 0;

        if(root->left != nullptr) {
            left_depth = maxDepth(root->left);
        }

        if(root->right != nullptr) {
            right_depth = maxDepth(root->right);
        }

        return (max(left_depth, right_depth) + 1);
    }
}
```