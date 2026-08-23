Previous: [[Convert Sorted Array to Binary Search Tree (E)]] | Next: [[Lowest Common Ancestor in Binary Search Tree (M)]]

Given the root of a binary search tree and an integer k, return true if there exist two elements in the BST such that their sum is equal to k, or false otherwise.

## Solution

There is an easier solution to this question than the implementation. Just lefel order traversal and adding values to hashmap. Doing the same again with find(k - val) in the hashmap, if found, return true. Pretty simple. The O(1) space solution is much more trickier. It works more for BST with a two pointers and two traversal approach. Essentially, each iteration one of the 2 traversal of the two pointers takes place, inorder if sum < k and reverse inorder if sum > k. We can track these 2 pointers and if sum == k ever, return true. If both stack are empty, ie no more traversals possible and ln == rn, break and return false.

```C++
class Solution {
public:
    bool findTarget(TreeNode* root, int k) {
        if (!root) {
            return false;
        }

        stack<TreeNode*> ls;
        stack<TreeNode*> rs;

        TreeNode* curr = root;
        while (curr) {
            ls.push(curr);
            curr = curr->left;
        }

        curr = root;
        while (curr) {
            rs.push(curr);
            curr = curr->right;
        }

        while (!ls.empty() && !rs.empty()) {
            TreeNode* ln = ls.top();
            TreeNode* rn = rs.top();

            if (ln == rn) {
                break;
            }

            int sum = ln->val + rn->val;

            if (sum == k) {
                return true;
            } else if (sum < k) {
                ls.pop();
                curr = ln->right;
                while (curr) {
                    ls.push(curr);
                    curr = curr->left;
                }
            } else {
                rs.pop();
                curr = rn->left;
                while (curr) {
                    rs.push(curr);
                    curr = curr->right;
                }
            }
        }

        return false;
    }
};
```