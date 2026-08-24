Previous: [[Valid Parentheses (E)]] | Next: [[Removing Stars From a String (M)]]

https://leetcode.com/problems/evaluate-reverse-polish-notation/

You are given an array of strings tokens that represents a valid arithmetic expression in Reverse Polish Notation.

Return the integer that represents the evaluation of the expression.

    The operands may be integers or the results of other operations.
    The operators include '+', '-', '*', and '/'.
    Assume that division between integers always truncates toward zero.
## Solution

Pretty easy, simple use of stack. The problem was C++ stdlib string and char usage that confused me the most. Otherwise quite simple solution.

```C++
class Solution {
   public:
    int evalRPN(vector<string>& tokens) {
        stack<int> s;

        for (const string& t : tokens) {
            if (t == "+" || t == "-" || t == "*" || t == "/") {
                int n2 = s.top();
                s.pop();
                int n1 = s.top();
                s.pop();

                if (t == "+") {
                    s.push(n1 + n2);
                } else if (t == "-") {
                    s.push(n1 - n2);
                } else if (t == "*") {
                    s.push(n1 * n2);
                } else if (t == "/") {
                    s.push(n1 / n2);
                }
            } else {
                s.push(stoi(t));
            }
        }

        return s.top();
    }
};

```
