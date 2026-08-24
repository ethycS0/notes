Previous: [[Time Needed to Buy Tickets (E)]] | Next: [[Reverse First K elements of a Queue (LOL)]]

Given a stack of integers, sort them in ascending or descending order.

## Solution

Use a temporary stack here. Basically juggle elements between both stacks. Keep empty checks intact.

```C++
stack<int> sort_stack(stack<int> &input) {
	stack<int> temp;
	
	while (!input.empty()) {
		int t = input.top();
		input.pop();
		
		while (!temp.empty() && temp.top() < t) {
			input.push(temp.top());
			temp.pop();
		}
		
		temp.push(t);
	}
	
	return temp;
}
```
