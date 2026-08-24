Previous: [[Linked List Cycle (E)]] | Next: [[Remove Linked List Elements (E)]]

https://leetcode.com/problems/reverse-linked-list/

Given the beginning of a singly linked list head, reverse the list, and return the new beginning of the list.

## Solution

This requires 2 pointers, current, previous and head. We move the head till nullptr, head = head->next. We keep track of current and set current->next = previous, prev = curr and curr = head.

```C++
class Solution {
   public:
    ListNode* reverseList(ListNode* head) {
        ListNode* curr = head;
        ListNode* prev = nullptr;

        while (curr != nullptr) {
            ListNode* next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }

        return prev;
    }
}
```
