Previous: [[Min Stack (M)]] | Next: [[Evaluate Reverse Polish Notation (M)]]

You are given a string s consisting of the following characters: '(', ')', '{', '}', '[' and ']'.

The input string s is valid if and only if:

    Every open bracket is closed by the same type of close bracket.
    Open brackets are closed in the correct order.
    Every close bracket has a corresponding open bracket of the same type.

Return true if s is a valid string, and false otherwise.

## Solution
 
This is pretty simple. If the brackets are opening, just push on the stack. If they are not opening brackets, then check top and validate if the symbols top and c are correct. If not or stack is randomly empty, return false. After full iterations, check if stack is empty. If yes, return true else false.

```C++
class Solution {
   public:
    bool isValid(string s) {
        stack<char> prt;

        for (int i = 0; i < s.size(); i++) {
            if (s[i] == '(' || s[i] == '{' || s[i] == '[') {
                prt.push(s[i]);
            } else {
                if (prt.empty()) {
                    return false;
                }

                char top = prt.top();
                if ((s[i] == ')' && top == '(') || (s[i] == '}' && top == '{') ||
                    (s[i] == ']' && top == '[')) {
                    prt.pop();
                } else {
                    return false;
                }
            }
        }

        return prt.empty();
    }
};
```