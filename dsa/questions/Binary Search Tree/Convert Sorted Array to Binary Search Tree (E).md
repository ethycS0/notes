Previous: [[Insert into a Binary Search Tree (M)]] | Next: [[Two Sum IV - Input is a BST]]

Given an integer array nums where the elements are sorted in ascending order, convert it to a height-balanced binary search tree.

## Solution

This one is actually quite difficult to grasp mentally. Especially iteratively, it feels quite difficult to maintain the positions and subarrays. The overall explanation is that we are basically doing binary search, creating these small arrays and the mid points are root nodes, the secondary mids are connceted to the root and so on. So if we now just implement a recursive solution, it magically works.

```C++
class Solution {
private:
    TreeNode* build_bst(vector<int>& n, int left, int right) {
        if (left > right) {
            return nullptr;
        }

        int mid = left + (right - left) / 2;
        TreeNode* node = new TreeNode;

        node->val = n.at(mid);
        node->left = build_bst(n, left, mid - 1);
        node->right = build_bst(n, mid + 1, right);

        return node;
    }

public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        if (nums.empty()) {
            return nullptr;
        }

        TreeNode* root = build_bst(nums, 0, nums.size() - 1);
        return root;
    }
};
```