Previous: [[Permutations (M)]] | Next: [[Linked List Cycle (E)]]

https://leetcode.com/problems/middle-of-the-linked-list/

You are given the head of a singly linked list, return the middle node of the linked list.

If there are two middle nodes, return the second middle node.

## Solution

This question can be done with fast and slow pointers. When fast pointer that jumps twice and slow only goes next once, when fast->next = nullptr then the answer is slow. If fast->next->next is nullptr, slow-> next is mid. Thats how we can use fast and slow pointers to get the answer. Just keep edge cases in mind.

```C++
class Solution {
   public:
    ListNode* middleNode(ListNode* head) {
        // ListNode* fast = head ? head->next : nullptr;    // For first middle
        ListNode* fast = head;
        ListNode* slow = head;

        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }

        return slow;
    }
}
```
