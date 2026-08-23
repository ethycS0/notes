Previous: [[Missing Number (E)]] | Next: [[Two Sum (E)]]

You are given an array nums of n integers where nums[i] is in the range [1, n], return an array of all the integers in the range [1, n] that do not appear in nums.

## Solutions

- Linear: Insert all values into set and then loop over all values and check if present.
- Cooler Linear: iterate and do temp=[i]-1, if [temp] > 0 then negate [temp]. Finally iterate over the array to find positive integers and append i+1.{I dont understand this shit but works}

```C++
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        unordered_set<int> s;
        vector<int> r;

        for (auto n : nums) {
            s.insert(n);
        }

        for (int i = 1; i < nums.size() + 1; i++) {
            if(s.find(i) == s.end()) {
                r.push_back(i);
            }
        }

        return r;
    }
}
```