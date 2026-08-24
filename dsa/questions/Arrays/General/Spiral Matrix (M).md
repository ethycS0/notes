Previous: [[Minimum Time Visiting All Points (E)]] | Next: [[Rotate Array (M)]]

https://leetcode.com/problems/spiral-matrix/

Given an m x n matrix of integers matrix, return a list of all elements within the matrix in spiral order.

## Solution

t b l r

t, l = 0
b, r = size - 1

while t <= b & l <= r:

insert l -> r 
t += 1
insert t -> b 
r -= 1
check t <= b insert r -> l
b -= 1
check l <= r insert b -> t
l += 1

```C++
class Solution {
   public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if (matrix.empty() || matrix[0].empty()) {
            return {};
        }

        vector<int> r;

        int top = 0;
        int bottom = matrix.size() - 1;
        int left = 0;
        int right = matrix[0].size() - 1;

        while (top <= bottom && left <= right) {
            for (int i = left; i <= right; i++) {
                r.push_back(matrix[top][i]);
            }
            top += 1;

            for (int i = top; i <= bottom; i++) {
                r.push_back(matrix[i][right]);
            }
            right -= 1;

            if (top <= bottom) {
                for (int i = right; i >= left; i--) {
                    r.push_back(matrix[bottom][i]);
                }
                bottom -= 1;
            }

            if (left <= right) {
                for (int i = bottom; i >= top; i--) {
                    r.push_back(matrix[i][left]);
                }
                left += 1;
            }
        }

        return r;
    }
};

```
