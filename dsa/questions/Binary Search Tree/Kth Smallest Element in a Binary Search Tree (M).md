Previous: [[Delete Node in a Binary Search Tree (M)]] | Next: [[Kth Largest Element in an Array (M) TODO]]

https://leetcode.com/problems/kth-smallest-element-in-a-bst/

Given the root of a binary search tree, and an integer k, return the kth smallest value (1-indexed) in the tree.

A binary search tree satisfies the following constraints:

    The left subtree of every node contains only nodes with keys less than the node's key.
    The right subtree of every node contains only nodes with keys greater than the node's key.
    Both the left and right subtrees are also binary search trees.

## Solution

This is a simple solution if you know inorder traversal iteratively. We just decrement i every node visited when we start the inorder traversal. Since inorder guarantees Minimum -> Maximum ascending order traversal, we can get kth element as so.

```C++
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode*> s;

        while (root || !s.empty()) {
            if (root) {
                s.push(root);
                root = root->left;
                continue;
            } 
            
            root = s.top();
            s.pop();
            k -= 1;

            if (k == 0) {
                return root->val;
            }

            root = root->right;
        }

        return 0;
    }
};
```
