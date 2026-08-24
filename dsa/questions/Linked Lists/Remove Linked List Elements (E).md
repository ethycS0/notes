Previous: [[Reverse a Linked List (E)]] | Next: [[Reverse Linked List II (M)]]

https://leetcode.com/problems/remove-linked-list-elements/

You are given the head of a linked list and an integer val, remove all the nodes of the linked list that has Node.val == val, and return the new head.


## Solution

Pretty simple, please remember to use dummy nodes and like, think. Think about edge cases. It is pretty simple otherwise.

```C++
class Solution {
   public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode dummy(-1, head);
        ListNode* curr = &dummy;

        while (curr->next != nullptr) {
            if (curr->next->val == val) {
                ListNode* temp = curr->next;
                curr->next = temp->next;
                delete temp;
            } else {
                curr = curr->next;
            }
        }

        return dummy.next;
    }
}
```
