Previous: [[Reverse Linked List II (M)]] | Next: [[Merge Two Sorted Linked Lists (E)]]

You are given the head of a singly linked list, return true if it is a palindrome or false otherwise.

A palindrome is a sequence that reads the same forward and backward.

## Solution

Find the mid point. Reverse the list from the mid point and compare head vs mid while iterating over both the lists.

```C++
class Solution {
   private:
    ListNode* get_mid(ListNode* head) {
        ListNode* fast = head ? head->next : nullptr;
        ListNode* slow = head;

        while(fast != nullptr && fast->next != nullptr){
            slow = slow->next;
            fast = fast->next->next;
        }

        return slow;
    }

    ListNode* reverse_list(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;

        while(curr != nullptr) {
            ListNode* next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }

        return prev;
    }

   public:
    bool isPalindrome(ListNode* head) {
        ListNode* head2 = reverse_list(get_mid(head));

        while(head != nullptr) {
            if(head->val != head2->val) {
                return false;
            }

            head = head->next;
            head2 = head2->next;
        }

        return true;
    }
};
```