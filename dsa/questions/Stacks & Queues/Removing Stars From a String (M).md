Previous: [[Evaluate Reverse Polish Notation (M)]] | Next: [[Implement Stack Using Queues (E)]]

https://leetcode.com/problems/removing-stars-from-a-string/

You are given a string s, which contains stars *.

In one operation, you can:

    Choose a star in s.
    Remove the closest non-star character to its left, as well as remove the star itself.

Return the string after all stars have been removed.

Note:

    The input will be generated such that the operation is always possible.
    It can be shown that the resulting string will always be unique.

## Solution
```C++
class Solution {
public:
    string removeStars(string s) {
        string answer = "";
        for(auto i : s) {
            if(i == '*') {
                answer.pop_back();
            } else {
                answer.push_back(i);
            }
        }

        return answer;
        
    }
};
```