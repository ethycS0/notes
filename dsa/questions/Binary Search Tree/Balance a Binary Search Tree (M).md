Previous: [[Minimum Absolute Difference in Binary Search Tree (E)]] | Next: [[Delete Node in a Binary Search Tree (M)]]

Given the root of a binary search tree, return a balanced binary search tree with the same node values. If there is more than one answer, return any of them.

A binary search tree is balanced if the depth of the two subtrees of every node never differs by more than 1.

## Solution

This problem requires usage of patterns learnt in other problems. Basically, we can traverse the given unbalanced BST inorder and create a sorted array out of it. Once we can do that, we can then create a height balanced BST out of that sorted array. If known inorder implementation and solved the sorted array question, this problem is simple.

```C++
class Solution {
private:
    vector<int> arr;
    void inorder(TreeNode* node) {
        if (!node) {
            return;
        }

        inorder(node->left);
        arr.push_back(node->val);

        inorder(node->right);
    }

    TreeNode* construct_tree(int left, int right) {
        if (left > right) {
            return nullptr;
        }

        int mid = left + (right - left) / 2;
        TreeNode* node = new TreeNode;

        node->val = arr.at(mid);
        node->left = construct_tree(left, mid - 1);
        node->right = construct_tree(mid + 1, right);

        return node;
    }

public:
    TreeNode* balanceBST(TreeNode* root) {
        inorder(root);

        return construct_tree(0, arr.size() - 1);
    }
};
```