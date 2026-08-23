Previous: [[Squares of a Sorted Array (E)]] | Next: [[Longest Mountain in Array (M)]]

Given an integer array nums, return all the triplets [nums[i], nums[j], nums[k]] where nums[i] + nums[j] + nums[k] == 0, and the indices i, j and k are all distinct.

The output should not contain any duplicate triplets. You may return the output and the triplets in any order.

## Solution

- Cubic: Bruteforce to find result.
- Quadratic: Sort, Loop to get ith value. Then use two pointers such that j = i + 1 and k = n.size() - 1. If nums[i] + nums[j] + nums[k] < 0, j += 1, else > 0, k -= 1 else == 0, append nums[i], nums[j], nums[k] to res. Edge cases, i j and k should skip dupliates. we should break when nums[i] > 0 since then we will never get to 0 anyways.

```C++
class Solution {
   public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> result;
        int n = nums.size();
        if (n < 3) {
            return result;
        }

        sort(nums.begin(), nums.end());

        for (int i = 0; i < n - 2; i++) {
            if (nums[i] > 0) {
                break;
            }

            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }

            int j = i + 1;
            int k = n - 1;

            while (j < k) {
                int sum = nums[i] + nums[j] + nums[k];

                if (sum == 0) {
                    result.push_back({nums[i], nums[j], nums[k]});

                    while (j < k && nums[j] == nums[j + 1]) {
                        j++;
                    }

                    while (j < k && nums[k] == nums[k - 1]) {
                        k--;
                    }

                    j++;
                    k--;

                } else if (sum < 0) {
                    j++;
                } else {
                    k--;
                }
            }
        }

        return result;
    }
};
```