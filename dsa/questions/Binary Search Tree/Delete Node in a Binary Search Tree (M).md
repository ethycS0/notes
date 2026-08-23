Previous: [[Balance a Binary Search Tree (M)]] | Next: [[Kth Smallest Element in a Binary Search Tree (M)]]

Given a root node reference of a BST and a key, delete the node with the given key in the BST. Return the root node reference (possibly updated) of the BST.

Basically, the deletion can be divided into two stages:

    Search for a node to remove.
    If the node is found, delete the node.

## Solution

This was the toughest question of today. Deletion of a node in a BST is quite weird because we have to handle 3 cases with the last case being worst.
1. Leaf Node -> we can just delete
2. Parent of one -> here we need to replace the current node with the singular child. We cannot just copy over the value and delete cuz the child may have more children.
3. Parent of two -> the weirdest case. I still done understand fully clearly but here you go. Basically, we need to find a successor. It has to be the lowest of the right branch. This can be found using inorder, basically just going left the whole time. Once the successor is found, we copy over the value to the current node. Now we have to handle the successors childer, we cannot just delete the successor. Fortuately the magic of recursivelly calling the delete function implemented on the right subtree with the key passed as the successor value will handle the situation pretty damn well.

```C++
class Solution {
   private:
    TreeNode* inorder(TreeNode* node) {
        if (!node->left) {
            return node;
        }

        return inorder(node->left);
    }

    TreeNode* delete_recursive(TreeNode* node, int key) {
        if (!node) {
            return nullptr;
        }

        if (key < node->val) {
            node->left = delete_recursive(node->left, key);
        } else if (key > node->val) {
            node->right = delete_recursive(node->right, key);
        } else {
            if (!node->left || !node->right) {
                TreeNode* temp = (node->left != nullptr) ? node->left : node->right;
                delete node;
                return temp;
            } else {
                TreeNode* temp = inorder(node->right);
                node->val = temp->val;
                node->right = delete_recursive(node->right, temp->val);
            }
        }

        return node;
    }

   public:
    TreeNode* deleteNode(TreeNode* root, int key) {
        root = delete_recursive(root, key);
        return root;
    }
};
```