Previous: [[Middle of the Linked List (E)]] | Next: [[Reverse a Linked List (E)]]

Given the beginning of a linked list head, return true if there is a cycle in the linked list. Otherwise, return false.

There is a cycle in a linked list if at least one node in the list can be visited again by following the next pointer.

Internally, index determines the index of the beginning of the cycle, if it exists. The tail node of the list will set it's next pointer to the index-th node. If index = -1, then the tail node points to null and no cycle exists.

## Solution

Again we employ use of fast and slow pointers. If fast->next or fast->next->next ever are nullptr, we can return false. If fast == slow ever that means we have gotten a loop and return true.

```C++
class Solution {
public:
    bool hasCycle(ListNode* head) {
        ListNode* fast = head;
        ListNode* slow = head;

        while(fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;

            if(slow == fast) {
                return true;
            }
        }

        return false;
    }
}
```