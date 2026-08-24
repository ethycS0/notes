Previous: [[Search in a Binary Search Tree (E)]] | Next: [[Convert Sorted Array to Binary Search Tree (E)]]

https://leetcode.com/problems/insert-into-a-binary-search-tree/

You are given the root node of a binary search tree (BST) and a value val to insert into the tree. Return the root node of the BST after the insertion. It is guaranteed that the new value does not exist in the original BST.

## Solution

No need to overthink this. it is quite straight forward. Bascially if current node value is smaller, go right else go left. Then we check if the direction is a nullptr, if it is, create a new node with the required value and attach current node left or right to the new node. if it is not a nullptr, just traverse that direction. When attached a new node, break and return the root.

```C++
class Solution {
   public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if (root == nullptr) {
            return new TreeNode(val);
        }

        TreeNode* node = root;

        while (node != nullptr) {
            int n = node->val;

            if (n < val) {
                if (node->right == nullptr) {
                    TreeNode* new_node = new TreeNode(val);
                    node->right = new_node;
                    break;
                } else {
                    node = node->right;
                }
            } else {
                if (node->left == nullptr) {
                    TreeNode* new_node = new TreeNode(val);
                    node->left = new_node;
                    break;
                } else {
                    node = node->left;
                }
            }
        }

        return root;
    }
};
```
