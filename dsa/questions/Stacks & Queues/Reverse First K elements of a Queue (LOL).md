Previous: [[Stack Sorting (LOL)]] | Next: [[LRU Cache (M)]]

Given an integer k and a queue of integers, we need to reverse the order of the first k elements of the queue, leaving the other elements in the same relative order.

Only following standard operations are allowed on queue.

    enqueue(x) : Add an item x to rear of queue
    dequeue() : Remove an item from front of queue
    size() : Returns number of elements in queue.
    front() : Finds front item.

Note: 

    The above operations represent the general processings.
    In-built functions of the respective languages can be used to solve the problem.
    If the size of queue is smaller than the given k , then return the original queue.

## Solution

Pretty simple, dequeue k elements from the queue and push to the stack. Now do the opposite, the FIFO -> LIFO coversion will reverse the elements but might require cycling the queue a bit.

```C++
queue<int> modifyQueue(queue<int> q, int k) {
    if (q.empty() || k <= 0 || k > q.size()) {
        return q;
    }

    stack<int> st;
	
    for (int i = 0; i < k; ++i) {
        st.push(q.front());
        q.pop();
    }
	
    while (!st.empty()) {
        q.push(st.top());
        st.pop();
    }

    int size = q.size();
    for (int i = 0; i < size - k; ++i) {
        q.push(q.front());
        q.pop();
    }

    return q;
}
```
