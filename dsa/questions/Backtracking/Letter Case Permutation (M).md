Previous: [[Range Sum Query Immutable (E)]] | Next: [[Subsets (M)]]

Given a string s, you can transform every letter individually to be lowercase or uppercase to create another string.

Return a list of all possible strings we could create. Return the output in any order.

## Solution

Here we start with Backtracking with BFS like code structure. Essentially, first we create an empty vector of strings. The we loop ove all the characters of the given string. We check if the character is an interger, if yes, we append the integer to all the strings in res vector. If it is an alphabet, we push_back a new string with the character being upper case and add the lower case letter to the remaining strings. This is how we build each string.


Example:


a1b2


a -> not a digit
res = ["a", "A"]

1 -> digit
res = ["a1", "A1"]

b -> not a digit
res = ["a1b", "A1b", "a1B", "A1B"]

2 -> digit
res = ["a1b2", "A1b2", "a1B2", "A1B2"]

```C++
class NumArray {
private: 
    vector<int> dp;

public:
    NumArray(vector<int>& nums) {
        dp.resize(nums.size() + 1, 0);

        for (int i = 0; i < nums.size(); i++) {
            dp.at(i + 1) = dp.at(i) + nums.at(i);
        }
    }
    
    int sumRange(int left, int right) {
        return (dp.at(right + 1) - dp.at(left));
    }
};
```