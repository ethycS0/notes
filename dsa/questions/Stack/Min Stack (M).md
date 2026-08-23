Previous: [[Merge Two Sorted Linked Lists (E)]] | Next: [[Valid Parenthesis (E)]]

Design a stack class that supports the push, pop, top, and getMin operations.

    MinStack() initializes the stack object.
    void push(int val) pushes the element val onto the stack.
    void pop() removes the element on the top of the stack.
    int top() gets the top element of the stack.
    int getMin() retrieves the minimum element in the stack.

Each function should run in O(1)O(1) time.

## Solution

There are multiple ways to solve this question. Bruteforce where get_min() will traversethe stack and find the minimum, mine which was a stack of pairs and the best method which does arithetic to maintain min to be 0 and all others variables to be val - min or something.

```C++
class MinStack {
private:
    stack<pair<int, int>> s;
    
public:
    MinStack() = default;
    
    void push(int val) {
        if (s.empty()) {
            s.push({val, val});
            return;
        }

        pair<int, int> t = s.top();
        pair<int, int> n = {val, -1};

        if (val < t.second) {
            n.second = val;
        } else {
            n.second = t.second;
        }

        s.push(n);
    }
    
    void pop() {
        s.pop();
    }
    
    int top() {
        return s.top().first;
    }
    
    int getMin() {
        return s.top().second;
    }
};

```