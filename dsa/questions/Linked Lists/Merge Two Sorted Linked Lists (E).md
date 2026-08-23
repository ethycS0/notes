Previous: [[Palindrome Linked List (E)]] | Next: [[Min Stack (M)]]

You are given the heads of two sorted linked lists list1 and list2.

Merge the two lists into one sorted linked list and return the head of the new sorted linked list.

The new list should be made up of nodes from list1 and list2.


## Solution

Handle edge cases, null lists etc. When loop till both heads are null. We will then check if val of 1 <= 2. If its the start, you might prefer taking current nullptr and attaching to a list. Ideally, dummy pointer maybe the best. Basically, just keep track and properly attach and detach nodes. Simple enough. 

```C++
class Solution {
   public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if(list1 == nullptr) {
            return list2;
        }

        if(list2 == nullptr) {
            return list1;
        }

        ListNode* curr = nullptr;
        ListNode* head = nullptr;

        while (list1 != nullptr && list2 != nullptr) {
            if (list1->val <= list2->val) {
                if(curr == nullptr) {
                    curr = list1;
                } else {
                    curr->next = list1;
                    curr = list1;
                }

                list1 = list1->next;

            } else {
                if(curr == nullptr) {
                    curr = list2;
                } else {
                    curr->next = list2;
                    curr = list2;
                }

                list2 = list2->next;

            }

            if(head == nullptr) {
                head = curr;
            }
        }

        if(list1 == nullptr) {
            curr->next = list2;
        } else if(list2 == nullptr) {
            curr->next = list1;
        }

        return head;
    }
};

```