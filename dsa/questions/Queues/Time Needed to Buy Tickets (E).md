Previous: [[Implement Stack Using Queues (E)]] | Next: [[Reverse First K elements of a Queue (LOL)]]
There are n people in a line queuing to buy tickets, where the 0-th person is at the front of the line and the (n - 1)-th person is at the back of the line.

You are given a 0-indexed integer array tickets of length n where the number of tickets that the i-th person would like to buy is tickets[i].

Each person takes exactly 1 second to buy a ticket. A person can only buy 1 ticket at a time and has to go back to the end of the line (which happens instantaneously) in order to buy more tickets. If a person does not have any tickets left to buy, the person will leave the line.

Return the time taken for the person initially at position k (0-indexed) to finish buying tickets.

## Solution

Deceptively tough for me. I kept tracking the person while the queue was moving with indices and managing it while the queue is shortening was tough. The final solution was to just simulate a queue and decrement tickets vector separately. Tracking and ticket counting got easier that way. Another solution is a mathematical one with queue simulation.

```C++
class Solution {
   public:
    int timeRequiredToBuy(vector<int>& tickets, int k) {
        queue<int> q;
        for (int i = 0; i < tickets.size(); i++) {
            q.push(i);
        }

        int sec = 0;

        while (!q.empty()) {
            int p = q.front();
            q.pop();

            tickets[p] -= 1;
            sec += 1;

            if (p == k && tickets[p] == 0) {
                return sec;
            }

            if (tickets[p] > 0) {
                q.push(p);
            }
        }

        return sec;
    }
}
```

```C++
class Solution {
   public:
    int timeRequiredToBuy(vector<int>& tickets, int k) {
        int seconds = 0;
        int target = tickets[k];

        for (int i = 0; i < tickets.size(); ++i) {
            if (i <= k) {
                seconds += std::min(tickets[i], target);
            } else {
                seconds += std::min(tickets[i], target - 1);
            }
        }

        return seconds;
    }
}
```