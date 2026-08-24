Previous: [[Binary Tree Level Order Traversal (M)]] | Next: [[Path Sum (E)]]

https://leetcode.com/problems/same-tree/

Given the roots of two binary trees p and q, return true if the trees are equivalent, otherwise return false.

Two binary trees are considered equivalent if they share the exact same structure and the nodes have the same values.

## Solution

Pretty simple solution, just make sure to manage edge cases of roots being nullptrs. Then simply traverse the nodes and compare each. Make sure to have a good nullptrs check.

```C++
class Solution {
   public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if (p == nullptr && q == nullptr) {
            return true;
        } else if (p == nullptr) {
            return false;
        } else if (q == nullptr) {
            return false;
        }

        queue<pair<TreeNode*, TreeNode*>> s;
        s.push({p, q});

        while (s.size() > 0) {
            pair<TreeNode*, TreeNode*> p = s.front();
            TreeNode* x = p.first;
            TreeNode* y = p.second;

            s.pop();

            if (x->val != y->val) {
                return false;
            }

            if (x->right != nullptr && y->right != nullptr) {
                s.push({x->right, y->right});
            } else if (x->right == nullptr) {
                if (y->right != nullptr) {
                    return false;
                }
            } else if (y->right == nullptr) {
                if (x->right != nullptr) {
                    return false;
                }
            }
            if (x->left != nullptr && y->left != nullptr) {
                s.push({x->left, y->left});
            } else if (x->left == nullptr) {
                if (y->left != nullptr) {
                    return false;
                }
            } else if (y->left == nullptr) {
                if (x->left != nullptr) {
                    return false;
                }
            }
        }

        return true;
    }
};
```
