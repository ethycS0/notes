Previous: [[Maximum Depth of Binary Tree (E)]] | Next: [[Binary Tree Level Order Traversal (M)]]

Given the root of an unsorted binary tree, find and return both the minimum and maximum values stored in the tree.

## Solution

```C++
class Solution {
public:
    std::pair<int, int> findMinMax(TreeNode* root) {
        if (!root) {
            return {0, 0};
        }

        auto [leftMin, leftMax] = findMinMax(root->left);
        auto [rightMin, rightMax] = findMinMax(root->right);

        int currentMin = std::min({root->val, leftMin, rightMin});
        int currentMax = std::max({root->val, leftMax, rightMax});

        return {currentMin, currentMax};
    }
```