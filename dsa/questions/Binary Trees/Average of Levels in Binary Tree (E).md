Previous: [[Binary Search (E)]] | Next: [[Minimum Depth of Binary Tree (E)]]

https://leetcode.com/problems/average-of-levels-in-binary-tree/

Given the root of a binary tree, return the average value of the nodes on each level in the form of an array. Answers within 10-5 of the actual answer will be accepted. 

## Solution
Pretty simple solution here. We do BFS. To isolate the size, ie, make sure that only level items are removed every iteration for an average, we precalculate sizeof queue and iterate over that size only. We append the values to a level array and find average at the end. MAKE SURE TO PREALCULATE THE SIZE OF Q.

```C++
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        queue<TreeNode*> q;
        vector<double> res;
        q.push(root);

        while (!q.empty()) {
            vector<double> level;
            int n = q.size();

            for (int i = 0; i < n; i++) {
                TreeNode* node = q.front();
                q.pop();
                level.push_back(node->val);

                if (node->left != nullptr) {
                    q.push(node->left);
                }

                if (node->right != nullptr) {
                    q.push(node->right);
                }
            }

            double r = 0;
            for (auto x : level) {
                r += x;
            }

            r = r / level.size();
            res.push_back(r);
        }

        return res;
    }
};
```
